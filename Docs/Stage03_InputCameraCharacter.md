# Stage 03 — Input, Camera, Character & Interaction

> **Status:** design-complete (structure + interaction contracts authored). Full `.h/.cpp`, Enhanced
> Input assets, camera tuning, and automation runs happen on a UE5 workstation/CI — see §9.
> **Skill lens:** `/anatomy` **(invoked)** — interaction-prompt prioritization, input-context
> structure, controller-first focus, reachability — with `/function` (wiring, no input bleed, no
> flicker) and `/style` (prompt/glyph feedback) supporting.
> **Governs:** `Prompts/01_MASTER_PROMPT.md`; gated by `Prompts/03_QUALITY_GATES.md`.
> **Consumes from Stage 02:** `USTATInputRoutingSubsystem`, `USTATPlayerProfileSubsystem`, message
> bus, world clock, flow states.

---

## 1. Anatomy Audit (diagnose before building)

`/anatomy` Phase 1+2. This is not a mobile screen, so the laws are reframed for a 3D action game:
the "target" is a world object; "reach" is reticle-angle + world-distance; "thumb zone" becomes
controller ergonomics + title-safe HUD area.

```
╔══════════════════════════════════════════════════════════════╗
║  ANATOMY AUDIT — on-foot interaction & input contexts         ║
╠══════════════════════════════════════════════════════════════╣
║  Surface:      Real-time 3D interaction + context stack       ║
║  Entry intent: Act on the right object; never fight controls  ║
║  Primary task: Trigger the ONE intended interaction among      ║
║                several nearby targets, instantly               ║
║  Reach model:  reticle/forward proximity + world distance      ║
║  Contexts:     OnFoot ⊕ Vehicle, overlaid by Menu / Cinematic  ║
╠══════════════════════════════════════════════════════════════╣
║  🔴 CRITICAL (design out now)                                 ║
║   • Multiple simultaneous prompts → Hick's-Law paralysis      ║
║   • Prompt flicker between near-equal targets (no hysteresis) ║
║   • Menu opens with NO default focus → controller user stuck  ║
║   • Input bleed across a context swap (enter-press re-fires)  ║
║  🟡 FRICTION                                                  ║
║   • Hold-only interaction with no toggle (accessibility)      ║
║   • Critical prompt in TV overscan edge → clipped             ║
║  🟢 OPPORTUNITY                                               ║
║   • Contextual priority (revive crew > grab cash in a fight)  ║
║     makes the single prompt feel intelligent                  ║
╚══════════════════════════════════════════════════════════════╝
```

**Lenses cited:** Hick's · Fitts's (reframed) · Information Scent · Cognitive Load · Jakob's · plus
game-native title-safe / no-dead-end-focus reachability.

---

## 2. Interaction-Prompt Prioritization (the core structural problem)

**Hick's Law:** show **exactly one primary prompt** at a time — never a cluster. When several
interactables are in range, a deterministic score picks the single winner; at most one *secondary*
hint is demoted below it (and only outside combat).

### 2.1 Scoring (deterministic, stable)
`USTATInteractionScanner` (LocalPlayer-scoped, reads the world clock) scores each candidate:

```
Score = w_angle * AngleFactor      // Fitts reframed: smaller angle from camera-forward/reticle = better
      + w_dist  * DistanceFactor    // closer in world = better (within MaxInteractRange)
      + w_ctx   * ContextPriority   // designer Gameplay-Tag priority (revive > loot > door, etc.)
      + w_avail * Availability      // locked/expended/blocked candidates score ~0
```
- **Single winner** drives the prompt; ties broken by a **stable rule** (lowest StableId) so the
  choice never depends on iteration order — `/function` determinism.
- **Hysteresis (anti-flicker):** the current winner keeps a bonus margin `H`; a challenger must beat
  it by `> H` to take over. Plus a short `MinHoldTime` from the world clock. This kills the 🔴 flicker
  between two near-equal targets — the classic stress-test failure.

### 2.2 Suppression by state (Cognitive Load)
`STAT.State.Player.*` gates what may prompt: in `Combat`/`Wanted`, only combat-relevant interactions
(revive, take cover, grab weapon) prompt; ambient loot/flavor is suppressed until the state clears.

