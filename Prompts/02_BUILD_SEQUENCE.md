# STAT Build Sequence

Each section is an independent implementation prompt governed by `01_MASTER_PROMPT.md`. **Invoke the
stage's governing skill(s) first** for a ranked design/audit, then implement. Apply `03_QUALITY_GATES.md`
before accepting a stage. Everything ships from one UE5/C++ codebase to all six platforms with explicit
scalability tiers.

---

## 01 - Project, Modules, Multi-Platform Toolchain, Scalability Tiers, Budgets
**Governing:** `/function` (system spine, budgets, measure-before-optimize).

Create the UE5 C++ project; runtime/editor/test modules; plugin boundaries (hospital sim, agents, VEX
clinical, treatment, triage/crisis, economy, roles, narrative, UI, audio, save, telemetry, content,
tools, online); coding rules; **per-platform targets (Win/macOS/Linux/PlayStation/Android/iOS)**;
device-profile **Scalability tiers** (mobile → high-end); Asset Manager rules; Gameplay Tags; CI/build
scripts; crash symbols; and explicit **per-tier** CPU/GPU/frame-time/memory/streaming/package-size/
load-time budgets with a documented minimum spec per platform.

Deliverables: compiling Editor + one desktop + one mobile target and an empty startup map; module/plugin
structure; naming/folder conventions, Primary Asset Types, Gameplay Tag hierarchy, validation commands;
build scripts (clean compile, tests, packaged smoke on desktop + mobile); scalability-tier definitions;
architecture + per-tier performance-budget docs.

Acceptance: a new dev builds desktop **and** mobile targets from documented steps; CI needs no private
credentials; PlayStation build path is architected and documented (built later on licensed SDK/hardware).

---

## 02 - Core Lifecycle & Data Architecture
**Governing:** `/function` (lifecycle wiring, single source of truth, no leak) + `/dataman` (validated data).

Define game flow, subsystem ownership, typed event/message contracts, sim clock, deterministic RNG
streams, save IDs, Primary Data Asset bases + validation framework (stable IDs, schema versions,
actionable messages), debug commands, and the automation-test harness. Deliver an empty but robust
front-end → hospital → front-end loop.

Deliverables: GameInstance/World/LocalPlayer subsystem ownership map (one owner per fact); menu → hospital
→ menu flow with a tested teardown contract (zero state leak); message bus; `UPrimaryDataAsset` bases +
validators + registry; seeded RNG; automation tests (lifecycle, invalid data, RNG determinism).

Acceptance: menu ↔ hospital with **no leaked state**; invalid data assets fail validation with
**actionable** messages.

---

## 03 - Player Presence: Input, Camera, Character, 3D Interaction
**Governing:** `/anatomy` (interaction structure, reachability, no dead ends) + `/function` (wiring, no bleed).

Enhanced Input for **KBM + gamepad + touch**, remapping, context switching (Explore / Examine / Menu /
Cinematic), third-person (and/or first-person) camera with collision, locomotion, a **single-winner
interaction scanner** (prioritized prompts, hysteresis to prevent flicker), accessibility assists, and
deterministic interaction tests.

Deliverables: input actions/contexts/glyphs/remap/sensitivity; **touch controls** for mobile; camera
orbit/collision; interaction scanner with priority + anti-flicker; controller/touch **default focus**
never empty; accessibility assists (hold↔toggle, assist toggles, buffering); tests (mapping, context
switch, interaction priority, device-swap prompts).

Acceptance: fully controllable on **desktop and mobile**; prompts/glyphs swap with the active device;
no input bleed across context swaps.

---

## 04 - The Hospital Space: Fully-3D Hospital, Streaming, Navigation
**Governing:** `/function` (streaming hitch = jank, per-tier budget) + `/anatomy` (legible layout, wayfinding).

One production-quality **fully-3D hospital wing** via World Partition + Data Layers (floors/wings/
departments), HLOD, navigation, occlusion, streaming sources, lighting scenarios, and automated
streaming walks — tuned so it streams without hitches on **each target tier including mobile**.

Deliverables: the wing map (ER, one department, corridors, rooms, landmarks) with legible wayfinding;
WP/Data Layer/HLOD/nav/occlusion/source setup; **per-tier** streaming budget + velocity/route prefetch;
automated walk tests + Insights captures on desktop and mobile; content-density budget before expansion.

