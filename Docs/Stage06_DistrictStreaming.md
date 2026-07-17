# Stage 06 — Dense District & Streaming

> **Status:** design-complete (architecture + budgets authored). Building the actual World Partition
> map, HLOD, nav, lighting, and capturing streaming/Insights evidence happen on a UE5 workstation/CI — §8.
> **Skill lens:** `/function` **(applied)** — streaming hitch = jank (game-thread discipline,
> perception thresholds, memory budget, resilience at streaming boundaries) — with `/anatomy`
> (district legibility, landmarks, route IA) supporting.
> **Governs:** `Prompts/01_MASTER_PROMPT.md`; gated by `Prompts/03_QUALITY_GATES.md`.
> **Consumes:** hero vehicle + significance (Stage 05), encounter/arena needs (Stage 04), budgets (01).

---

## 1. Function + Anatomy Audit

```
╔══════════════════════════════════════════════════════════════╗
║  AUDIT — one dense district, streamed                         ║
╠══════════════════════════════════════════════════════════════╣
║  Critical path: move (foot/vehicle) → streaming source loads  ║
║   WP cells ahead → HLOD bridges distance → nav/occlusion →    ║
║   stable frame; cross cell boundary at speed without hitch    ║
║  Budgets (Stage 01): streaming hitch ≤1 frame over; mem ≤8GB; ║
║   load ≤20s cold; frame 33ms min-spec                         ║
╠══════════════════════════════════════════════════════════════╣
║  🔴 design out                                                ║
║   • Vehicle outruns streaming → hitch/empty world at speed     ║
║     (perception threshold: a >16ms stall is a felt bug)        ║
║   • HLOD gaps → visible pop (legibility + polish fail)         ║
║   • Save/load mid-district doesn't restore streamed state      ║
║  🟡 friction                                                  ║
║   • Nav/occlusion not built for alleys/interiors → AI/draw cost║
║   • Landmarks weak → players can't navigate the chase routes   ║
║  🟢 opportunity                                               ║
║   • Lighting scenarios (day/night) via Data Layers for mood    ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 2. District Layout (Anatomy — legibility & route IA)

`/anatomy` Information Scent + wayfinding applied to a 3D space: the district must be **navigable from
landmarks**, with legible, intentional routes — not a uniform grid.

- **One production-quality district** (`L_District_Slice`, World Partition): roads, alleys, interiors
  or clear entry points, **safehouse** (the "home" anchor — always re-findable), a **combat space**
  (Stage 04 arena integrated, not bolted on), **pursuit routes** (Stage 08), and **landmarks**
  (silhouette beacons for orientation).
- **Route legibility:** distinct, readable paths for **foot / vehicle / chase / mission** — each with
  traversal metrics (§4) so designers tune pacing. Landmarks anchor the chase routes so a fleeing
  player can navigate at speed (information scent at 80 km/h).
- **Density is intentional** (Cognitive Load): dense where it matters (set-piece blocks), breathing
  room elsewhere — measured against a content-density budget (§7), not maxed everywhere.

---

## 3. Streaming Architecture (Function — no felt hitch)

- **World Partition + Data Layers:** cells stream by distance; Data Layers gate gameplay/lighting
  variants (day/night, mission-state set-dressing) without duplicate maps.
- **Streaming sources on player AND vehicle**, with **velocity-based prefetch**: at vehicle speed the
  source look-ahead extends along the velocity vector so the world is resident *before* arrival — the
  headline 🔴 (outrunning streaming) is designed out, not hoped away.
- **HLOD strategy:** hierarchical proxies bridge unloaded distance so there's no pop; tuned per cell
  tier. The "no visible pop" bar is a legibility + polish requirement.
- **Navigation:** runtime/baked nav for foot + AI across alleys/interiors; nav invalidation on
  streamed geometry handled so AI never paths into not-yet-loaded space.
- **Occlusion:** precomputed/again-runtime occlusion for dense blocks + interiors to hold the draw
  budget.

---

## 4. Traversal Metrics

Authored metrics (data, debug-visualized) so pacing is measured, not guessed:

| Route | Metric captured |
|---|---|
| Foot | time safehouse→combat space, alley shortcuts, vertical traversal points |
| Vehicle | lap/route times, turn radii, jump/shortcut viability at hero-vehicle tune |
| Chase | escape-route lengths, choke points, line-of-sight breaks (feeds Stage 08 search) |
| Mission | objective-to-objective distances, approach variety (feeds Stage 09) |

---

## 5. Resilience & Thread Discipline

- **Boundary crossings at speed:** prefetch + budgeted cell load keep the frame within budget; a stall
  >1 frame over budget is a 🔴 (Insights stall capture).
- **Save/load mid-district:** streamed-world restoration brings back the correct cells/Data Layers and
  the player/vehicle position without a hitch or a fall-through-world (Stage 15 contract).
- **Async loading:** cell load/HLOD transitions off the game thread; no synchronous load on the
  critical path. Memory held to budget by unloading behind the streaming source.
- Markers: `STAT.Streaming` trace; `stat streaming`, `stat memory`; debug view for active cells +
  sources + nav + HLOD tier.

---

## 6. Lighting Scenarios

Day/night (and weather-ready, Stage 13) lighting via **Data Layers / lighting scenarios** so mood
changes without map duplication, within the GPU/VRAM budget. Low settings must preserve essential
readability (gate) — lighting never hides gameplay-critical information.

---

## 7. Content-Density Budget (gate before scale)

A written budget — actors/cell, draw calls/view, nav complexity, memory/cell, HLOD coverage — that the
district must meet **before any city-scale expansion** (the project's hard rule: prove the slice
first). "Content density feels intentional" is checked against this budget, not vibes.

---

## 8. Priority, Acceptance & Evidence

```
🔴 FIX NOW
  1. Velocity-based streaming prefetch — no hitch/empty-world when the vehicle outruns default streaming
  2. HLOD coverage — no visible pop across the district
  3. Streamed-world save/restore — correct cells + position on load
