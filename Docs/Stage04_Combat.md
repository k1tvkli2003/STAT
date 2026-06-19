# Stage 04 — Combat Vertical Slice

> **Status:** design-complete (behavioral architecture + contracts authored). Full `.h/.cpp`, GAS
> assets, AI graphs, the arena map, and Insights/automation runs happen on a UE5 workstation/CI — §9.
> **Skill lens:** `/function` **(invoked)** — wiring integrity, single source of truth, integration
> contracts, state completeness, game-thread discipline, resilience — with `/anatomy` (threat-HUD
> order) and `/style` (telegraph readability) supporting.
> **Governs:** `Prompts/01_MASTER_PROMPT.md`; gated by `Prompts/03_QUALITY_GATES.md`.
> **Consumes from Phase A:** input contexts + interaction scanner (Stage 03), message bus + world
> clock + seeded RNG + data-asset validation (Stage 02), `USTATWeaponDataAsset` + budgets (Stage 01).

This is greenfield, so FUNCTION MODE is applied **forward**: design the wiring, contracts, state arms,
and resilience so the named failure classes *cannot* occur, and specify the automation tests that lock
them. The Android/Compose reference layer is translated to UE5 (GAS, components, subsystems, async
tasks, GameplayCues, Insights) — no Kotlin/Compose/web code.

---

## 1. Function Audit (design out the failure classes)

```
╔══════════════════════════════════════════════════════════════╗
║  FUNCTION AUDIT — combat vertical slice                       ║
╠══════════════════════════════════════════════════════════════╣
║  Critical path: aim → GA_Fire → trace/projectile → hit →      ║
║    GE_Damage → armor exec → health clamp → GameplayCue +      ║
║    hit-react → threat event → EncounterDirector → resolve     ║
║  Encounter arms: Arming · Active · Lull(=Empty) · Won · Fail/Recover ║
║  Authority: health/armor/ammo = ONE ASC AttributeSet         ║
║  Hotspots: hitscan traces, AI perception, EQS, cue/VFX spawn  ║
╠══════════════════════════════════════════════════════════════╣
║  🔴 BROKEN-IF-NOT-DESIGNED                                    ║
║   • Kill never notifies director → encounter can't resolve    ║
║     (Integration Contract gap — looks done, isn't)            ║
║   • Player downed with no recover branch → soft-lock          ║
║     (State Completeness — missing arm)                        ║
║   • HUD reads health on Tick → drift + per-frame cost         ║
║     (Single Source of Truth + tick economy)                   ║
║  🟡 FRICTION / RESILIENCE                                     ║
║   • Reload/fire re-entrancy on rapid input (no guard)         ║
║   • EQS returns unreachable cover; enemy spawns in geometry   ║
║   • Save mid-encounter doesn't restore combat state           ║
║  🟢 OPPORTUNITY                                               ║
║   • Significance: disable tick/perception on off-screen        ║
║     combatants → reclaim AI budget                            ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 2. Single Source of Truth — attribute authority (GAS, justified)

The master prompt allows **GAS where justified**; combat health/armor/ammo + abilities is the canonical
justification (one replicated authority, prediction-ready for co-op at Stage 17). One `AttributeSet`,
no duplicates.

```cpp
// Interface design (C++). One authority for combat numbers.
UCLASS() class USTATCombatAttributeSet : public UAttributeSet {
  // ATTRIBUTE_ACCESSORS for each:
  FGameplayAttributeData Health, MaxHealth, Armor, MaxArmor, Ammo, MaxAmmo, ReserveAmmo;
  virtual void PostGameplayEffectExecute(const FGameplayEffectModCallbackData&) override; // clamp + downed check
};
```
- **HUD never polls.** The combat HUD (Stage 14) binds to `GetGameplayAttributeValueChangeDelegate(...)`
  and updates **on change only** — the UE analog of "don't recompose on an unstable param every frame."
  A health bar driven by a per-frame `Tick`/property-binding is a 🔴 (drift risk + wasted game-thread).
- **Ammo lives in the same authority**, decremented inside the fire ability's cost (GameplayEffect cost),
  so "fire" and "ammo HUD" can never disagree.

---

## 3. Wiring Integrity — every input resolves to a real effect

Each combat input from Stage 03's `IMC_OnFoot` maps to a real GameplayAbility or component call. No
bound-but-dead ability (the game analog of an empty `onClick {}`); no AnimNotify that nothing consumes.

| Input / trigger | Ability / effect | Real result (no dead wiring) |
|---|---|---|
| Fire (RT / LMB) | `GA_Fire` | cost: −Ammo; trace/projectile; `GE_Damage` to target ASC; noise event |
| Aim (LT / RMB) | `GA_Aim` | camera + spread state; threat-assist on |
| Reload | `GA_Reload` | montage; on `AnimNotify_AmmoLoaded` move Reserve→Ammo (**guarded**, §6) |
| Swap weapon | `GA_EquipWeapon` | sets active `USTATWeaponComponent`; rebinds ammo source |
| Take cover | `GA_EnterCover` | cover component binds to cover point; `STAT.Combat.Cover.*` tag |
| Takedown | `GA_Takedown` (context from scanner) | stealth (no noise) or alerted; victim removed; director notified |
| Melee | `GA_Melee` | short-range `GE_Damage`; stagger cue |

**AnimNotify contract:** `ApplyDamage`, `FootstepNoise`, `AmmoLoaded`, `TakedownCommit` each have a
real consumer; an orphan notify is a dead control. Locked by `Combat.NotifiesWired` test (§9).

---

## 4. Integration Contracts — cross-feature side effects MUST fire

The combat equivalent of "finish lesson → streak moves." Each row is invoked in code or it's a 🔴.

| Action | Must fire / write | Observed by | Recomputes |
|---|---|---|---|
| Enemy downed | `EncounterDirector.OnCombatantDowned` | Encounter director | encounter count, Lull/Won check |
| Enemy downed **if witnessed** | Heat event `STAT.Heat.Event.Witnessed` (Stage 08 hook) | Heat system | wanted level |
| Enemy downed in mission | Objective check (Stage 09 hook) | Mission system | objective progress |
| Shot fired | Noise stimulus to AI perception + telemetry combat event | AI hearing, telemetry | investigate/alert, metrics |
| Player damaged | `GE_Damage` → attribute change → threat event | HUD, camera, downed check | health bar, shake, downed |
| Player downed | Fail/Recover policy + checkpoint hook (Stage 15) + crew-revive (Stage 11) | Mission/checkpoint, crew | fail state OR revive |

Hooks to later stages are **declared interfaces** now (e.g. `ISTATHeatReporter`, `ISTATObjectiveSink`)
so Stage 04 fires them even though Stage 08/09 implement the receiver — no retrofitting, no silent gap.

---

## 5. State Completeness — the encounter has a Lull (Empty) arm

The "missing `Empty` branch = blank screen" lesson, applied to encounters. `USTATEncounterDirector`
runs an explicit machine; the commonly-missed arms are **Lull** and **Recover**.

```
Arming ──spawn/arm──► Active ──all visible down, reinforcements pending──► Lull
   ▲                    │  threat persists                                  │
   │                    ▼                                                    ▼
