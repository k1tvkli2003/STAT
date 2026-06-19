# Stage 01 — Project, Modules, Toolchain & Budgets

> **Status:** design-complete (platform-independent artifacts authored). Generating the `.uproject`,
> `*.Build.cs`/`*.Target.cs`, compiling the Editor target, and capturing build logs requires a UE5
> workstation/CI — see §11 Evidence.
> **Skill lens:** `/function` (the system spine — ownership, wiring, budgets) + `/anatomy` (module
> boundaries treated as the codebase's information architecture).
> **Governs:** `Prompts/01_MASTER_PROMPT.md`; gated by `Prompts/03_QUALITY_GATES.md`.

---

## 1. Stage Audit (what this stage must establish)

Per `Prompts/02_BUILD_SEQUENCE.md §01`, the foundation must deliver: a compiling Editor target +
empty startup map; module/plugin structure; asset naming/folder conventions, Primary Asset Types,
Gameplay Tag hierarchy, validation commands; build scripts (clean compile, tests, packaged smoke);
and architecture + performance-budget documentation — buildable by a new developer with no private
credentials.

`/function` audit framing for a greenfield project: there is no code to trace yet, so the failure mode
to prevent is **structural** — modules with unclear ownership, circular dependencies, core rules that
end up only in Blueprint, and budgets that are never written down (so nothing can fail a gate later).
This document fixes ownership, dependency direction, and budgets **before** any system is written.

### Toolchain baseline (assumptions — confirm on the dev box)
- **Unreal Engine 5.4+** (World Partition, Data Layers, Enhanced Input, CommonUI, StateTree, Chaos, MetaSounds).
- **Visual Studio 2022** (MSVC v143), Windows 11 SDK; `.NET` for UAT/BuildGraph.
- **Git + Git LFS** for binary content; symbol store for crash decoding.
- Target platform: **Windows (Win64) only.**

---

## 2. Module & Plugin Architecture

**Dependency rule (strict, enforced by `/function`):** dependencies point **downward only**. A higher
layer may depend on a lower one; never the reverse, never sideways across feature plugins except
through a lower shared layer. This keeps core systems testable and prevents the "everything depends on
everything" rot.

```
            ┌─────────────────────────── Editor / Tools (editor-only) ─────────────────────────┐
            │  STATEditor (validators, asset actions)   STATTools (debug, gauntlet helpers)     │
            └───────────────────────────────────────────────────────────────────────────────────┘
                                            ▼ depends on
   Feature plugins (game) ───────────────────────────────────────────────────────────────────────
   STATCombat  STATVehicles  STATAI  STATMissions  STATWorld  STATCrew  STATNarrative  STATUI
        │            │          │          │           │          │           │            │
        └────────────┴──────────┴──────────┴───────────┴──────────┴───────────┴────────────┘
                                            ▼ depends on
   Platform services ─────────────────────────────────────────────────────────────────────────────
   STATSave   STATAudio   STATInput   STATTelemetry   STATOnline(optional)
                                            ▼ depends on
   Core (no game logic, no feature deps) ─────────────────────────────────────────────────────────
   STATCore  (subsystem bases, event/message bus, data-asset bases, RNG streams, gameplay tags,
              budget/stat macros, validation framework, debug-command registry)
                                            ▼
   Engine (UE5)

   Isolated, parallel island (depends only on Core + platform services, NEVER on game features):
   ┌──────────────────────── STATVex (VEX Protocol — educational) ───────────────────────────────┐
   │  Own game flow, own SaveGame namespace, own UI; reuses Save/Audio/Input/Telemetry/UI tokens   │
   └────────────────────────────────────────────────────────────────────────────────────────────┘
```

### Module table (ownership · lifetime · threading · save · why it exists)

| Module / Plugin | Type | Owns | Primary lifetime | Threading | Save impact |
|---|---|---|---|---|---|
| **STATCore** | Runtime | Subsystem & data-asset bases, message bus, deterministic RNG, tags, budget macros, validation | GameInstance/Engine | Game thread; async helpers | Defines save contracts; owns none |
| **STATSave** | Runtime | Versioned snapshots, atomic write, slots, migration, corruption recovery | GameInstance subsystem | Async IO worker | **Owner of save pipeline** |
| **STATInput** | Runtime | Enhanced Input config, contexts, remap, glyphs | LocalPlayer | Game thread | Remap prefs persisted |
| **STATAudio** | Runtime | MetaSounds mix, music/ambience/pursuit states | World/GameInstance | Audio thread | Mix prefs persisted |
| **STATTelemetry** | Runtime | Consent-aware events, offline buffer, sink abstraction | GameInstance subsystem | Async | Consent flag persisted |
| **STATWorld** | Plugin | District streaming, traffic/crowd hooks, time/weather state | World subsystems | Game + Mass (if profiled) | District state |
| **STATCombat** | Plugin | Weapons, damage, health/armor, cover, AI combat hooks | Pawn/components | Game thread; trace budget | Encounter state at checkpoints |
| **STATVehicles** | Plugin | Chaos vehicle pawn/components, entry/exit, damage | Pawn/components | Physics + game thread | Vehicle persistence |
| **STATAI** | Plugin | Perception, StateTree/BT, EQS, encounter/pursuit directors | World/AIController | Game thread; tick-budgeted | Police/heat state |
| **STATMissions** | Plugin | Mission/objective graph, triggers, checkpoints, rewards hooks | World subsystem | Game thread | **Mission state + checkpoints** |
| **STATCrew** | Plugin | Crew data assets, loyalty/injury/availability, safehouse services | GameInstance/World | Game thread | Crew + economy state |
| **STATNarrative** | Plugin | Narrative state, dialogue conditions, subtitles, Sequencer rules | World subsystem | Game thread | Narrative flags |
| **STATUI** | Plugin | CommonUI shell, HUD, map, phone, menus, accessibility, design tokens | LocalPlayer/HUD | Game thread; cheap invalidation | UI prefs persisted |
| **STATVex** | Plugin | VEX Protocol mode: cases, budget, scoring, debrief | Own GameInstance flow | Game thread; deterministic | **Own isolated SaveGame** |
| **STATOnline** | Plugin (optional) | Session/replication abstraction for co-op | GameInstance | Net threads | Session-scoped |
| **STATEditor** | Editor | Data validators, asset actions, content checks | Editor | Editor | — |
| **STATTools** | Editor/Dev | Debug overlays, cheat/heat tools, automation helpers | Editor/Dev | — | — |
| **STATTests** | Test | Automation specs across modules | Test target | — | — |

> **C++ authority note:** every module above implements its *rules* in C++. Blueprints subclass to
> compose content, tune exposed `UPROPERTY` values, and sequence events — they never become the only
> implementation of a save/mission/combat/vehicle/AI/economy/online/telemetry/validation rule.

---

## 3. Build Target Configuration

| Target | Purpose | Notes |
|---|---|---|
| `STATEditor` (Editor) | Day-to-day dev | Loads runtime + editor + tools modules |
| `STAT` (Game) | Standalone runtime | Runtime modules only; no editor deps |
| `STATTests` (Program/Functional) | Automation | Runs `STATTests` specs via Gauntlet/Automation |
| `STATClient`/`STATServer` (deferred) | Co-op only | Created at Stage 17 if slice is stable |

Configs: **Debug, Development, Test, Shipping.** Shipping strips debug commands and verbose logging;
Test keeps automation + stat capture. No editor-only API may leak into the Game target (compile guard).

---

## 4. Coding Standards (C++ authority)

- **Naming:** `U`Object, `A`Actor, `F`struct, `E`enum, `I`interface, `T`template, `b`bool. Subsystems
  `U<Feature>Subsystem`. One public type per header where reasonable. Modules prefix files by domain.
- **Ownership by lifecycle:** pick the *narrowest* owner — GameInstance/Engine/World/LocalPlayer/
  PlayerController/Pawn/ActorComponent/Subsystem/DataAsset. Document the choice in the type's header.
- **Event-driven over polling:** prefer delegates / Gameplay Messages / subsystem events to per-tick
  `Get...()` polling. Any `Tick` must declare why and carry a budget (§9).
- **Determinism:** systems that are scored/saved/replayed/synced/VEX use **stable event ordering** and
  **seeded RNG** from `STATCore` streams — never `FMath::Rand()` in those paths.
- **No hidden risk:** no `TODO`-gated core logic, no fabricated data, no pseudocode-as-implementation.
- **Errors & recovery:** validate at boundaries (nav args, data-asset refs, save fields); fail with
  actionable messages; never silently swallow. (`/function`: a swallowed error is a 🔴.)
- **Const-correctness, `UPROPERTY` save/replication specifiers, and category metadata** are mandatory
  on gameplay-facing members.
- **Comments/text:** English, localization-ready; player-facing strings are `FText`, externalized.

---

## 5. Asset Naming & Folder Conventions

**Prefix_AssetName_Variant** (Unreal community standard, enforced by a `STATEditor` validator).

| Type | Prefix | Example |
|---|---|---|
| Blueprint class | `BP_` | `BP_HeroVehicle` |
| Widget Blueprint | `WBP_` | `WBP_PursuitHUD` |
| Primary Data Asset | `DA_` | `DA_Mission_SlicePrologue` |
| Data Table | `DT_` | `DT_EconomyPrices` |
| Material / Instance | `M_` / `MI_` | `MI_CarPaint_Neon` |
| Texture (suffix role) | `T_` | `T_Asphalt_D` (`_D/_N/_ORM`) |
| Static / Skeletal Mesh | `SM_` / `SK_` | `SM_Streetlight` |
| Niagara System | `NS_` | `NS_TireSmoke` |
| MetaSound / Cue | `MS_` / `A_` | `MS_SirenLoop` |
| Level (World Partition) | `L_` | `L_District_Slice` |
| Input Action / Context | `IA_` / `IMC_` | `IA_Fire`, `IMC_OnFoot` |

**Folder layout** (content mirrors module ownership; no cross-feature reach-ins):
```
Content/
  STAT/
    Core/            Characters/      Vehicles/        Weapons/
    World/Slice/     AI/              Missions/Slice/  Crew/
    Narrative/       UI/{Tokens,HUD,Menus,Map,Phone}/  Audio/
    VFX/             Vex/{Cases,UI}/   _Dev/   (debug-only, excluded from Shipping)
```

---

## 6. Primary Asset Types & Asset Manager Rules

Registered Primary Asset Types (validated; each has a `UPrimaryDataAsset` C++ base in its module):

| PrimaryAssetType | Base (module) | Notes |
|---|---|---|
| `Mission` | `USTATMissionDataAsset` (STATMissions) | objective graph, conditions, rewards, checkpoints |
| `Vehicle` | `USTATVehicleDataAsset` (STATVehicles) | handling, damage, seats |
| `Weapon` | `USTATWeaponDataAsset` (STATCombat) | damage, recoil, spread, ammo |
| `CrewMember` | `USTATCrewDataAsset` (STATCrew) | role, perks, loyalty rules |
| `District` | `USTATDistrictDataAsset` (STATWorld) | streaming sources, layers, budgets |
| `VexCase` | `USTATVexCaseDataAsset` (STATVex) | budget, investigations, scoring, disclaimer |
| `UITheme` | `USTATUIThemeDataAsset` (STATUI) | design tokens (color/type/spacing/motion) |

Asset Manager: cook-by-type rules per PrimaryAssetType; chunk assignment deferred to Stage 20 patching
plan. Every type carries **stable ID + schema version** (save/migration contract from `STATCore`).

---

## 7. Gameplay Tag Hierarchy (root)

Authored in C++/`.ini`, validated so no asset references an undeclared tag.

```
STAT.Input.{OnFoot,Vehicle,Menu,Cinematic}
STAT.State.Player.{Idle,Combat,Driving,Wanted,Arrested,Downed}
STAT.Combat.{Damage.Ballistic,Damage.Melee,Cover.High,Cover.Low,Takedown}
STAT.Heat.Level.{0,1,2,3,4,5}        STAT.Heat.Event.{Witnessed,Evidence,Reported,Lost}
STAT.Mission.{Phase.Intro,Phase.Active,Phase.Fail,Phase.Recover,Phase.Complete}
STAT.Vehicle.{Seat.Driver,Seat.Passenger,Damage.Engine,Damage.Tire}
STAT.Crew.Role.{Driver,Lookout,Mechanic,Shooter}   STAT.Crew.State.{Available,Injured,Busy}
STAT.UI.Layer.{HUD,Menu,Popup,Modal}   STAT.Accessibility.{ReduceShake,HighContrast,HoldToggle}
STAT.Vex.{Phase.Briefing,Phase.Investigation,Phase.Diagnosis,Phase.Debrief}  // isolated namespace
```

---

## 8. Source Control & LFS

- **Git LFS** tracks binary content (`*.uasset *.umap *.wav *.fbx *.png *.tga *.exr *.mp4 *.ttf`) via
  `.gitattributes` (added when first content lands). Code + text stay in plain git.
- **Never committed:** `Binaries/ Intermediate/ DerivedDataCache/ Saved/ Build/` (see `.gitignore`),
  secrets, local-only `.ini`. Crash symbols (`.pdb`) go to a **symbol store**, not the repo.
- Enforce LFS-or-reject and naming validation in CI (§10) so binaries can't sneak into plain git.

---

## 9. Performance & Minimum-Spec Budgets

Targets are **commitments to measure** (`/function`: "never claim a speedup without a measurement
plan"). Each gets a `stat`/Insights marker; gates compare captures to these numbers.

| Budget | Min-spec target | Recommended target | Marker |
|---|---|---|---|
| Frame time (1080p) | 33.3 ms (30 fps) | 16.6 ms (60 fps) | `stat unit` |
| Game thread | ≤ 16 ms | ≤ 9 ms | Insights `Frame` |
| Render thread | ≤ 16 ms | ≤ 9 ms | Insights `RenderThread` |
| GPU (1080p) | ≤ 33 ms | ≤ 16 ms | `stat gpu` |
| CPU AI (traffic+police+combat) | ≤ 4 ms | ≤ 2.5 ms | `STAT.AI` trace |
| Streaming hitch | none > 1 frame over budget | — | Insights stalls |
| Memory (working set) | ≤ 8 GB | ≤ 12 GB | `memreport` |
| VRAM (1080p) | ≤ 6 GB | ≤ 8 GB | `stat rhi` |
| Package size (slice) | ≤ 15 GB | — | build report |
| Level load (slice) | ≤ 20 s cold | ≤ 12 s | load trace |

**Minimum spec (working hypothesis to validate at Stage 19):** quad-core CPU, GTX 1060 6 GB / RX 580,
16 GB RAM, SSD, Windows 10/11 64-bit, 1080p Low @ 30 fps with essential feedback preserved.

---

## 10. CI / Build Guidance (no private credentials)

Scripts to author on the dev box (described here; **no fabricated logs**):

1. **Clean compile** — `RunUAT BuildCookRun -project=STAT.uproject -build -target=STATEditor -configuration=Development`.
2. **Automation tests** — run `STATTests` specs headless (`-ExecCmds="Automation RunTests STAT;Quit"`).
3. **Packaged smoke** — `BuildCookRun -cook -stage -pak -archive` for Win64 Development; boot to the
   empty startup map, assert no fatal log, capture a screenshot.
4. **Content validation** — invoke the `STATEditor` validators (naming, tags, data-asset refs).
5. **BuildGraph** (Stage 20) wraps the above for reproducible Shipping builds + signing + chunking.

CI must run from a clean clone with the engine available, **no machine-specific paths or secrets**.
A new developer reproduces it from this document + the scripts.

---

## 11. Acceptance Status & Evidence Checklist

| Acceptance criterion (pack §01) | Status | Evidence / where |
|---|---|---|
| New dev can build Editor target from documented steps | ◐ documented | §1–§3, §10 — **compile log pending UE5 box** |
| CI/build scripts need no private credentials | ✅ documented | §10 |
| Module/plugin structure defined | ✅ | §2 |
| Naming, folders, Primary Asset Types, Tag hierarchy, validation | ✅ | §5–§7, §10.4 |
| Build scripts: clean compile, tests, packaged smoke | ✅ described | §10 (run on toolchain to capture logs) |
| Architecture + performance-budget docs | ✅ | this file + `Plan/00–01` |

**To close Stage 01 on a UE5 workstation:** generate `STAT.uproject` + the `*.Build.cs`/`*.Target.cs`
for the modules in §2, create the empty `L_Startup` map, run §10.1–§10.4, and paste the compile/test/
smoke logs into `Plan/02_PROGRESS_LOG.md`. Then Stage 01 flips ✅ and Stage 02 begins.

---

## 12. Next Dependency

**Stage 02 (Core Lifecycle & Data Architecture)** consumes from here: the module map (§2), the
Primary Asset Types + bases (§6), the Gameplay Tag root (§7), and the budget markers (§9). It adds the
GameInstance/World/LocalPlayer subsystem ownership map, the menu→world→menu flow, typed event/message
contracts, the `UPrimaryDataAsset` validation framework with stable IDs + schema versions, deterministic
RNG streams, and the automation-test harness. Primary lens `/function`; supporting `/dataman`, `/anatomy`.