Acceptance: streams without major hitches on target hardware **per tier**; layout is legible and density
intentional before the hospital scales.

---

## 05 - Patient & Staff Simulation
**Governing:** `/function` (population within per-tier CPU budget, single owner) + `/dataman` (agent data).

Scalable NPC agents — patients, staff, visitors — with conditions, routines, reactions, significance/LOD,
pooling, stuck recovery, and cleanup on streaming unload. Use Mass **only where profiling proves value**.

Deliverables: a single population authority with per-tier caps; patient/staff/visitor archetypes +
behavior state machines (with panic/evacuate/report arms wired to crisis + triage systems); significance/
pooling/cleanup (no leaked actors); debug heatmaps; validated data catalogs; per-tier density captures.

Acceptance: population reacts believably within the **per-tier** CPU budget; patient condition changes
feed the clinical + triage systems.

---

## 06 - The VEX Clinical Engine (Core Loop)
**Governing:** `/dataman` (case corpus + validation) + `/gamify` (scoring/mastery) + `/function` (determinism).

The heart of the game: a **budget-and-clock-limited clinical loop** — intake → examine in 3D → purchase
labs/imaging/consults from a finite budget with realistic delays → commit to a diagnosis → debrief/score.
Deterministic and auditable.

Deliverables: versioned `VexCase` Primary Data Assets (presentation, findings, budgeted investigations,
delayed results, correct diagnosis set, safety penalties, reviewer metadata, references, correction
history); the examination + test-shop + results-timeline + diagnosis-commit runtime; a deterministic
**event ledger** of purchases/results/decisions/timing; a scoring model rewarding accuracy, appropriate
investigation, budget retained, and avoided harm (never guessing/waste); one full case end-to-end; tests
for scoring, invalid cases, determinism, and disclaimer visibility.

Acceptance: one case runs intake→diagnosis→debrief; the same seed reproduces the same score; content is
validated and clearly **educational only**; unsafe/wasteful actions are penalized, not rewarded.

---

## 07 - Treatment & Procedures
**Governing:** `/function` (state correctness, no soft-lock) + `/dataman` (procedure/outcome data).

Data-driven treatments and 3D procedures that change patient state and produce outcomes — with every
state arm modeled (stable → deteriorating → treated → complication → recovered/failed) and no dead ends.

Deliverables: treatment/procedure Data Assets; patient-state model + deterministic outcome resolution;
complication + recovery arms (no soft-lock); integration contracts (treatment → patient state → VEX
score → economy/reputation → narrative); tests for outcomes, edge cases, and save-mid-procedure.

Acceptance: treatment deterministically changes patient state and outcome; a failed/complicated case
always has a defined, non-soft-locking continuation.

---

## 08 - Triage & Pressure Director
**Governing:** `/function` (the collision loop) + `/ideas` (signature pressure moments).

The systemic pressure engine: an emergent flow of cases/patients that **collide** and force triage and
prioritization — the game's "missions collide" tension, medicalized.

Deliverables: a triage/pressure director that paces intake, deterioration, and resource contention;
prioritization surfacing (who first, what now); pressure tiers by difficulty; signature emergent moments
(`/ideas`); tests for overload, starvation, fairness, and recovery.

Acceptance: cases collide and force real triage decisions without soft-locks; pressure is tunable and
readable.

---

## 09 - Crisis Events
**Governing:** `/function` (escalate/resolve/recover) + `/ideas` (strong, distinct crises).

Set-piece emergent crises — mass-casualty intake, outbreak, power failure, code blue — that escalate,
resolve, and are always recoverable, reusing simulation/clinical/economy systems (no bespoke hacks).

Deliverables: a crisis framework + at least one full crisis; escalation → response → resolution →
aftermath state; integration with population, resources, reputation, and save; debug triggers; tests for
escalation, resolution, recovery, and save/load mid-crisis.

Acceptance: one crisis escalates, is handled, resolves, and recovers with no soft-lock; it uses systems,
not one-off scripting.

---

## 10 - Resources, Reputation, Economy, Consequences
**Governing:** `/dataman` (ledgers) + `/function` (auditable) + `/gamify` (meaningful, non-abusable).