(designer trigger)   Failed ◄──player downed & no revive──┐         Active (reinforce) or Won
                        │                                  │
                     Recover ──checkpoint/respawn──► Active (retry) │ Won ──cleanup, release budget──►
```
- **Lull (= Empty):** all visible enemies down but encounter not Won (reinforcement timer, or player
  disengaged) — director keeps a defined behavior (spawn wave / de-escalate), never freezes.
- **Recover:** player downed → fail policy decides respawn-at-checkpoint or crew-revive; there is
  **always** a non-soft-lock exit (gate requirement: "failure states do not soft-lock the slice").
- The full encounter loop the arena must support: **enter → engage → recover → win/fail** (acceptance).

---

## 6. Resilience — hostile inputs & conditions (idempotent, guarded, seeded)

- **Re-entrancy guards:** `GA_Reload`/`GA_Fire` block on `STAT.State.Player.*` + ability tags so a
  rapid double-press can't double-load or fire mid-reload — the atomic-guard lesson (no check-then-act
  race), expressed as ability activation tags (`ActivationOwnedTags`/`BlockAbilitiesWithTag`).
- **Dry-fire branch:** firing at 0 ammo plays a click + suggests reload — a real branch, not a no-op.
- **Spawn validity:** combatants spawn only at validated nav-reachable points; an enemy that can't
  path falls back via EQS or is culled — never stuck in geometry (gate: no unfair/impossible spawns).
- **Save mid-encounter:** director serializes encounter state (combatants, wave, budget, player
  attributes) so a checkpoint restores the fight, not an empty room (Stage 15 contract).
- **Determinism:** damage variance / crit rolls draw from the seeded `"Combat"` RNG stream (Stage 02),
  so scored/replayed combat is reproducible — no `FMath::Rand()`.
- **No swallowed failures:** an ability that can't activate routes a reason to the debug/error channel;
  silently-failing input is the game analog of `catch {}`.

---

## 7. Game-Thread Discipline & Tick Economy (Amdahl on the game thread)

The game thread is STAT's serial bottleneck. Budget from Stage 01 §9: **AI ≤ 4 ms (min-spec)**.

| Hotspot | Discipline |
|---|---|
| Hitscan traces | Cap traces/frame; async trace where latency allows; single-line + small sweep, not many probes |
| AI perception | **Staggered** update (round-robin across frames), not every AI every tick |
| EQS (cover/flank/peek) | **Async, time-sliced**; cache results briefly; never block the frame |
| VFX/SFX | **GameplayCues** with **pooled** Niagara/MetaSounds; cosmetic-only, never gameplay-authoritative |
| Off-screen combatants | **Significance manager**: disable/long-period tick + perception when not relevant (🟢) |
| Animation | Montage/anim eval budgeted; URO (update-rate optimization) on distant enemies |

Markers: `STAT.Combat` trace group, `stat` counters for traces/frame and AI tick cost, debug view for
perception + EQS. "Never claim a speedup without a measurement plan" → each is verified in Insights (§9).

---

## 8. Encounter Director, Cover, AI & Arena (the assembled system)

- **`USTATEncounterDirector`** (per-encounter actor or world-subsystem-owned): owns the §5 state
  machine, a **spawn budget** (concurrent combatants ≤ tier cap), wave/reinforcement pacing,
  significance assignment, and fires the §4 resolution contracts. Single owner — no rival spawner.
- **AI:** `AIPerception` (sight/hearing/damage) → **combat StateTree** (Idle→Investigate→Engage→
  Suppressed→Flank→Flee/Down) with **EQS** for cover/flank/peek queries; readable **telegraphs**
  before committing attacks (the `/style` hook). Perception has memory (last-known position) for the
  Stage 08 search reuse.
- **Cover:** `USTATCoverComponent` + cover points (annotated volumes/EQS); high/low cover tags;
  peek/blindfire hooks.
- **Threat indicators:** damage-direction + incoming-fire events → HUD (anatomy orders them so the
  nearest lethal threat reads first; never a cluster).
- **Arena:** `L_CombatArena_Slice` with debug spawn pads, cover geometry, and designer-tunable
  `USTATWeaponDataAsset`/encounter data — supports the full enter→engage→recover→win/fail loop and
  stays **readable at low settings** (gate).

---

## 9. Priority, Acceptance & Evidence

```
🔴 FIX NOW (design in first)
  1. Kill→director resolution contract  — without it the encounter never ends (Integration Contract)
  2. Player-downed Recover arm          — without it a death soft-locks the slice (State Completeness)
  3. Event-driven HUD from the ASC       — polling drifts and burns game-thread (SSOT + tick economy)
