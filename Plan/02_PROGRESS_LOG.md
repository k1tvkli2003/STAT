# STAT — Progress Log

Living tracker for the 20-stage build. Update on every stage transition with: state, evidence, skill
used, assumptions, known limits, next dependency. Legend: ✅ done · 🟦 in progress · ⏳ queued · 🚫 blocked.

---

## Setup (pre-Stage)

| Item | State | Evidence |
|---|---|---|
| Extract & read both archives (STAT pack + commands) | ✅ | 5 pack files + 6 skill files read in full |
| Install 6 skills as project slash commands | ✅ | `.claude/commands/{anatomy,style,function,ideas,dataman,gamify}.md` |
| Import source pack into repo | ✅ | `Prompts/00–03`, `OVERVIEW.md` |
| Master execution plan | ✅ | `Plan/00_EXECUTION_PLAN.md` |
| Skills × stages integration | ✅ | `Plan/01_SKILLS_AND_STAGES.md` |
| Repo README + UE5 `.gitignore` | ✅ | `README.md`, `.gitignore` |

---

## Phase A — Prototype (Stages 01–03) → M1

### Stage 01 — Project, Modules, Toolchain, Budgets — 🟦 design-complete
- **Skill used:** `/function` (system spine) + `/anatomy` (module boundaries as IA).
- **Delivered now (real artifacts):** `Docs/Stage01_Foundations.md` — module/plugin map, coding
  standards, asset naming + folder conventions, Primary Asset Types, Gameplay Tag root hierarchy,
  performance & minimum-spec budgets, CI/build guidance, source-control/LFS rules.
- **Needs a UE5 workstation to close:** generate the actual `.uproject` + `*.Build.cs` + `*.Target.cs`,
  compile the Editor target, run the empty-map smoke, attach build logs.
- **Acceptance (from pack):** a new dev can build the Editor target from documented steps; CI needs no
  private credentials. → Documented; **compile evidence pending toolchain.**
- **Assumptions:** UE 5.4+; MSVC/VS 2022; Windows 11 dev box; Git LFS available.
- **Known limit:** no engine in this container → no compile/package log produced here.
- **Next dependency:** Stage 02 consumes the module map + Primary Asset Types + Gameplay Tag root.

### Stage 02 — Core Lifecycle & Data Architecture — 🟦 design-complete
- **Skill used:** `/function` (lifecycle wiring, single source of truth, resilience) + `/dataman`
  (data-asset validation framework) + `/anatomy` (front-end→world flow).
- **Delivered now:** `Docs/Stage02_LifecycleAndData.md` — subsystem ownership map (one owner per
  fact), game-flow state machine + transitions, typed message bus contracts, world clock + named
  deterministic RNG streams, `USTATPrimaryDataAsset` base + validation framework (actionable
  messages) + registry, debug commands, automation harness spec, tested teardown contract for
  zero state leak.
- **Needs UE5 box to close:** implement §2/§3/§6 as full `.h/.cpp`, author fixtures, run §8 specs.
- **Acceptance:** menu↔world no-leak + invalid-data-fails-loudly → design-complete & testable;
  **run evidence pending toolchain.**
- **Next dependency:** Stage 03 consumes input/profile subsystems, message bus, clock, flow states.

### Stage 03 — Input, Camera, Character, Interaction — 🟦 design-complete
- **Skill used:** `/anatomy` (invoked) — interaction/input structure & reachability; support
  `/function`, `/style`.
- **Delivered:** `Docs/Stage03_InputCameraCharacter.md`.

---

## Phase B — Vertical Slice (Stages 04–09) → M2  🟦 in progress

### Stage 04 — Combat Vertical Slice — 🟦 design-complete
- **Skill used:** `/function` (invoked) — wiring integrity, single source of truth, integration
  contracts, state completeness, game-thread discipline, resilience; support `/anatomy`, `/style`.
- **Delivered:** `Docs/Stage04_Combat.md` — GAS attribute authority (one ASC AttributeSet for
  health/armor/ammo; event-driven HUD, no polling), every combat input wired to a real ability,
  the kill→`EncounterDirector` resolution contract + cross-feature hook interfaces, encounter state
  machine with the Lull(Empty) and Recover(no soft-lock) arms, resilience guards (reload re-entrancy,
  dry-fire, valid spawns, seeded damage RNG, save-mid-encounter), game-thread/tick budgets, and the
  cover/AI/EQS/director/arena assembly.