### 2.3 Prompt content (Information Scent)
Verb-first, device-correct glyph: `Hold Ⓧ — Hotwire`, `Press Ⓕ — Enter Vehicle`. The label states the
*action*, never a vague "Interact." Glyph swaps live with the active device (§5). `Hold` vs `Press` is
explicit; every Hold interaction exposes a **toggle/tap alternative** in accessibility (§6).

---

## 3. Input-Context Structure (OnFoot / Vehicle / Menu / Cinematic)

`/anatomy` Section 8 (flow) + Jakob's Law, reframed as an **Input Mapping Context stack** owned by
`USTATInputRoutingSubsystem`. Exactly one **gameplay** context is active; **overlay** contexts stack
on top with higher priority.

```
priority ▲   Cinematic   (locks gameplay input; allows Skip/▢ + pause)      ── overlay
         │   Menu        (pauses OnFoot/Vehicle; UI navigation only)         ── overlay
         │   ───────────────────────────────────────────────────────────
         │   Vehicle     ⊕  OnFoot      (mutually exclusive gameplay)        ── base
```

Transition contracts (each is a real, wired swap via the message bus — no polling):

| From → To | Trigger | Must happen (the contract) |
|---|---|---|
| OnFoot → Vehicle | Enter interaction wins + confirmed | **Consume** the triggering input so it can't re-fire in Vehicle (kills 🔴 input bleed); remove `IMC_OnFoot`, add `IMC_Vehicle`; camera handoff; `STAT.Input.Vehicle` tag set |
| Vehicle → OnFoot | Exit input | symmetric: consume, swap IMC + camera, set `STAT.Input.OnFoot` |
| Any gameplay → Menu | Pause/Open | push `IMC_Menu`, dilate sim time to 0 (world clock), gameplay IMC suspended (not removed) |
| Menu → gameplay | Close/Resume | pop `IMC_Menu`, restore suspended IMC + time |
| Any → Cinematic | Sequencer start | push `IMC_Cinematic` (Skip + Pause only), gameplay suspended; save/stream-safe (Stage 12) |

**Jakob's Law consistency:** confirm = A/Enter, cancel/back = B/Esc, in **every** context. Enter/exit
vehicle is the genre-conventional single press. No "clever" rebinding of universal actions.

---

## 4. Controller-First Menu Focus (no dead ends)

CommonUI activatable-widget stack. The 🔴 here is a **controller user with no focus** — unrecoverable.

- **Default focus is mandatory** on every activatable widget; on activate, focus lands on the primary
  action (Resume on pause; Continue on main menu). A screen that can receive input must always have a
  focused element.
- **Focus order follows visual hierarchy** (top-start → down), grouped (Miller: ≤ ~7 per group), with
  predictable D-pad/stick spatial navigation and **defined wrap** behavior.
- **Hick's Law on top-level menus:** few options per level; depth via sub-screens, not 15-wide lists.
- **No dead ends:** every screen is reachable and exitable with the controller alone; back always
  returns to a defined parent; modals are *dismissed*, not "backed out of" (matches Stage 02 flow).
- KBM parity: mouse hover/click and arrow/Tab focus mirror the controller order; focus never desyncs
  between devices.

---

## 5. Reachability & Device Feedback

The game-native translation of "thumb zone / insets":

- **Title-safe HUD area:** critical prompts + reticle live inside the action-safe region (~90%); never
  in the outer overscan edge (the 🟡 clipped-prompt failure) — STAT's equivalent of gesture-zone insets.
- **Controller ergonomics:** the most frequent action (fire/interact) on the most accessible input
  (RT / A / Ⓕ); avoid required awkward simultaneous combos; honor input buffering so a slightly-early
  interaction press still registers.
- **Device-aware glyphs:** prompts + menu hints swap KBM ↔ gamepad the moment the active device
  changes (acceptance criterion); driven by `USTATInputRoutingSubsystem`, surfaced through tags.

---

## 6. Accessibility Assists (persist via player profile)