🟡 HIGH VALUE
  1. Reload/fire re-entrancy guards · 2. EQS reachable-cover + valid spawn · 3. Save-mid-encounter restore
🟢 POLISH
  1. Significance tick-disable off-screen · 2. Telegraph/threat-read tuning
```
**Highest-conviction change:** the **kill→`EncounterDirector` resolution contract**, because without it
the encounter *looks* complete (enemies die) yet never reaches Won — every downstream system (mission
completion, heat cooldown, rewards) stalls on it; confirm with the `Encounter.ResolvesOnLastDown` test.

| Acceptance criterion (pack §04) | Status | Evidence / where |
|---|---|---|
| Arena supports enter→engage→recover→win/fail | ◐ design-complete | §5, §8 — run on UE5 box |
| Combat readable at low settings | ◐ design-complete | §7 (cues cosmetic, telegraphs) + `/style` Stage 13 |
| Weapon/damage/health/armor/ammo/recoil/spread/reload | ✅ specified | §2, §3 |
| Cover, takedowns, threat indicators, telegraphs | ✅ specified | §3, §8 |
| AI perception + StateTree/BT + EQS + director | ✅ specified | §8 |
| Profiling for traces/projectiles/anim/VFX/AI tick | ✅ specified | §7 (Insights markers + budgets) |

**Automation tests (`STATTests`):** `Combat.NotifiesWired` · `Encounter.ResolvesOnLastDown` ·
`Encounter.LullSpawnsOrDeEscalates` · `Combat.PlayerDownedRecovers` (no soft-lock) ·
`Combat.ReloadReentrancyGuarded` · `Combat.DamageDeterministicSeeded` · `AI.SpawnsNavReachable` ·
`Combat.SaveRestoresEncounter`. **Profiling:** Insights capture of a full arena encounter vs the
AI/frame budgets (Stage 01 §9).

---

## 10. Next Dependency

**Stage 05 (Vehicle Vertical Slice)** consumes: the GAS attribute/ability pattern (vehicle damage as
attributes), the encounter/threat hooks (drive-by + pursuit handoff to Stage 08), input contexts
(OnFoot↔Vehicle from Stage 03), and the seeded RNG + save contracts. Primary lens `/function`;
supporting `/style` (drive feel feedback).
```
─────────────────────────────────────────────────────────
Stage 04 behavior locked. The kill→director resolution contract is the keystone;
missions, heat cooldown, and rewards all wait on the encounter actually reaching "Won".
─────────────────────────────────────────────────────────
```