🟡 HIGH VALUE
  1. Nav/occlusion for alleys + interiors · 2. Landmark beacons for chase wayfinding
🟢 POLISH
  1. Day/night lighting scenarios via Data Layers
```
**Highest-conviction change:** **velocity-based streaming prefetch**, because the whole "dense reactive
city + heavy driving" promise collapses the first time a player at speed hits an unloaded cell — a felt
hitch (>16ms) reads as broken; confirm with a drive-the-perimeter Insights capture showing zero
budget-breaking stalls.

| Acceptance criterion (pack §06) | Status | Evidence / where |
|---|---|---|
| District streams without major hitches on target HW | ◐ design-complete | §3, §5 — Insights capture on UE5 box |
| Content density feels intentional before scale | ✅ budgeted | §7 |
| WP/Data Layers/HLOD/nav/occlusion/sources/lighting | ✅ specified | §3, §6 |
| Traversal metrics (foot/vehicle/chase/mission) | ✅ specified | §4 |
| Automated streaming walk/drive tests + Insights | ✅ specified | §8 below |

**Automation tests:** `Streaming.DriveLoopNoHitch` (automated drive of the perimeter, assert no
budget-breaking stall) · `Streaming.WalkLoopNoHitch` · `Streaming.SaveRestoresCells` ·
`Nav.ReachableAllRoutes` · `Streaming.MemoryWithinBudget`. **Profiling:** Insights captures for the
foot, vehicle, chase, and mission journeys.

---

## 9. Next Dependency

**Stage 07 (Traffic, Crowds, Civilians)** consumes: the district nav + lanes, significance/streaming
budget headroom, and landmark/route layout (traffic flows along the legible routes). Primary lens
`/function` (population within CPU budget) with `/dataman` (spawn tables/zones).
