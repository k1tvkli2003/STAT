# Stage 02 — Core Lifecycle & Data Architecture

> **Status:** design-complete (architecture + interface contracts authored). Full `.h/.cpp`
> implementation, compile, and automation-run happen on a UE5 workstation/CI — see §10.
> **Skill lens:** `/function` (lifecycle wiring, single source of truth, state completeness,
> resilience) · `/dataman` (data-asset validation framework, stable IDs, schema versions) ·
> `/anatomy` (front-end→world flow as information architecture — no dead ends, no leaks).
> **Governs:** `Prompts/01_MASTER_PROMPT.md`; gated by `Prompts/03_QUALITY_GATES.md`.
> **Consumes from Stage 01:** module map, Primary Asset Types + bases, Gameplay Tag root, budget markers.

---

## 1. Stage Audit

Per `02_BUILD_SEQUENCE.md §02`, deliver an **empty but robust front-end-to-world loop** plus the data
spine: subsystem ownership map; menu→world→menu flow; typed event contracts; `UPrimaryDataAsset` bases
with a validation framework, stable IDs, schema versions, debug commands; deterministic RNG; and an
automation harness. Acceptance: (a) menu↔world with **no leaked state**, (b) invalid data assets fail
validation with **actionable messages**.

**`/function` trace framing.** The two failure classes to design out now:
1. **State leak across the lifecycle** — a subsystem, timer, delegate binding, streamed level, or
   loaded asset that survives a return-to-menu and corrupts the next session. (Single Source of Truth
   + lifecycle discipline.)
2. **Silent data drift** — a data asset with a duplicate/missing ID, dangling reference, or stale
   schema version that loads anyway and fails later. (Validation is the function, not polish.)

**`/anatomy` flow framing.** Treat game flow like navigation IA: every state has a defined entry,
exit, and back path; no state is a dead end; "continue" and "new game" are reachable and unambiguous;
pause is a modal overlay (dismissed, not "backed out of"), matching Jakob's-Law conventions.

---

## 2. Subsystem Ownership Map (Single Source of Truth)

`/function`'s first law here: **one owner per fact.** Each subsystem below is the sole authority for
its domain; nothing else stores a second copy. Lifetime is chosen as the *narrowest* that fits.

| Subsystem | Base class | Lifetime | Authoritative for | Threading |
|---|---|---|---|---|
| `USTATGameFlowSubsystem` | `UGameInstanceSubsystem` | App | Current game-flow state + transitions | Game thread |
| `USTATSaveSubsystem` | `UGameInstanceSubsystem` | App | Save slots, schema version, migration (impl Stage 15) | Async IO |
| `USTATDataRegistrySubsystem` | `UGameInstanceSubsystem` | App | Loaded Primary Data Assets, validation results | Game thread; async load |
| `USTATRNGSubsystem` | `UGameInstanceSubsystem` | App | Named deterministic RNG streams + seeds | Game thread |
| `USTATMessageSubsystem` | `UGameInstanceSubsystem` | App | Typed event/message bus routing | Game thread |
| `USTATWorldClockSubsystem` | `UWorldSubsystem` | World | Sim clock, tick phases, pause-aware time | Game thread |
| `USTATWorldStateSubsystem` | `UWorldSubsystem` | World | District/mission/heat runtime roots (filled later stages) | Game thread |
| `USTATInputRoutingSubsystem` | `ULocalPlayerSubsystem` | LocalPlayer | Active input context + mapping (impl Stage 03) | Game thread |
| `USTATPlayerProfileSubsystem` | `ULocalPlayerSubsystem` | LocalPlayer | Profile, settings, accessibility prefs | Game thread |

**Ownership rule of thumb:** App-wide truth → GameInstance subsystem; per-world truth dies with the
world → World subsystem; per-player truth → LocalPlayer subsystem. A return-to-menu **tears down all
World subsystems**; GameInstance subsystems persist but are **reset to a clean state** via an explicit
`ResetForNewSession()` contract (§9) — never relying on default-constructed leftovers.

---

## 3. Game Flow States & Transitions

`/anatomy`: a small, explicit state machine owned by `USTATGameFlowSubsystem`. No global polling —
transitions are events; every arrow below is a real, wired transition with entry/exit work.

```
        ┌─────────┐  start   ┌──────────────┐  New Game / Continue
        │  Boot   ├─────────►│   FrontEnd   ├───────────────┐
        └─────────┘          │ (main menu)  │◄──────────┐    │
                             └──────┬───────┘  Quit to  │    ▼
                                    │          Menu     │ ┌──────────┐
                       Settings/Profile (modal)         │ │ Loading  │ (async: stream world,
                                    │                    │ └────┬─────┘  restore save or new)
                                    ▼                    │      │ ready
                             (overlay, dismiss)          │      ▼
                                                         │ ┌──────────┐  Pause (modal overlay)
                                                         └─┤  InWorld ├───────────────┐
                                                           └────┬─────┘◄── resume ────┘
                                                       Quit App │
                                                                ▼
                                                            ┌───────┐
                                                            │ Exit  │
                                                            └───────┘
```