Model beds/supplies/staff-time/budget, hospital reputation, and persistent consequences of outcomes —
auditable, deterministic where scored, and free of irreversible soft-locks.

Deliverables: resource + reputation models with stable save IDs; an auditable economy ledger
(sources/sinks); consequence events affecting resources, case availability, department access, and
patient trust; soft-lock prevention + recovery; tests for transactions, migration, thresholds, gating.

Acceptance: consequences are meaningful without permanently trapping the player; economy is auditable and
deterministic where scored.

---

## 11 - Roles, Departments, Progression, Unlocks
**Governing:** `/gamify` (progression that broadens play) + `/anatomy` (role/department flow).

Playable roles (attending / resident / paramedic), unlockable departments, and progression that
**broadens** play rather than inflating numbers or gating essentials.

Deliverables: role definitions changing verbs/access; department unlock structure (information scent,
reachable); progression tied to mastery, not grind; assignment/selection flow; tests for role effects,
unlock gating (no essential lockout), and save/load.

Acceptance: roles meaningfully change play; unlocks broaden without hiding essential functionality behind
game gates.

---

## 12 - Narrative, Dialogue, Patient Stories, Cinematics
**Governing:** `/dataman` (dialogue/subtitle data) + `/style` (cinematic language) + `/anatomy` (flow).

Narrative state, dialogue conditions, patient stories with continuity, ethics beats, subtitles, Sequencer
conventions, skip/replay, camera safety, and cinematic streaming — localization-ready.

Deliverables: narrative-state + dialogue-condition system; subtitle/localization data contracts; patient
story continuity; Sequencer conventions + skip/replay + streaming policy; intro/outro for the slice; tests
for missing subtitles, invalid IDs, save-during-cinematic, skipped sequences.

Acceptance: cinematics never break save/input/streaming; all spoken/important content has subtitles;
patient stories persist coherently.

---

## 13 - Atmosphere: Time, Ambience, Audio, VFX, Lighting
**Governing:** `/style` (mood, readability, motion) + `/function` (per-tier budgets).

Time-of-day, ambient hospital life, MetaSounds ambience/music states, a diegetic audio mix, Niagara
budgets, and lighting scenarios — atmosphere that **improves readability** and holds on the mobile tier.

Deliverables: time/ambience state; audio state system (ambience, tension, procedure, crisis, UI,
accessibility mix); VFX/lighting budgets + per-tier scalability; motion-with-meaning for key state
changes; profiling + debug toggles.

Acceptance: atmosphere aids readability rather than obscuring it; the low/mobile tier preserves essential
feedback.

---

## 14 - UI / HUD / Charts / Menus / Map / Accessibility
**Governing:** `/anatomy` (structure, reachability) + `/style` (visual system, a11y).

CommonUI front end, HUD, the patient **chart** (the core information surface), test-shop, results
timeline, department map, menus, and settings — controllable on **touch + gamepad + KBM**, WCAG-clean,
light/dark, localization/RTL-ready.

Deliverables: CommonUI shell (menu, pause, settings, save/load, profile); the patient chart + clinical
HUD (vitals, budget, clock, findings, objectives); map + department navigation; accessibility (subtitles,
colorblind-safe, contrast, text size, hold/toggle, reduced motion, remap, aim/read assists); tests across
touch/gamepad/KBM, aspect ratios, scaling, pause, settings persistence.

Acceptance: UI is readable and controllable on **every** platform and input surface; accessibility
options persist and take effect; no `/anatomy` or `/style` anti-pattern present.

---

## 15 - Save / Checkpoint / Migration / Recovery
**Governing:** `/function` (atomic, versioned, recoverable) + `/dataman` (schema/migration).

Versioned snapshots, stable object IDs, async atomic saves, checkpoint scope, streamed-world restoration,
corruption fallback, migration tests, multiple slots, a **cross-platform / cloud** save abstraction, and
explicit autosave indicators.

Deliverables: save schema (stable IDs, versioning, migration, subsystem ownership); async/atomic writes,
slots, autosave indicators, corruption fallback; checkpoint policy (cases, treatment, crisis, economy,
reputation); cross-platform save shape + cloud abstraction; tests for migration, corruption, interrupted
save, streamed restore, multi-slot, cross-platform load.

