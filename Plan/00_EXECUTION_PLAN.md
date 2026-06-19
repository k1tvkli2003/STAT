# STAT — Master Execution Plan

> This plan turns the **prompt pack** (`Prompts/`) plus the **six skills** (`.claude/commands/`) into
> a single, ordered way of working. It says *what to build, in what order, with which skill, and what
> evidence proves a stage is done.* It does not replace the prompt pack — it drives it.
>
> Authority order when anything conflicts:
> **`Prompts/01_MASTER_PROMPT.md` → `Prompts/03_QUALITY_GATES.md` → this plan → the skills.**

---

## 0. The One-Paragraph Strategy

Prove one dense district as a **vertical slice** (drive, shoot, get chased, run one full mission,
save/load, on a packaged Windows build) before scaling anything. Implement every core rule in
**C++**; let Blueprints compose and tune. Use the six skills as **design and audit lenses** layered on
top of the engineering, applied most literally where STAT actually has screens, data, and progression:
**UI/HUD (Stage 14), VEX Protocol (Stage 18), progression & telemetry (Stage 16)**. Ship nothing a
gate can't verify.

---

## 1. How The Skills Fit A UE5/C++ Game (Read This First)

The six skills were authored against an **Android-Compose / Web** reference stack (Kotlin, Room, Hilt,
Tailwind, Framer Motion). STAT's master prompt **forbids web/React/TypeScript code** and mandates
UE5/C++. These are not in conflict if we separate the two layers of every skill:

| Layer of a skill | Use it for STAT? | How |
|---|---|---|
| **Thinking layer** — phases, diagnostic lenses, UX laws, design principles, audit discipline, prioritization, "diagnose before you touch" | **Yes, fully** | Apply the reasoning to UE5 systems, UMG/CommonUI, data assets, and design docs. |
| **Reference/code layer** — Compose snippets, Room DAOs, `BaseViewModel`, Tailwind tokens | **As analogy only** | Translate the *intent* into UE equivalents (UMG/CommonUI, `UPrimaryDataAsset`, Gameplay Tags, Enhanced Input, subsystems). Never paste Kotlin/Compose/web code into the game. |

So: a skill's **UX law, principle, lens, checklist, and anti-pattern list transfer**; its literal
Kotlin/Compose/CSS does not. Every skill output for STAT must obey
`Prompts/03_QUALITY_GATES.md` (C++ authority, save integrity, performance budgets, accessibility).

Full per-skill, per-stage mapping lives in **`Plan/01_SKILLS_AND_STAGES.md`**.

---

## 2. Phases & Milestones

The prompt pack's 20 stages roll up into five delivery phases. Each milestone is a **gate**, not a date.

```
Phase A — PROTOTYPE          Stages 01–03   → M1: Controllable character in an empty world
Phase B — VERTICAL SLICE     Stages 04–09   → M2: One district, one mission, one chase, packaged
Phase C — PRODUCTION SYSTEMS Stages 10–16   → M3: Persistent world, crew, narrative, UI, saves, progression
Phase D — SCALE & OPTIONAL   Stages 17–18   → M4: Co-op prototype + VEX Protocol (both gated, both optional)
Phase E — SHIP               Stages 19–20   → M5: Reproducible signed Shipping build + QA matrix
```

| Milestone | Definition of done (the gate) |
|---|---|
| **M1** | Editor target compiles from documented steps; menu→world→menu loop with no state leak; character controllable on KB/M + gamepad. |
| **M2** | Packaged Windows build: enter district → engage combat → drive hero vehicle → trigger + survive a pursuit → complete one mission through ≥2 approaches → save/load restores it. Measured, not claimed. |
| **M3** | District state, economy, crew, narrative, full UI/accessibility, versioned saves, and progression all persist and recover; no soft-locks. |
| **M4** | Co-op does not destabilize single-player; VEX Protocol runs from the menu with isolated saves and a visible educational disclaimer. |
| **M5** | A signed, reproducible Shipping build is produced from documented steps with a QA acceptance matrix and known-limitations report. |

**Hard rule (from `00_INDEX.md`):** do not start Phase C content scale until M2 passes.

---

## 3. The Per-Stage Execution Loop

Run this identical loop for **every** stage. It merges the master prompt's Execution Contract with the
skills' shared 5-phase shape (Audit → Diagnose → Apply → Prioritize → Present).