**State contract (`/function` state-completeness — model every arm, not the happy path):**

```cpp
// Interface design (C++), implemented on the UE5 box.
UENUM(BlueprintType)
enum class ESTATGameFlowState : uint8 { Boot, FrontEnd, Loading, InWorld, Paused, Exiting };

// Transition request is validated; illegal transitions are rejected with a logged reason,
// never silently ignored (a swallowed transition is a /function 🔴).
struct FSTATFlowTransition { ESTATGameFlowState From; ESTATGameFlowState To; FName Reason; };

// One-shot effects (load/unload/notify) fire via the message bus (§4), NOT stored as State booleans
// (a "navigate" bool re-fires on the next tick / re-entry — the MVI Effect lesson from /function).
```

- **Loading** is always async: stream the startup/world, then restore save (Stage 15) or seed a new
  game, then transition to **InWorld** only when `IsReady` contracts from each subsystem resolve.
- **Paused** is a modal overlay over InWorld (time dilation 0 via the world clock), dismissed on resume.
- **Quit-to-Menu** runs the teardown contract (§9) before re-entering FrontEnd.

---

## 4. Typed Event / Message Contracts

`/function` Wiring Integrity: cross-system communication goes through **one typed bus**
(`USTATMessageSubsystem`, wrapping UE Gameplay Message Router) so features stay decoupled and every
listener is traceable. No feature reaches into another feature's internals.

```cpp
// Interface design (C++). Messages are typed structs tagged with a Gameplay Tag channel.
USTRUCT(BlueprintType) struct FSTATFlowStateChanged { ESTATGameFlowState NewState; ESTATGameFlowState Old; };
USTRUCT(BlueprintType) struct FSTATSessionReset    { FName Reason; };
USTRUCT(BlueprintType) struct FSTATWorldReady       { FPrimaryAssetId District; };

// Channels (Gameplay Tags from Stage 01 §7), e.g.:
//   STAT.Msg.Flow.StateChanged   STAT.Msg.Session.Reset   STAT.Msg.World.Ready
// Publish:  Message.Broadcast(Tag, Payload);
// Listen:   Message.Listen<FSTATWorldReady>(Tag, this, &ThisClass::OnWorldReady);  // auto-unbound on destroy
```

**Contract rule:** a listener registration returns a handle that is released in the owner's
`Deinitialize`/`EndPlay`. An un-released listener that survives teardown is a leak (the acceptance
criterion failure) — caught by the lifecycle leak test in §8.

---

## 5. Clocks & Deterministic RNG

Determinism is a cross-cutting track (Plan §6) and a gate. Two pieces land here:

- **`USTATWorldClockSubsystem`** — the single sim clock. Systems read time/delta from it (pause-aware,
  dilation-aware), never from raw `UWorld::GetTimeSeconds` in scored/saved paths. Fixed-step
  accumulator option for systems that must be frame-rate independent.
- **`USTATRNGSubsystem`** — **named seeded streams** so any scored/saved/replayed/VEX system is
  reproducible. No `FMath::Rand()` in those paths.

```cpp
// Interface design (C++).
FRandomStream& USTATRNGSubsystem::GetStream(FName StreamName);     // created on first use from session seed
void           USTATRNGSubsystem::SeedSession(int64 SessionSeed);  // persisted to save; restores identical sequences
// Streams: "Traffic", "CrowdBarks", "LootJitter", "Vex.CaseShuffle", ... each independent & reproducible.
```

`/function` determinism test (§8): seed → record N draws across streams → reseed → assert identical.

---

## 6. Primary Data Asset Architecture & Validation (`/dataman`)

`/dataman` flow: **audit → brief → expand-in-layers → validate.** Stage 02 builds the *framework*
(bases + validators + registry); later stages populate the catalogs.

### 6.1 Base hierarchy
```cpp
// Interface design (C++). All content data assets derive from one validated base.
UCLASS(Abstract)
class USTATPrimaryDataAsset : public UPrimaryDataAsset {
  GENERATED_BODY()
public:
  UPROPERTY(EditDefaultsOnly, AssetRegistrySearchable) FName StableId;   // unique, never reused
  UPROPERTY(EditDefaultsOnly) int32 SchemaVersion = 1;                   // bump on shape change
#if WITH_EDITOR
  virtual EDataValidationResult IsDataValid(FDataValidationContext& Ctx) const override; // §6.2
#endif
  virtual FPrimaryAssetId GetPrimaryAssetId() const override;            // {Type, StableId}
};
```
Concrete bases from Stage 01 §6 (`USTATMissionDataAsset`, `…Vehicle/Weapon/Crew/District/VexCase/UITheme`)
extend this and add their own `IsDataValid` rules.

### 6.2 Validation framework (the acceptance criterion)
Editor-time (`IsDataValid`) **and** a commandlet/automation pass for CI. Each check yields an
**actionable** message (`/function`: errors name the field and the fix, never a generic failure):

