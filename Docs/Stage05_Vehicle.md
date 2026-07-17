# Stage 05 — Vehicle Vertical Slice

> **Status:** design-complete (behavioral architecture + contracts authored). Full `.h/.cpp`, Chaos
> vehicle assets, the hero vehicle tune, and physics/Insights/automation runs happen on a UE5
> workstation/CI — §9.
> **Skill lens:** `/function` **(applied)** — single source of truth, wiring, integration contracts,
> state completeness, physics/thread discipline across frame rates, resilience — with `/style` (drive
> feel) supporting (Stage 13).
> **Governs:** `Prompts/01_MASTER_PROMPT.md`; gated by `Prompts/03_QUALITY_GATES.md`.
> **Consumes:** input contexts + camera handoff (Stage 03), GAS attribute pattern + encounter/heat
> hooks (Stage 04), message bus + world clock + seeded RNG + save/validation (Stage 02), budgets (01).

---

## 1. Function Audit (design out the failure classes)

```
╔══════════════════════════════════════════════════════════════╗
║  FUNCTION AUDIT — vehicle vertical slice                      ║
╠══════════════════════════════════════════════════════════════╣
║  Critical path: enter → context+camera handoff → Chaos        ║
║   movement (throttle/steer/brake/handbrake) → surface/damage  ║
║   response → HUD(event) → exit / repair / respawn             ║
║  Vehicle states: Empty · Occupied · Damaged · Disabled ·       ║
║   Repairing · Respawning   (all arms defined — no soft-lock)  ║
║  Authority: one vehicle-state owner; HUD/seat event-driven    ║
║  Hotspots: Chaos physics sub-stepping, camera collision probe ║
╠══════════════════════════════════════════════════════════════╣
║  🔴 design out                                                ║
║   • Handling that changes with frame rate (no sub-stepping)   ║
║     → "heavy but controllable" only at 60fps (gate fail)      ║
║   • Disabled vehicle with no respawn/exit → soft-lock          ║
║   • Vehicle state lost on save/world-transition (gate fail)   ║
║  🟡 friction / resilience                                     ║
║   • Enter/exit at speed, flipped/stuck recovery, double-tap   ║
║   • Camera clips geometry in tight alleys                     ║
║  🟢 opportunity                                               ║
║   • Significance: cheap physics/tick for distant/parked cars  ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 2. Single Source of Truth — vehicle state authority

One owner per fact, mirroring Stage 04's attribute discipline.

```cpp
// Interface design (C++). Chaos vehicle pawn + a state component that is the authority.
UCLASS() class ASTATVehiclePawn : public AWheeledVehiclePawn { /* Chaos movement comp */ };

UCLASS() class USTATVehicleStateComponent : public UActorComponent {
  // authoritative, replication-ready (co-op Stage 17), saved (Stage 15):
  float BodyHealth, EngineHealth; uint8 TireFlags; ESTATVehicleState State; FName OccupantDriverId;
  // change delegates drive the HUD — NO per-frame polling of speed/health:
  FOnVehicleStateChanged OnChanged;
};
```
- Handling/tuning is **data-driven** from `USTATVehicleDataAsset` (Stage 01) — designers tune mass,
  engine, suspension, grip, steering curves without code.
- Speed/RPM/damage HUD binds to `OnChanged` / movement-comp accessors via events; a vehicle HUD that
  reads physics every `Tick` is the recomposition-storm analog (drift + cost) — a 🔴.

---

## 3. Wiring Integrity — every control resolves to real movement

| Input (`IMC_Vehicle`) | Real result |
|---|---|
| Throttle / Brake | Chaos movement comp set throttle/brake (analog) |
| Steer | analog steer with speed-sensitive curve + assist (data-driven) |
| Handbrake | rear-grip cut for handbrake turns |
| Reverse | reverse gear engage at ~0 speed (not throttle-flip mid-roll) |
| Horn / Siren | audio hook (cosmetic) + AI/traffic awareness stimulus (Stage 07/08) |
| Exit | `GA_ExitVehicle`: consume input (no bleed), context+camera handoff to OnFoot (Stage 03) |
| Repair (at safehouse) | repair service → reset health attrs (Stage 11 garage) |

No dead control: an input bound but not driving the movement component is the game analog of an empty
`onClick {}`. Locked by `Vehicle.InputsDriveMovement` test.

---

## 4. Integration Contracts — cross-feature side effects MUST fire

| Action | Must fire / write | Observed by | Recomputes |
|---|---|---|---|
| Enter vehicle | context swap + camera handoff + occupant register | Input routing (03), camera, seat rules | control scheme, seat HUD |
| Exit vehicle | symmetric handoff; vehicle parked/idle | Input routing, vehicle state | on-foot control |
| Vehicle damaged | health attrs ↓ + surface/impact cue + disable check | HUD, VFX, state machine | damage HUD, Disabled arm |
| Vehicle used in crime | `STAT.Heat.Event.*` + vehicle-recognition tag (Stage 08 hook) | Heat system | wanted level, recognition |
| Vehicle state change | save-dirty mark | Save (Stage 15) | persistence on checkpoint |
| Hero vehicle stored | garage spawn record (Stage 11 hook) | Garage/safehouse | spawn availability |

Receivers in later stages are declared interfaces now (`ISTATHeatReporter`, garage spawn hook) so the
contract fires without retrofitting.

---

## 5. State Completeness — every arm, no soft-lock

```
Empty ──enter──► Occupied ──damage accumulates──► Damaged ──critical──► Disabled
   ▲                │                                                      │
   │ park/exit      │ exit                                    respawn/garage│ (always an exit)
   └────────────────┘                                                      ▼
                         Repairing ◄──service──┐               Respawning ──► Empty (at safe point)