```
1. INSPECT   Read current modules, content, config, data assets, maps, build scripts, tests,
             profiling notes. (Skill phase 1: Audit/Trace — never build blind.)

2. DESIGN    Define ownership, lifetime, data contracts, asset contracts, threading, replication
             relevance, save impact, input/accessibility impact, performance budget.
             Invoke the stage's mapped skill(s) here in DIAGNOSE mode (§4 table).

3. BUILD     Implement complete C++ (headers + sources), editor-facing data, validators, automation
             tests, debug commands, debug visualization, docs. No TODO-hiding, no pseudocode-as-impl.

4. COVER     Handle invalid data, missing assets, streaming boundaries, save migration, input loss,
             low settings, corrupted saves, package differences, recovery states.

5. VERIFY    Compile Editor target; run relevant automation tests; profile perf-sensitive systems;
             verify packaged Windows build at milestones. Capture EVIDENCE (logs, Insights, shots).

6. GATE      Run every applicable gate in 03_QUALITY_GATES.md. A stage is incomplete until each
             applicable gate has evidence.

7. REPORT    Changed files · exact commands & evidence · assumptions · known limits · next dependency.
             Update Plan/02_PROGRESS_LOG.md.
```

**Skill rhythm inside the loop:** skills **diagnose first and stop** (their Phase 4 says present the
🔴🟡🟢 ranking, then implement only what's approved). So in step 2 a skill produces a ranked design
audit; in step 3 we implement the approved items the UE5 way.

---

## 4. Stage → Skill Map (the heart of the plan)

For each stage: its prompt-pack focus, the **primary** skill lens, **supporting** lenses, and the
single most important evidence the gate wants. (Detail + rationale: `Plan/01_SKILLS_AND_STAGES.md`.)

| # | Stage focus | Primary skill | Supporting | Key gate evidence |
|---|---|---|---|---|
| 01 | Project, modules, toolchain, budgets | `/function` | `/anatomy` | New dev builds Editor target from docs; budgets written |
| 02 | Lifecycle & data architecture | `/function` | `/dataman`, `/anatomy` | Menu↔world loop, no leak; invalid data fails validation |
| 03 | Input, camera, character, interaction | `/anatomy` | `/function`, `/style` | KB/M + gamepad controllable; prompts/glyphs swap by device |
| 04 | Combat vertical slice | `/function` | `/anatomy`, `/style` | Full encounter loop; readable at low settings |
| 05 | Vehicle vertical slice | `/function` | `/style` | Heavy-but-controllable driving; state survives save/load |
| 06 | Dense district & streaming | `/function` | `/anatomy` | Streams without major hitches; density intentional |
| 07 | Traffic, crowds, civilians | `/function` | `/dataman` | Believable reactions within CPU budget; feeds heat |
| 08 | Police heat & pursuit | `/function` | `/ideas`, `/anatomy` | Chase starts→escalates→searches→resolves; no unfair spawns |
| 09 | Mission framework | `/anatomy` | `/function`, `/dataman` | One mission, ≥2 approaches, systemic not hacked |
| 10 | Factions, economy, consequences | `/dataman` | `/function`, `/gamify` | Auditable economy; meaningful but no soft-lock |
| 11 | Crew & safehouse | `/dataman` | `/ideas`, `/anatomy` | Crew matters without spreadsheet micromanagement |
| 12 | Narrative, dialogue, cinematics | `/dataman` | `/style`, `/anatomy` | Cinematics don't break save/input/streaming; subtitles complete |
| 13 | Time, weather, audio, VFX, destruction | `/style` | `/function` | Atmosphere aids readability; low settings keep feedback |
| 14 | **UI, map, phone, HUD, accessibility** | `/anatomy` + `/style` | `/function` | Readable & controllable across PC display modes; a11y persists |
| 15 | Save, checkpoints, migration, recovery | `/function` | `/dataman` | Slice restores accurately; corrupt saves recover safely |
| 16 | **Progression, rewards, telemetry** | `/gamify` | `/function`, `/dataman` | Rewards un-farmable & idempotent; telemetry consented/disableable |
| 17 | Optional co-op | `/function` | `/ideas` | Co-op doesn't destabilize single-player |
| 18 | **VEX Protocol** | `/gamify` + `/dataman` | `/anatomy`, `/style`, `/function` | Isolated from campaign; clearly educational; deterministic scoring |
| 19 | Performance, scalability, stability | `/function` | — | Measured captures meet budgets; soak runs don't leak |
| 20 | Pipeline, QA, packaging, release | `/dataman` | `/function` | Reproducible signed build; QA acceptance matrix |

> The three **bold** stages (14, 16, 18) are where the skills apply most literally — real screens,
> real progression, real datasets. Everywhere else the skills are diagnostic lenses over C++ systems.

---

## 5. Skill Invocation Protocol

When a stage reaches step 2 (DESIGN) of the loop, invoke its mapped skill with a STAT-scoped argument
and treat the output as a **ranked design brief**, then implement the approved items as UE5/C++.

```
/anatomy   the pursuit HUD + mission objective stack for the slice district
/style     the heat/wanted meter and damage feedback language (light/dark, colorblind-safe)
/function  trace: does completing an objective actually write mission state + reward + save?
/ideas     signature "heat" moments that make STAT's chases feel different from the genre
/dataman   the VEX Protocol clinical case schema + first validated case set
/gamify    the campaign progression ledger: what is rewarded, capped, and un-farmable
```

Rules:
- A skill **diagnoses, ranks (🔴🟡🟢), and stops.** Implementation happens after the ranking is accepted.
- Skill output is **subordinate to the gates.** If a skill suggests something that violates C++
  authority, save integrity, determinism, or a perf budget, the gate wins — record the trade-off.
- Pairing chains (from the skills themselves): `/ideas` → `/anatomy` → `/style` → `/function`;
  `/gamify` and `/dataman` plug in wherever progression or content data is involved.

---

## 6. Cross-Cutting Tracks (run continuously, not once)

These never "finish"; they get re-verified every stage:

- **Determinism** — scored/saved/replayed/synced/VEX systems use stable event ordering + seeded RNG.
- **Save integrity** — every save-affecting system: stable IDs, schema version, migration, corruption recovery.
- **Performance** — every perf-sensitive system: budget + stat group/trace marker + debug view + capture.
- **Accessibility** — KB/M + gamepad + remap + glyphs + subtitles + colorblind-safe + assists + text scale.
- **Localization-ready** — all player-facing text, tool labels, comments, docs in English, externalized.
- **VEX isolation** — VEX never mutates campaign state/fiction; always labeled educational-only.

---

## 7. Governance & Anti-Patterns (project-wide red lines)

From the master prompt + quality gates, enforced every stage:

- ❌ No city-scale expansion before the slice (M2) passes.
- ❌ No core rule that exists only in Blueprint with no testable C++.
- ❌ No fabricated benchmarks, fake assets, or pseudocode presented as implementation.
- ❌ No risky work hidden behind `TODO`.
- ❌ No `fallbackToDestructiveMigration`-style save wipes; migrations are tested and append-only.
- ❌ No web/React/TypeScript code in the game (skills' web snippets are analogy only).
- ❌ VEX never presented as clinical decision support.

---

## 8. Definition Of Done (whole project)

STAT is done only when the packaged Windows build proves the dense-district vertical slice is **fun,
stable, performant, save-safe, accessible, and extensible**; the optional VEX Protocol remains
isolated and clearly educational; and every expansion system carries validation, profiling, and
recovery evidence. (Mirrors `Prompts/00_INDEX.md` and `03_QUALITY_GATES.md`.)

---

## 9. Current Status

| Item | State |
|---|---|
| Skills installed (`.claude/commands/`) | ✅ Done |
| Source pack imported (`Prompts/`, `OVERVIEW.md`) | ✅ Done |
| Execution plan (this doc + `01_SKILLS_AND_STAGES.md`) | ✅ Done |
| **Stage 01 — Project foundations** (`Docs/Stage01_Foundations.md`) | ✅ First execution pass (design-complete; UE5 binaries needed to compile) |
| Stage 02 → 20 | ⏳ Queued — proceed per `Plan/02_PROGRESS_LOG.md` |

**Environment limit, stated honestly:** this container has no Unreal Engine toolchain, so stages that
require *compiling/packaging* cannot produce build logs here. Those steps are authored as
ready-to-run plans and design-complete specs; the compile/package/profile **evidence** is captured on
a UE5 workstation/CI. Everything platform-independent (architecture, conventions, data schemas, design
audits, budgets) is produced as real, reviewable artifacts now.
