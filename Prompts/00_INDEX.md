# STAT Prompt Pack

Use this pack to build **STAT** — a standalone, fully-3D, AAA-scalable game whose entire world is one
**living hospital**, shipping from a single **Unreal Engine 5 / C++** codebase to **Windows, macOS,
Linux, PlayStation, Android, and iOS**, scaling from high-end AAA fidelity down to mid-range PCs and
mobile. Core gameplay is clinical reasoning under pressure, powered by the **VEX clinical engine**.

> STAT is **educational and entertainment only** — never clinical decision support or medical advice.

## The Skills Are The Standards

The six skills in `.claude/commands/` are this project's **governing protocols and acceptance
criteria**, not optional advice. Their reference code is Android/Web; for STAT (UE5/C++) their
**thinking layer transfers fully** — laws, lenses, checklists, and zero-tolerance anti-patterns.

| Skill | Governs | Its protocol (used throughout) |
|---|---|---|
| `/function` | behavior, wiring, performance | trace → diagnose (wiring integrity, single source of truth, integration contracts, state completeness, main/game-thread, resilience) → rank 🔴🟡🟢 → implement · measure before optimize |
| `/anatomy` | structure, flow, reachability | audit → diagnose with UX laws (Fitts, Hick, Miller, Jakob, information scent, progressive disclosure, cognitive load, reach) → restructure → no dead ends |
| `/style` | visual system, motion, a11y | audit design DNA → principles (WCAG contrast, hierarchy, gestalt, rhythm, motion-with-meaning, depth) → tokens → light/dark + localization |
| `/ideas` | feature quality | investigate → 7 lenses → 5 tiers → kill criteria + idea quality bar |
| `/dataman` | data & medical cases | audit → brief → expand in layers (repair/complete/enrich/stress/document) → validate (IDs, refs, provenance, no fake-as-real) |
| `/gamify` | motivation, scoring, anti-abuse | motivation lenses (competence/autonomy/relatedness/curiosity/commitment/status/flow) → event-ledger → idempotency, caps, cooldowns · reward mastery, never grind |

## How To Execute

1. Keep `01_MASTER_PROMPT.md` active throughout every implementation session.
2. Execute `02_BUILD_SEQUENCE.md` in order; each numbered section is a separate prompt and milestone.
3. **Before implementing a stage, invoke its governing skill(s)** (Stage Map below) to produce a ranked
   design/audit; implement the approved items the UE5 way.
4. Enforce `03_QUALITY_GATES.md` — which encodes the skills' criteria and anti-patterns — before
   accepting a stage.
5. End each stage with changed files, build/test/profile evidence (per platform tier where relevant),
   assumptions, known limits, and the next dependency.
6. **Prove the vertical slice first.** Do not scale hospital size, systems, content, co-op, or optional
   modes before the slice proves core fun, cross-platform performance, save integrity, accessibility,
   and stability.

## Delivery Strategy

`Prototype -> Vertical Slice -> Production Systems -> Content Scale -> Shipping`

The **vertical slice** = one hospital wing (ER + one department), a complete **VEX clinical loop**
(investigate → diagnose → treat → consequence), representative patients/staff, one emergent crisis
event, core save/load, deep accessibility, and a stable packaged build on **at least one desktop tier
and one mobile tier** — proving scalability, not just Windows.

## Non-Negotiables

- All six platforms from **one UE5 / C++ codebase**; C++ authority for core simulation, clinical,
  save, economy, telemetry, online, and performance-sensitive rules. Blueprints compose/tune/present.
- **Scalability is a first-class feature:** explicit fidelity tiers so the same game runs on mobile and
  mid-range as well as high-end. Every perf-sensitive system carries a budget per tier.
- The whole world is a **hospital** — fully 3D, authentic, original art.
- Every system defines ownership, lifetime, threading, replication relevance, save impact, data
  contracts, validation, debug tooling, per-tier performance budget, input surfaces (KBM/gamepad/touch),
  and accessibility impact.
