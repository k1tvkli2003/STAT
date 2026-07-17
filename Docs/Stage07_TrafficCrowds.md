# Stage 07 — Traffic, Crowds & Civilian Reactions

> **Status:** design-complete (architecture + budgets authored). Building spawners, the civilian state
> machine, and density/Insights captures happen on a UE5 workstation/CI — §8.
> **Skill lens:** `/function` **(applied)** — population within CPU budget, single owner, the
> witness→report integration contract, significance/pooling, stuck-recovery resilience, **"Mass only
> where profiling demonstrates value"** — with `/dataman` (spawn tables, zones, archetypes) supporting.
> **Governs:** `Prompts/01_MASTER_PROMPT.md`; gated by `Prompts/03_QUALITY_GATES.md`.
> **Consumes:** district nav + lanes + significance headroom (Stage 06), vehicle collision (Stage 05),
> AI perception pattern + heat hook (Stage 04), budgets (Stage 01).

---

## 1. Function + Data Audit

```
╔══════════════════════════════════════════════════════════════╗
║  AUDIT — scalable traffic + crowds + reactions               ║
╠══════════════════════════════════════════════════════════════╣
║  Critical path: population mgr → budgeted spawn along lanes/  ║
║   zones → significance/LOD tick → civilian sees crime →       ║
║   panic/flee + REPORT event → feeds heat (Stage 08)          ║
║  Budget (Stage 01): AI+traffic+combat CPU ≤4ms min-spec      ║
║  Owner: ONE population manager owns the spawn budget          ║
╠══════════════════════════════════════════════════════════════╣
║  🔴 design out                                                ║
║   • Witness never emits a report → heat can't rise (the core  ║
║     integration contract; "looks alive," police stay blind)   ║
║   • Unbounded spawns blow the CPU budget (no significance/cap)║
║   • Actors leak when a streaming cell unloads (no cleanup)    ║
║  🟡 friction / resilience                                     ║
║   • Stuck cars/peds at junctions (no recovery) → fake-looking ║
║   • Pop-in/pop-out from no pooling → alloc churn + hitches    ║
║  🟢 opportunity                                               ║
║   • Mass ONLY if profiling proves it beats pooled actors      ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 2. Single Owner & Budget (Function)

- **`USTATPopulationSubsystem`** (world subsystem) is the **sole** authority for spawn budget across
  traffic + crowds — no rival spawner. It enforces concurrent caps per density tier and reclaims
  budget as actors despawn. Two spawners with two budgets is the drift bug; one owner can't disagree
  with itself.
- **Density tiers** (data-driven, `/dataman`): Low/Med/High map to caps for vehicles + pedestrians,
  selected by the scalability tier (Stage 19) and current load. Budget is **measured**, not assumed
  (markers below) — the "Mass only where profiling demonstrates value" rule means **actors + pooling
  first**, Mass adopted only if an Insights capture shows it wins for this density.

---

## 3. Traffic (lanes, intersections, parking, recovery)

- **Lane + intersection data** (`/dataman` catalogs validated by Stage 02 framework): directed lane
  graph, junction right-of-way, speed limits, parking slots.
- **Spawners** place vehicles ahead of the streaming source along lanes within budget; **despawn**
  behind it; **pooling** reuses actors (no per-spawn alloc churn — the hitch 🟡).
- **Stuck recovery (resilience):** a vehicle blocked > T seconds (world clock) re-routes or is recycled
  — never a permanent fake-looking jam. Collisions integrate with the Stage 05 vehicle damage hooks.

---

## 4. Civilians (zones, state machine, the report contract)

Civilian behavior is a small, explicit state machine — the missing-arm lesson applies (panic/report
are the commonly-skipped arms that make a city feel dead):

```
Wander ──sees threat/crime──► Alert ──┬──► Flee ──► (despawn at edge / Evacuate zone)
   ▲                                  ├──► Cower (in place, low cost)
   │ calm                             └──► REPORT ──emits witness event──► Heat (Stage 08)
   └──────────────────────────────────────────────────────────────────────┘