| Check | Failure message shape |
|---|---|
| `StableId` non-empty & unique across type | `"DA_X: StableId 'foo' duplicates DA_Y — IDs must be unique."` |
| `SchemaVersion` ≥ 1 and ≤ current | `"DA_X: SchemaVersion 3 > current 2 — add a migration or downgrade."` |
| All soft refs resolve | `"DA_X.Reward: asset reference is null/unresolved."` |
| Tag fields are declared tags | `"DA_X.Phase: 'STAT.Mission.Phase.Bogus' is not a registered tag."` |
| Numeric ranges in-band | `"DA_X.Recoil: 12.0 outside [0,5]."` |
| Localization keys present for FText | `"DA_X.Title: missing localization key."` |

### 6.3 Registry
`USTATDataRegistrySubsystem` async-loads by Primary Asset Type, caches handles, exposes
`GetValidated(Type)` and surfaces validation results to debug tools (§7). Invalid assets are **excluded
from gameplay loads** and reported, never silently used.

---

## 7. Debug Commands

Registered through `STATCore`'s debug-command registry; compiled out of Shipping.

```
STAT.Flow.Goto <State>            // force a flow transition (dev) and assert it's legal
STAT.Session.Reset               // run teardown contract; verify clean state
STAT.Data.Validate [Type]        // run validators now; print actionable failures
STAT.Data.List <Type>            // list loaded assets + StableId + SchemaVersion + valid?
STAT.RNG.Dump <Stream> <N>       // print next N draws (determinism debugging)
STAT.Clock.Dilation <x>          // inspect/scale sim time
```

---

## 8. Automation Test Harness

`STATTests` specs (UE Automation). These are the Stage 02 gate evidence to run on the toolchain:

| Test | Asserts | Lens |
|---|---|---|
| `Lifecycle.MenuToWorldToMenu` | Boot→FrontEnd→Loading→InWorld→(Quit)→FrontEnd completes; flow state correct at each step | `/anatomy` |
| `Lifecycle.NoStateLeak` | After a full round-trip: 0 lingering World subsystems, 0 live timers, 0 dangling message listeners, GameInstance subsystems report `IsClean()` | `/function` |
| `Data.InvalidFailsLoudly` | Each malformed fixture (dup ID, bad tag, null ref, future schema) fails `IsDataValid` with the expected actionable message | `/dataman` |
| `Data.ValidLoads` | Well-formed fixtures load and register; `GetValidated` returns them | `/dataman` |
| `RNG.Deterministic` | Same session seed ⇒ identical multi-stream draw sequences; different seed ⇒ different | `/function` |
| `Clock.PauseFreezesSim` | In Paused, sim delta = 0; resume restores | `/function` |

Fixtures live in `Content/STAT/_Dev/TestData/` (and `STATTests` C++ fixtures), excluded from Shipping.

---

## 9. State-Leak Prevention (the hard acceptance criterion)

`/function` resilience design — an explicit, tested teardown contract instead of hoping GC handles it:

- **`ISTATSessionScoped`** interface: subsystems/objects holding session state implement
  `ResetForNewSession()`; `USTATGameFlowSubsystem` calls all of them on Quit-to-Menu and broadcasts
  `FSTATSessionReset`.
- **Deterministic teardown order:** World subsystems uninitialize → streamed levels unloaded → async
  loads cancelled → message listeners released → GameInstance subsystems `ResetForNewSession()`.
- **Leak assertions** (debug + `Lifecycle.NoStateLeak`): no orphaned timers, delegates, async handles,
  or streamed levels survive the transition. `IsClean()` on each GameInstance subsystem must be true.

This is exactly the "menu↔world without leaking state" criterion, made testable rather than assumed.

---

## 10. Acceptance Status & Evidence Checklist

| Acceptance criterion (pack §02) | Status | Evidence / where |
|---|---|---|
| Menu→world→menu with no leaked state | ◐ design-complete | §2, §3, §9 + `Lifecycle.NoStateLeak` — **run on UE5 box for log** |
| Invalid data assets fail validation with actionable messages | ◐ design-complete | §6.2 + `Data.InvalidFailsLoudly` — **run on UE5 box for log** |
| Subsystem ownership map | ✅ | §2 |
| Front-end→world flow (new game/continue/pause/quit) | ✅ | §3 |
| Typed event/message contracts | ✅ | §4 |
| Primary Data Asset bases + validation + stable IDs + schema versions + debug cmds | ✅ | §6, §7 |
| Automation tests (lifecycle, invalid data, RNG, validation) | ✅ specified | §8 (implement + run to capture) |

**To close Stage 02 on a UE5 workstation:** implement the §2/§3/§6 classes as full `.h/.cpp`, author
the malformed + valid test fixtures, run the §8 specs headless, and paste results into
`Plan/02_PROGRESS_LOG.md`. Then Stage 02 flips ✅.

---

## 11. Next Dependency

**Stage 03 (Input, Camera, Character, Interaction)** consumes: `USTATInputRoutingSubsystem` +
`USTATPlayerProfileSubsystem` (settings/accessibility prefs), the message bus (input/interaction
events), the world clock (camera/interaction timing), and the flow states (input contexts switch with
FrontEnd/InWorld/Paused). Primary lens `/anatomy`; supporting `/function`, `/style`.