Acceptance: the slice restores accurately; corrupt saves fail safely with visible recovery; saves are
portable across platforms where feasible.

---

## 16 - Progression / Rewards / Challenge / Telemetry
**Governing:** `/gamify` (ledger, anti-abuse) + `/function` (idempotent wiring).

Progression, challenges, achievements, difficulty assists, **educational/mastery metrics**, and
privacy-aware telemetry — with idempotent rewards and anti-farming.

Deliverables: an event-ledger progression system (idempotent reward events, source attribution, daily
caps, cooldowns, audit log); challenges/achievements/case grading/assists; a telemetry schema (consent,
PII minimization, offline buffering, debug viewer); an economy/mastery dashboard; tests for duplicate
rewards, caps, challenge completion, difficulty effects, telemetry consent.

Acceptance: rewards can't be farmed through retries or sync duplication; telemetry is consented and
disableable without breaking gameplay; progression rewards mastery, never unsafe/wasteful play.

---

## 17 - Optional Co-op (a colleague joins)
**Governing:** `/function` (authority, replication, no single-player destabilization).

Only after the single-player slice is stable: define server authority, session flow, replication/
relevancy, prediction boundaries, join/leave, role ownership, disconnect recovery, anti-cheat posture,
and network profiling for a **second player joining as a colleague**.

Deliverables: a co-op feasibility ADR + scope boundary; session/authority/replication/ownership
contracts; a narrow prototype (one supported role) if slice-stable; disconnect/reconnect + save
compatibility; network profiling + anti-cheat notes.

Acceptance: co-op does not destabilize single-player architecture; ownership decisions are explicit, not
blind retrofits.

---

## 18 - Content Pipeline: Case Corpus + Art Assets + Validation
**Governing:** `/dataman` (audit → brief → expand → validate; Jules delegation).

Grow the validated **clinical case corpus** and the original 3D art library (environments, patients,
staff) with naming rules, provenance, reviewer metadata, and automated validation. Large data creation
may be Jules-delegated **after a written brief**, validated locally before merge.

Deliverables: a content validation suite + asset QA checklist; expanded, validated case corpus
(diversity, edge cases, difficulty distribution, provenance, reviewer tags — synthetic-illustrative,
never presented as real); art-asset specs + validation; a delegation brief template.

Acceptance: case + asset catalogs are unique-ID'd, reference-clean, provenance-tracked, distribution-
balanced, and pass validation; no fake-as-real data; no secrets committed.

---

## 19 - Performance, Scalability & Stability (All Platforms)
**Governing:** `/function` (measured, per-tier, no leaks).

Profile CPU/GPU/memory/IO/shaders/streaming/hitches/AI/audio/UI/save across **every platform tier
including mobile**. Define scalability tiers, PSO strategy, automated soak routes, leak checks, crash
recovery, and per-platform minimum/recommended-spec evidence.

Deliverables: Insights captures for slice journeys on desktop **and** mobile; finalized scalability tiers
+ per-platform settings; automated soak/streaming/crisis routes; PSO/shader strategy, hitch/memory
reports, crash triage; a stability report with thresholds.

Acceptance: measured captures meet documented **per-tier** budgets (including a mobile tier); soak runs
don't leak or hitch beyond accepted thresholds; performance is measured, not guessed.

---

## 20 - Platform Packaging, QA & Release (per-platform incl. console)
**Governing:** `/dataman` (validation/QA matrices) + `/function` (reproducible builds).

Validators, naming rules, functional test plans, a save-compatibility matrix, localization pipeline,
BuildGraph/CI, signed packaging **per platform**, patch/chunk strategy, crash reporting, store/console
compliance boundaries, release docs, and reproducible builds for every target.

Deliverables: content-validation suite + asset QA checklist; a functional test plan (slice, VEX, saves,
crises, UI, accessibility, per platform); localization pipeline + text audit; BuildGraph/CI packaging +
signing notes per platform (console via licensed SDK/hardware); patch/chunk plan; crash-reporting hooks;
reproducible packaged builds + a release README.

Acceptance: reproducible packaged builds are produced from documented steps for each platform (console on
licensed toolchain); QA has a clear per-platform acceptance matrix and known-limitations report.