```
- **Pedestrian zones** (`/dataman`): sidewalks, crossings, gathering points, evacuation targets.
- **The integration contract (🔴 keystone):** a civilian that witnesses a crime emits a
  `STAT.Heat.Event.Witnessed`/`Reported` stimulus that the heat system (Stage 08) consumes. Acceptance
  literally requires "reporting can feed the police heat system" — so the report path is wired and
  tested now, with the receiver declared as `ISTATHeatReporter`.
- **Avoidance:** crowd avoidance/flow so peds don't clump or walk through each other; panic spreads
  believably (contagion radius) without a CPU spike.

---

## 5. Significance, LOD, Pooling, Cleanup (Function)

- **Significance manager:** full simulation only near the player/camera; distant traffic/peds drop to
  cheap LOD ticks or sleep — this is how the CPU budget is held when density is High.
- **Pooling:** fixed pools for vehicles + peds; spawn = acquire, despawn = release; no runtime
  allocation storms.
- **Cleanup on streaming unload (resilience):** when a Stage 06 cell unloads, its population returns to
  the pool — **no leaked actors** (the game analog of a leaked listener). Locked by a cleanup test.
- **Debug heatmaps:** population density, spawn sources, stuck hotspots, report events — visualized for
  tuning. Markers: `STAT.Population` trace; `stat` for spawn counts + per-category tick cost.

---

## 6. Data (Dataman)

Validated catalogs (Stage 02 framework; unique IDs, reference integrity, UI-safe, no secrets):
vehicle archetypes, pedestrian archetypes, lane/zone definitions, density-tier tables, reaction
parameters. Synthetic-realistic; provenance noted. Expandable later via the `/dataman` layered process
(repair→complete→enrich→stress→document), optionally Jules-delegated **after a written brief**.

---

## 7. Priority, Acceptance & Evidence

```
🔴 FIX NOW
  1. Witness→report→heat contract — without it the city looks alive but police never react (gate)
  2. Budgeted single-owner spawning with significance — hold AI/traffic CPU ≤4ms
  3. Pool return on streaming unload — no leaked actors
🟡 HIGH VALUE
  1. Stuck recovery (cars/peds) · 2. Pooling to kill spawn alloc churn · 3. Crowd avoidance/flow
🟢 POLISH
  1. Evaluate Mass vs pooled actors with a real capture; adopt only if it wins
```
**Highest-conviction change:** the **witness→report→heat contract**, because "systemic police that
remember behavior" is a core pillar and it is dead until a civilian's observation actually reaches the
heat system; confirm with `Population.WitnessEmitsReport` + a heat-rise integration check.

| Acceptance criterion (pack §07) | Status | Evidence / where |
|---|---|---|
| Traffic + civilians react believably within CPU budget | ◐ design-complete | §2–§5 — density captures on UE5 box |
| Reporting feeds police heat | ✅ wired + testable | §4 |
| Lanes/intersections/parking/spawn/despawn/stuck recovery | ✅ specified | §3 |
| Civilian zones/state/panic/witness/report/evac/avoidance | ✅ specified | §4 |
| Significance/LOD/pooling/budget/debug heatmaps | ✅ specified | §5 |

**Automation tests:** `Population.RespectsSpawnCap` · `Population.WitnessEmitsReport` ·
`Traffic.StuckRecovers` · `Population.PoolReturnsOnUnload` (no leak) · `Crowd.NoOverlapAvoidance`.
**Profiling:** Insights captures at Low/Med/High density tiers vs the CPU budget.

---

## 8. Next Dependency

**Stage 08 (Police Heat & Pursuit)** consumes: the **report/witness events** (its primary input),
traffic for roadblocks/chase obstacles, the district pursuit routes (Stage 06), and the vehicle/combat
systems. Primary lens `/function` (the heat→dispatch→search→cooldown loop) with `/ideas` (signature
pursuit moments) and `/anatomy` (wanted-meter/HUD legibility).