- **Needs UE5 box to close:** implement GAS abilities/attribute set, AI StateTree+EQS, the arena map;
  run the 8 automation tests + an Insights capture vs the AI/frame budgets.
- **Keystone:** kill→director resolution contract (encounter must actually reach "Won").
- **Next dependency:** Stage 05 reuses the GAS pattern, threat/pursuit hooks, input contexts, save/RNG.

### Stage 05 — Vehicle Vertical Slice — 🟦 design-complete
- **Skill used:** `/function` (applied) — single vehicle-state authority, wiring, integration
  contracts, no-soft-lock state arms, **physics sub-stepping for frame-rate-independent handling**,
  resilience; support `/style` (drive-feel hooks, Stage 13).
- **Delivered:** `Docs/Stage05_Vehicle.md` — Chaos pawn + authoritative state component, data-driven
  tuning, input→movement wiring, enter/exit context+camera handoff, Disabled arm with exit/respawn,
  async-physics handling stable at 30/60fps, surface response, garage spawn + save persistence, hero
  vehicle, tests + Insights plan.
- **Keystone:** async physics + sub-stepping (handling identical at 30 vs 60 fps).
- **Needs UE5 box:** tune the hero vehicle, run physics/latency/save tests + captures.

### Stage 06 — Dense District & Streaming — 🟦 design-complete
- **Skill used:** `/function` (applied) — streaming hitch = jank, velocity prefetch, memory budget,
  streamed save restore; support `/anatomy` (district legibility, landmarks, route IA).
- **Delivered:** `Docs/Stage06_DistrictStreaming.md` — `L_District_Slice` (WP + Data Layers), safehouse/
  combat space/pursuit routes/landmarks, **velocity-based streaming prefetch** so the vehicle never
  outruns streaming, HLOD/nav/occlusion, traversal metrics (foot/vehicle/chase/mission), lighting
  scenarios, content-density budget gate, streaming walk/drive tests + Insights captures.
- **Keystone:** velocity-based streaming prefetch (no felt hitch at speed).
- **Needs UE5 box:** build the map + HLOD/nav/lighting, capture drive-loop streaming evidence.

### Stage 07 — Traffic, Crowds & Civilian Reactions — 🟦 design-complete
- **Skill used:** `/function` (applied) — single-owner spawn budget, **witness→report→heat contract**,
  significance/LOD/pooling, stuck-recovery + cleanup resilience, "Mass only where profiling wins";
  support `/dataman` (spawn tables, zones, archetypes).
- **Delivered:** `Docs/Stage07_TrafficCrowds.md` — `USTATPopulationSubsystem` sole spawn authority +
  density tiers, traffic lane/intersection/parking + stuck recovery, civilian state machine with the
  panic/flee/**report** arms wired to the heat system, significance/pooling + pool-return-on-unload
  (no leaks), debug heatmaps, validated data catalogs, density-tier Insights plan.
- **Keystone:** witness→report→heat contract (the city is "dead" to police without it).
- **Needs UE5 box:** build spawners + state machine, capture Low/Med/High density vs CPU budget.

08 Police heat/pursuit · 09 Mission — ⏳ queued.
Gate M2 = packaged Windows build proving the full slice loop, measured.

## Phase C — Production Systems (Stages 10–16) → M3  ⏳ queued
10 Factions/economy · 11 Crew/safehouse · 12 Narrative/cinematics · 13 Time/weather/audio/VFX ·
14 UI/map/phone/HUD/a11y · 15 Save/migration · 16 Progression/telemetry.

## Phase D — Scale & Optional (Stages 17–18) → M4  ⏳ queued
17 Co-op (only after slice stable) · 18 VEX Protocol (isolated, educational).

## Phase E — Ship (Stages 19–20) → M5  ⏳ queued
19 Performance/stability · 20 Pipeline/QA/packaging/release.

---

## Decisions & Trade-offs (append-only)

- **2026-06-19 — Skills installed as slash commands, not Agent Skills.** Files use `$ARGUMENTS` and
  cross-reference as `/name`, which is the slash-command format → `.claude/commands/`. They activate
  as `/anatomy` … `/gamify` in new sessions in this repo.
- **2026-06-19 — Skill code layer is analogy-only for STAT.** Master prompt forbids web/React/TS and
  mandates C++ authority; skills are used as thinking lenses (see `Plan/01_SKILLS_AND_STAGES.md` §A).
- **2026-06-19 — Compile/package/profile evidence deferred to a UE5 box/CI.** No Unreal toolchain in
  this environment; all platform-independent artifacts produced now, build evidence captured later.
