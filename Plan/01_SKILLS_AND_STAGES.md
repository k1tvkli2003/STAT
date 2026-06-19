# STAT — Skills × Stages Integration

How each of the six installed skills is used on a UE5/C++ game, what it maps to, and exactly when to
invoke it. Companion to `Plan/00_EXECUTION_PLAN.md` §4–5.

---

## A. Concept Translation (Android/Web skill term → UE5/STAT equivalent)

The skills speak Compose/Room/Web. Read them through this table so their reasoning lands on STAT.

| Skill term (as written) | STAT / UE5 equivalent |
|---|---|
| Screen / Composable | UMG/CommonUI Widget, HUD layer, menu surface |
| Navigation / NavHost / routes | CommonUI activatable-widget stack, game flow / GameInstance subsystem states |
| ViewModel / `BaseViewModel<S,I,E>` | View model object / UI controller + `UGameInstanceSubsystem` / `UWorldSubsystem` |
| State / Intent / Effect (MVI) | Replicated/owned state struct, input/gameplay event, one-shot Gameplay Message/Effect |
| Room DAO / entity / migration | SaveGame schema + `UPrimaryDataAsset`; save version + migration step |
| `Flow` (hot) vs `suspend` (one-shot) | Event/delegate subscription vs polled getter; prefer event-driven |
| Hilt singleton / single source of truth | One owning subsystem/component; no duplicated authority |
| Recomposition economy | Widget invalidation / tick cost; avoid per-frame UMG rebuilds & per-tick polling |
| Main-thread discipline (ANR) | Game-thread budget; push heavy work to async tasks / worker threads |
| Tailwind tokens / MaterialTheme | UMG style assets, a shared design-token data asset, Slate styles |
| Framer Motion / Compose animation | UMG animations, Sequencer, material/Niagara feedback, tween curves |
| RTL / Persian (§12 of `/style`) | Localization-ready UMG + bidi text; FText everywhere, no baked English layout |
| LeakCanary / Macrobenchmark / Profiler | Unreal Insights, `stat` commands, memreport, automation perf tests |
| Supabase / backend | Online subsystem abstraction + telemetry sink (no vendor lock-in) |

**Golden rule:** transfer the *law, lens, checklist, anti-pattern*. Do **not** emit Kotlin/Compose/CSS
into STAT — the master prompt forbids web/React/TS and mandates C++ authority.

---

## B. Per-Skill Charter For STAT

### `/anatomy` — Structure (IA, flow, reachability)
- **Owns:** Stage 03 (interaction/prompt priority), Stage 09 (mission/objective flow & legibility),
  Stage 14 (UI/HUD/map/phone/menus), Stage 18 (VEX case flow).
- **Transfers cleanly:** Hick's/Miller's/Jakob's/Fitts's laws → menu depth, HUD element count, input
  prompt prioritization, settings grouping, pause/save flow, controller focus order.
- **Reads as analogy:** "thumb zones / 44dp targets / bottom-nav" → controller focus navigation,
  gamepad-first menu layout, gaze/reticle ergonomics, safe-area & HUD edges.
- **STAT anti-patterns it catches:** HUD overload during a chase (too many simultaneous signals →
  Miller/Hick); buried but frequently-used actions (phone/map deep in menus); inconsistent back/close
  semantics between menus; objective stack that doesn't show the next action.

### `/style` — Visual (color, type, motion, feedback)
- **Owns:** Stage 13 (atmosphere/audio-visual mix), Stage 14 (visual language of UI/HUD), supports 03/04/05/18.
- **Transfers cleanly:** contrast/WCAG, hierarchy (size→weight→color→position), motion-with-meaning,
  colorblind-safe states, consistent token system → applies directly to HUD, damage/heat feedback,
  menus, VEX result timeline.
- **Reads as analogy:** Compose/Tailwind snippets → UMG style assets, material params, Niagara, Sequencer.
- **STAT must-haves it enforces:** never a static state change (chase start, wanted-level up, mission
  fail all need readable motion + sound + color that survive low settings and colorblind modes);
  light/dark and localization/bidi-readiness baked into every surface.

### `/function` — Behavior (wiring, state, perf, resilience)
- **Owns:** the engineering spine — Stages 01, 02, 04, 05, 06, 07, 08, 15, 17, 19; audits everything.
- **Transfers cleanly (this is the most universal skill):** every control resolves to a real effect
  (no dead buttons / empty Blueprint events); single source of truth (no duplicated authority that
  drifts — e.g. wanted level owned in one place); integration contracts must fire (finish objective →
  write mission state → reward → save → HUD update); exhaust all states (Loading/Success/**Empty**/Error
  → mission/encounter has intro/active/fail/recover/win); main-thread/game-thread discipline; resilience
  (offline/corrupt-save/double-input/rotation→alt-tab/low-end).