- STAT is **educational/entertainment only**; clinical content is validated and responsibly framed,
  never presented as clinical decision support.
- Player-facing text, tools, comments, and docs stay in English and localization-ready.

## Stage Map

| Stage | Focus | Governing skill(s) | Completion Signal |
|---|---|---|---|
| 01 | Project, modules, multi-platform toolchain, scalability tiers, budgets | `/function` | Editor + one desktop + one mobile target compile from docs |
| 02 | Core lifecycle & data architecture (deterministic, save, validation) | `/function`,`/dataman` | Menu↔hospital loop, no leak; invalid data fails loudly |
| 03 | Player presence: input (KBM/gamepad/**touch**), camera, character, 3D interaction | `/anatomy`,`/function` | Fully controllable on desktop + mobile; device-aware prompts |
| 04 | The hospital space: one fully-3D hospital, streaming by floor/wing, navigation | `/function`,`/anatomy` | Wing streams without hitch on target tiers; legible layout |
| 05 | Patient & staff simulation (agents, conditions, routines, crowds) | `/function`,`/dataman` | Believable population within per-tier CPU budget |
| 06 | **VEX clinical engine** (case, examine, budget/clock, tests, diagnosis, scoring) | `/dataman`,`/gamify`,`/function` | One case runs end-to-end; deterministic, auditable score |
| 07 | Treatment & procedures (interventions, patient state, outcomes) | `/function`,`/dataman` | Treatment changes patient state and outcome deterministically |
| 08 | Triage & pressure director (emergent case flow, prioritization, collisions) | `/function`,`/ideas` | Cases collide and force triage without soft-locks |
| 09 | Crisis events (mass-casualty, outbreak, power failure, code blue) | `/function`,`/ideas` | One crisis escalates, resolves, and is recoverable |
| 10 | Resources, reputation, economy, consequences | `/dataman`,`/function`,`/gamify` | Auditable economy; meaningful consequences, no soft-lock |
| 11 | Roles, departments, progression, unlocks | `/gamify`,`/anatomy` | Roles change play; unlocks broaden, don't gate essentials |
| 12 | Narrative, dialogue, patient stories, cinematics, ethics | `/dataman`,`/style`,`/anatomy` | Cinematics don't break save/input/streaming; subtitles complete |
| 13 | Atmosphere: time, ambience, audio, VFX, lighting | `/style`,`/function` | Atmosphere aids readability; low tier keeps feedback |
| 14 | **UI/HUD/charts/menus/map/accessibility** (CommonUI: touch+gamepad+KBM) | `/anatomy`,`/style` | Readable & controllable on every platform; a11y persists |
| 15 | Save/checkpoint/migration/recovery (+ cross-platform/cloud) | `/function`,`/dataman` | Slice restores accurately; corrupt saves recover safely |
| 16 | Progression/rewards/challenge/telemetry (educational metrics, anti-farming) | `/gamify`,`/function` | Rewards un-farmable & idempotent; telemetry consented/disableable |
| 17 | Optional co-op (a colleague joins) | `/function` | Co-op doesn't destabilize single-player |
| 18 | Content pipeline: medical case corpus + art assets + validation | `/dataman` | Validated, provenance-tracked case + asset catalogs |
| 19 | Performance, scalability & stability across ALL platforms | `/function` | Measured captures meet per-tier budgets incl. mobile |
| 20 | Platform packaging, QA & release (per-platform incl. console) | `/dataman`,`/function` | Reproducible builds + QA matrix for each platform |

## Final Definition Of Done

STAT is done only when packaged builds prove the hospital vertical slice is **fun, stable, performant,
save-safe, accessible, and extensible on every platform tier — from mobile to high-end**; the VEX
clinical engine is deterministic, auditable, and responsibly framed as educational only; and every
system passes the governing skills' criteria and zero-tolerance anti-patterns with validation,
profiling, and recovery evidence.
