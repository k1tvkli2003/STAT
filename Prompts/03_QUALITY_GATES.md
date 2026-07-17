# STAT Quality Gates

A stage is incomplete until every applicable gate has evidence. Do not accept claims without compile logs
(per target), automation results, packaged smoke tests **on desktop and mobile tiers**, profiling
captures **per tier**, debug screenshots, or explicit manual verification notes. These gates **encode the
six skills' acceptance criteria and zero-tolerance anti-patterns** — passing a stage means passing them.

## Build, Platforms & Playability
- Every milestone ends in a controllable build; vertical-slice milestones require packaged builds on **at
  least one desktop tier and one mobile tier** (not editor-only, not Windows-only).
- Editor + per-platform targets compile cleanly; runtime/editor/test modules load. The PlayStation path
  is architected and documented (built on licensed SDK/hardware).
- Build scripts and setup are reproducible from documented commands; no private credentials or local-only
  paths required.

## Scalability (a first-class feature)
- Every performance-sensitive system has a **budget per platform tier** (high-end → mobile) with a stat
  group / trace marker and a captured measurement on desktop **and** mobile.
- The same game is playable and readable on the mobile/mid-range tier — reduced fidelity, same systems.
- Device profiles / scalability settings are defined and verified, not left at engine defaults.

## `/function` — Behavior & Performance (zero tolerance)
- Every control/input/ability resolves to a **real effect** — no bound-but-dead node, no `TODO()` logic.
- **One authority per fact**; no duplicated state that can drift (vitals, budget, reputation, save data).
- Cross-feature **integration contracts fire** (`action → writes → observer → recompute`) and are tested.
- Every state has all arms — Loading/Active/**Empty**/Error/Recover — **no soft-lock** anywhere.
- No blocking work on the game thread; event-driven over polling; tick economy respected.
- Hostile-condition resilience: guards/idempotency on rapid input, timeouts + backoff on external calls,
  cleanup on streaming unload (no leaked actors/handles).
- No swallowed errors; no speedup claimed without a measurement plan.

## `/anatomy` — Structure & Reachability (zero tolerance)
- Reasoned from UX laws; no dead ends; **mandatory default focus** for gamepad/touch (never "stuck").
- The one primary action is always reachable; menu depth shallow and grouped (Hick/Miller); prompt
  prioritization is single-winner and stable (no flicker).
- Consistent confirm/cancel/back semantics across every context; title-safe HUD; touch reach honored.

## `/style` — Visual & Accessibility (zero tolerance)
- **WCAG AA contrast** on all text; never color-only signaling (critical for clinical UI).
- One hierarchy, one token system, consistent radius/spacing/motion; **motion-with-meaning** on key state
  changes (never a silent change).
- **Light + dark** and **localization/RTL-ready** on every surface; low/mobile tier preserves feedback.

## `/ideas` — Feature Quality
- Every feature passes the quality bar: specific, grounded in the design, user-first, novel, effort-
  honest, reason-to-exist-now; it strengthens the core loop and respects vertical-slice-first.
- Features that fight the loop or fail the kill criteria are cut, not shipped.

## `/dataman` — Data & Medical Content (zero tolerance)
- All datasets (cases, agents, economy, dialogue, assets): unique IDs, reference integrity, distribution
  balance, UI-safe strings, recorded provenance.
- **No synthetic data presented as real**; clinical cases are synthetic-illustrative, reviewer-tagged,
  validated; no secrets committed; Jules used only after a written brief with local validation.
- Save-affecting data uses stable IDs, versioned schemas, tested migrations, corruption recovery.

## `/gamify` — Motivation, Scoring & Telemetry (zero tolerance)
- Rewards mastery and meaningful completion (accurate diagnosis, sound investigation, resources retained,
  avoided harm); never grind, guessing, or empty engagement.
- Event-ledger with **idempotency, caps, cooldowns, audit logs**; rewards cannot be farmed via retries or
  sync duplication.
- Reduced-motion parity; progression never distorts clinical correctness or hides essentials behind gates.

## Determinism & Save Integrity
- Scored/saved/replayed/synchronized/**VEX** systems use stable event ordering + seeded RNG (reproducible).
- Save/load restores the slice accurately across platforms; corrupt saves fail safely with visible
  recovery.

## Robustness & Recovery
- Missing assets, invalid data, streaming transitions, corrupted saves, input/device changes, low tier,
  paused state, per-platform differences, disconnects, and optional-mode disablement all recover
  gracefully. Failure states are player-readable and never soft-lock the slice.
- Debug commands/visualizations exist for lifecycle, hospital sim, agents, VEX clinical, treatment,
  triage/crisis, economy, save, streaming, and telemetry where relevant.

## Medical Responsibility (non-negotiable)
- STAT presents as **educational/entertainment only**, never clinical decision support; a disclaimer is
  always reachable.
- Clinical content is validated and responsibly authored; scoring never rewards unsafe or wasteful
  clinical behavior.

## Security, Privacy & Online
- Telemetry is consent-aware, privacy-minimizing, and disableable.
- Online authority, replication trust boundaries, anti-cheat posture, and secret handling are explicit
  where relevant.

## Delivery Evidence
- Changed files and assets listed; commands, automation, profiles (per tier), packaged checks, and manual
  verifications reported exactly; known limitations and deferred work explicit; the next dependency named.
- The stage explicitly confirms it triggers **none** of the governing skills' zero-tolerance anti-patterns.
