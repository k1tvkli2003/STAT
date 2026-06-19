# STAT — Street Tactical Assault Team

STAT is a standalone **Windows PC**, **Unreal Engine 5**, **C++-first** open-world action game about
high-pressure urban operations, crew loyalty, escalating heat, heavy driving, and systemic missions.
Its ambition is AAA presentation, but its production plan proves the game through a measurable
**vertical slice** before scaling the city.

`VEX Protocol` is an optional **educational** clinical cost-duel mode that shares platform
infrastructure but stays isolated from campaign state and fiction. It is educational only and must
never present itself as clinical decision support.

## Repository Map

| Path | What it is |
|---|---|
| `OVERVIEW.md` | Vision, pillars, and unique features. |
| `Prompts/` | The authoritative V3 prompt pack — master prompt, 20-stage build sequence, quality gates. Start at `Prompts/00_INDEX.md`. |
| `Plan/` | The execution plan that fuses the build sequence with the six design skills. **Start at `Plan/00_EXECUTION_PLAN.md`.** |
| `Docs/` | Concrete stage deliverables as they are produced (Stage 01 foundations is first). |
| `.claude/commands/` | The six installed design skills (`/anatomy`, `/style`, `/function`, `/ideas`, `/dataman`, `/gamify`). |

## How To Use This Repo

1. Read `OVERVIEW.md`, then `Prompts/00_INDEX.md` for the stage map.
2. Read `Plan/00_EXECUTION_PLAN.md` — it sequences the work and says **which skill to invoke at which stage**.
3. Keep `Prompts/01_MASTER_PROMPT.md` active during every implementation session.
4. Execute `Prompts/02_BUILD_SEQUENCE.md` in order; enforce `Prompts/03_QUALITY_GATES.md` before accepting a stage.
5. Track progress in `Plan/02_PROGRESS_LOG.md`.

## The Six Skills (installed as project slash commands)

`/anatomy` structure · `/style` visual · `/function` behavior · `/ideas` ideation ·
`/dataman` data · `/gamify` motivation.

They are committed in `.claude/commands/` and become available as slash commands in new Claude Code
sessions opened in this repo. Their reference code targets Android/Web; for STAT (UE5/C++) they are
used as **thinking lenses** and applied most directly to UI/HUD (Stage 14), VEX Protocol (Stage 18),
and progression/telemetry (Stage 16). See `Plan/01_SKILLS_AND_STAGES.md`.

## Non-Negotiables

- Windows PC only. Unreal Engine 5 with **C++ authority** for core systems.
- Vertical slice first — no city-scale expansion before the slice proves fun, performance, save
  integrity, accessibility, and stability.
- VEX Protocol stays isolated and clearly educational.
- All player-facing text, tools, comments, and docs in English and localization-ready.