Stored in `USTATPlayerProfileSubsystem`, applied at input/camera layer, persisted to save:

- Full **remapping** (KBM + gamepad), **sensitivity** + **invert** per axis.
- **Hold ↔ toggle** for interact / sprint / crouch / aim (every Hold has a tap/toggle alt — §2.3).
- **Aim assist** tiers, **input buffering** window, **camera-shake reduction**, **auto-vault** toggle.
- Subtitle hooks reserved for Stage 12; colorblind/contrast for prompts handed to `/style` (Stage 13/14).
- Don't-rely-on-color: prompts pair glyph + verb text, not color alone.

---

## 7. Camera & Character (foundation for the slice)

- Third-person spring-arm camera with **collision** (probe + smooth pull-in), shoulder offset, aim
  transition; consistent across OnFoot; clean handoff to the Vehicle camera on context swap (§3).
- Locomotion: walk/sprint/crouch, **cover-ready** movement, traversal hooks (vault/mantle) exposed for
  Stage 04 combat and Stage 06 district geometry.
- All camera/interaction timing reads the **world clock** (pause-aware), never raw delta — determinism.

---

## 8. Restructure Summary (Before → After)

```
CONCERN                         NAIVE                         STAT STRUCTURE              WHY (law)
──────────────────────────────────────────────────────────────────────────────────────────────────
Multiple interactables       →  show all prompts           →  one scored winner          Hick's
Two near-equal targets       →  prompt flickers            →  hysteresis + min-hold       Fitts/stability
Prompt label                 →  "Interact"                 →  "Hold Ⓧ — Hotwire"          Information Scent
Enter vehicle                →  press bleeds into car      →  consume + boundary swap     wiring / Jakob
Open menu on controller      →  focus on nothing (stuck)   →  mandatory default focus     no dead end
Critical prompt placement    →  screen edge (overscan)     →  title-safe action area      reachability
Hold-only interactions       →  no alternative             →  hold↔toggle option          accessibility
```

---

## 9. Priority, Acceptance & Evidence

`/anatomy` Phase 4 ranking → what must be correct first:

```
🔴 FIX NOW   one-prompt scoring + hysteresis; mandatory default menu focus;
             consume-input-on-context-swap (no bleed)
🟡 HIGH      device-aware glyph swap; hold↔toggle; title-safe HUD area
🟢 POLISH    contextual-priority tuning; camera shoulder/aim feel
```
**Highest-conviction structural change:** the **single-winner interaction scorer with hysteresis** —
it resolves the Hick's-Law prompt clutter *and* the flicker instability at once, and every later system
(combat pickups, vehicle entry, mission interactables) depends on it being stable and deterministic.

| Acceptance criterion (pack §03) | Status | Evidence / where |
|---|---|---|
| KB/M **and** gamepad fully controllable | ◐ design-complete | §3, §4 — run on UE5 box |
| Prompt text + glyphs update on device change | ◐ design-complete | §5 — run on UE5 box |
| Interaction priority correct & stable | ✅ specified + testable | §2 + tests below |
| Accessibility assists present & persisted | ✅ specified | §6 |

**Automation tests (`STATTests`) to run on the toolchain:**
`Interaction.SingleWinnerDeterministic` · `Interaction.NoFlickerUnderHysteresis` ·
`InputContext.NoBleedOnVehicleSwap` · `Menu.DefaultFocusAlwaysSet` ·
`Input.GlyphSwapsWithDevice` · `Accessibility.HoldToggleEquivalence`.

---

## 10. Next Dependency

**Phase A complete → Milestone M1** (controllable character in an empty world; menu↔world loop;
KB/M + gamepad). **Stage 04 (Combat)** consumes: the interaction scanner (weapon pickup, cover,
takedown prompts), input contexts (combat sits inside OnFoot), cover-ready locomotion, and the
camera aim transition. Primary lens `/function`; supporting `/anatomy`, `/style`.
```
─────────────────────────────────────────────────────────
Stage 03 structure locked. The single-winner interaction scorer is the keystone;
combat, vehicles, and missions all build on its stability.
─────────────────────────────────────────────────────────
```