```
- **Disabled (= the Empty/dead arm):** a wrecked/immobilized vehicle is never a dead end — player can
  exit on foot, call a garage respawn, or repair at the safehouse. Gate: "failure states do not
  soft-lock the slice."
- **Respawning:** validated safe placement (nav-reachable, not in geometry/traffic) — never spawns the
  car inside a wall (mirrors Stage 04 spawn validity).

---

## 6. Physics & Thread Discipline — stable at multiple frame rates (the hard one)

"Driving feels heavy but controllable" must hold at **30 and 60 fps** (and uncapped). Frame-rate-coupled
handling is the headline 🔴.

- **Async physics + sub-stepping:** Chaos vehicle on the physics thread with fixed sub-steps so tire
  forces integrate consistently regardless of render frame rate — handling is the *same* at 30/60/120.
- **Input latency budget:** steering/throttle sampled and applied each physics sub-step; measure
  input→response latency in Insights (no multi-frame buffering on control inputs).
- **Camera collision:** spring-arm probe with smooth pull-in for tight alleys (the 🟡 clip); reuses the
  Stage 03 collision-camera pattern, tuned for speed and look-ahead.
- **Significance:** parked/distant vehicles drop to cheap or sleeping physics + low tick (🟢) to protect
  the frame budget when traffic arrives (Stage 07).
- Markers: `STAT.Vehicle` trace group; `stat` for physics sub-step cost + input latency; debug view for
  suspension/contact. Verified in Insights vs Stage 01 budgets — no guessed numbers.

---

## 7. Hero Vehicle, Surface Response, Garage & Persistence

- **One hero vehicle** fully tuned (`USTATVehicleDataAsset`) for **KB/M and gamepad**, validated at
  multiple frame rates — the slice's signature drive.
- **Surface response:** per-surface grip/handling/audio (asphalt/wet/dirt) via physical materials;
  feeds the Stage 13 wetness/weather hook.
- **Collision/traffic:** impact damage + transfer; hooks for traffic-vehicle collision (Stage 07).
- **Garage spawn + persistence:** hero vehicle stored/retrieved at the safehouse (Stage 11 hook);
  vehicle state (health, tire flags, position, fuel if modeled) **survives save/load and world
  transitions** (acceptance) via the Stage 15 save contract + stable IDs.

---

## 8. (reserved — style hooks)

Drive-feel *feedback* (camera shake, speed FOV, damage deformation, dust/skid VFX, engine mix) is
specified by `/style` at Stage 13; Stage 05 exposes the hooks/events only, so feel can be tuned without
touching vehicle behavior.

---

## 9. Priority, Acceptance & Evidence

```
🔴 FIX NOW
  1. Async physics + sub-stepping  — frame-rate-independent handling (else "controllable" fails at 30fps)
  2. Disabled-arm exit/respawn      — no soft-lock on a wrecked car
  3. Vehicle state in save/transition — survives save/load + world change (gate)
🟡 HIGH VALUE
  1. Enter/exit-at-speed + flip/stuck recovery · 2. Camera collision in tight space · 3. Event-driven HUD
🟢 POLISH
  1. Significance physics/tick for parked/distant cars
```
**Highest-conviction change:** **async physics with sub-stepping**, because the entire "heavy but
controllable" pillar is meaningless if handling changes with frame rate — it must be identical at 30 and
60 fps; confirm with an input-latency + handling-consistency capture across capped frame rates.

| Acceptance criterion (pack §05) | Status | Evidence / where |
|---|---|---|
| Driving feels heavy but controllable | ◐ design-complete | §2, §6, §7 — tune + capture on UE5 box |
| Vehicle state survives save/load + world transitions | ◐ design-complete | §4, §7 + `Vehicle.SaveRestores` |
| Pawn/component architecture, data-driven tuning | ✅ specified | §2 |
| Enter/exit, seats, camera, assists, handbrake, reverse, horn/siren, damage | ✅ specified | §3 |
| Surface response, collision, repair, respawn, garage spawn | ✅ specified | §5, §7 |
| Profiling: physics stability, input latency, camera, save restore | ✅ specified | §6, §9 |

**Automation tests (`STATTests`):** `Vehicle.InputsDriveMovement` · `Vehicle.HandlingFrameRateStable`
(30 vs 60) · `Vehicle.DisabledArmNoSoftLock` · `Vehicle.SaveRestores` · `Vehicle.RespawnNavValid` ·
`Vehicle.EnterExitContextNoBleed`. **Profiling:** Insights physics sub-step cost + input latency vs
budgets.

---

## 10. Next Dependency

**Stage 06 (Dense District & Streaming)** consumes: the hero vehicle (drive routes, chase paths),
camera collision behavior, and significance hooks (parked cars become district set-dressing). Primary
lens `/function` (streaming/hitch discipline) with `/anatomy` (district legibility/landmarks).