- **Reads as analogy:** Room/coroutines → SaveGame/async tasks/worker threads; StrictMode/LeakCanary →
  Unreal Insights/`stat`/memreport/`MallocLeakDetection`.
- **STAT contract table to keep filled (per feature):** `action → what it must persist → who observes
  it → what recomputes` (e.g. *crime committed → witness/evidence record → heat system → dispatch &
  HUD wanted meter*). An assumed-but-never-invoked contract is a 🔴.

### `/ideas` — Ideation (what makes STAT singular)
- **Owns:** creative pressure-testing — strongest at Stage 08 (signature pursuit moments), Stage 11
  (crew identity), and any "this feels generic" gap; light touch elsewhere.
- **Transfers cleanly:** First-Principles / JTBD / SCAMPER / Inversion / Analogous-Domains / Data-Leverage
  lenses → applied to STAT's core loop ("dense reactive city," "heat that remembers"), retention hook,
  and the unfair advantage (systemic collisions between mission/traffic/police/weather/crew).
- **Discipline it imposes:** every idea tied to something real in the design, effort-honest, killable.
  Prevents feature-creep that fights the vertical-slice-first rule.

### `/dataman` — Data (catalogs, schemas, validation, Jules)
- **Owns:** Stage 10 (economy tables), Stage 11 (crew data assets), Stage 12 (dialogue/subtitle data),
  Stage 18 (VEX clinical case corpus), Stage 20 (content validation suite).
- **Transfers cleanly:** dataset audit → brief → expand-in-layers (repair/complete/enrich/stress/document)
  → validate (unique IDs, reference integrity, distribution, UI-safe strings, no secrets). Maps directly
  to `UPrimaryDataAsset` catalogs + validators + automation content tests.
- **Jules delegation:** usable for *large reviewable data creation* (e.g. expanding the VEX case bank or
  vehicle/crew/economy tables) **only after a written data brief**, with `JULES_API_KEY` from env (never
  committed), plan-approval for broad changes, and **local validation before merge**. Codex/STAT keeps
  ownership of schema, prompts, validation, and integration.
- **Hard rule for STAT:** VEX clinical content is **synthetic-illustrative, educational only**, never
  presented as real patient data or clinical decision support; provenance recorded.

### `/gamify` — Motivation (progression that rewards mastery)
- **Owns:** Stage 16 (progression/rewards/challenge/telemetry), Stage 18 (VEX scoring & mastery), supports Stage 10.
- **Transfers cleanly:** event-ledger architecture (`action → event → rule engine → reward →
  derived progress`), idempotency + caps + cooldowns (anti-farming — directly satisfies Stage 16's
  "rewards cannot be farmed through retries or sync duplication"), reward-moment tiers & motion specs,
  reduced-motion/accessibility parity.
- **STAT framing:** reward **skill and meaningful completion** (clean pursuit escapes, mission mastery,
  crew loyalty, VEX diagnostic accuracy & budget discipline) — never empty engagement or grind. The
  ledger's determinism + idempotency is also exactly what the save/telemetry gates require.
- **Prime-directive guard:** gamification amplifies STAT's real value; it must not distort the fiction
  or pacing, and (per OVERVIEW) VEX education stays separated from campaign scoring.

---

## C. When To Invoke Which Skill (quick trigger list)

| If the work is about… | Invoke |
|---|---|
| Where something lives, menu depth, prompt/objective ordering, controller focus | `/anatomy` |
| How it looks/feels/animates, feedback readability, colorblind/contrast, atmosphere | `/style` |
| Does it actually work/connect/persist/perform; dead controls; leaks; races | `/function` |
| "This feels generic / what's our hook"; new systemic moments | `/ideas` |
| Catalogs, schemas, seed/case data, validation, bulk reviewable content | `/dataman` |
| Progression, rewards, challenges, anti-farming, VEX scoring, telemetry shape | `/gamify` |

Typical build chain for a screen/system:
`/ideas` (what & why) → `/anatomy` (where & order) → `/style` (look & motion) →
`/function` (make it work, connect, fly) → `/dataman` (feed it real data) → `/gamify` (make progress felt).

---

## D. Skill Output ↔ Quality Gate Crosswalk

Each skill's discipline satisfies specific gates in `Prompts/03_QUALITY_GATES.md`:

| Gate area | Satisfied largely by |
|---|---|
| Build & Playability | `/function` |
| Vertical Slice Discipline | `/ideas` (scope honesty) + plan governance |
| C++ Authority & Data Integrity | `/function` + `/dataman` |
| Robustness & Recovery | `/function` |
| PC Quality & Accessibility | `/anatomy` + `/style` |
| Performance & Stability | `/function` |
| Security, Privacy, Online | `/function` + `/gamify` (telemetry consent) |
| Delivery Evidence | the per-stage REPORT step (all skills) |

If skill advice and a gate conflict, **the gate wins**; log the trade-off in `Plan/02_PROGRESS_LOG.md`.
