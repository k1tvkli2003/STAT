# STAT — a living hospital game

**STAT** is a standalone, fully-3D, **AAA-scalable** game whose entire world is one **living hospital**.
It runs from a single **Unreal Engine 5 / C++** codebase on **Windows, macOS, Linux, PlayStation,
Android, and iOS**, scaling from high-end AAA fidelity down to **mid-range PCs and mobile**. The core
gameplay is clinical reasoning under pressure — investigate within a budget and clock, commit to a
diagnosis, treat, and live with the consequences — powered by the **VEX clinical engine** at the heart
of a simulated hospital of patients, staff, resources, and emergent crises.

> STAT is **educational and entertainment only** — never clinical decision support or medical advice.

This repository holds **only the prompt pack and the skills** that drive it — no game build lives here.

## Repository Map

| Path | What it is |
|---|---|
| `OVERVIEW.md` | Vision, platforms, and the unique features. |
| `Prompts/` | The authoritative prompt pack. **Start at `Prompts/00_INDEX.md`.** |
| `.claude/commands/` | The six skills (`/anatomy`, `/style`, `/function`, `/ideas`, `/dataman`, `/gamify`) — the project's governing standards. |

## How To Use This Repo

1. Read `OVERVIEW.md`, then `Prompts/00_INDEX.md` (execution guide + stage map + which skills govern each stage).
2. Keep `Prompts/01_MASTER_PROMPT.md` active during every implementation session.
3. Execute `Prompts/02_BUILD_SEQUENCE.md` in order; enforce `Prompts/03_QUALITY_GATES.md` before accepting a stage.

## The Six Skills = The Standards

`/anatomy` structure · `/style` visual · `/function` behavior · `/ideas` ideation ·
`/dataman` data · `/gamify` motivation.

The prompts **build their protocols and acceptance criteria in directly** — the skills' laws, lenses,
checklists, and zero-tolerance anti-patterns are the bar every stage must pass. Invoke a stage's
governing skill (see `Prompts/00_INDEX.md`) to design/audit that stage. The skills' reference code is
Android/Web; for STAT (UE5/C++) their **thinking layer** transfers fully, and applies most literally to
the medical UI, the VEX clinical engine, and progression/telemetry.

## Non-Negotiables

- One UE5 / C++ codebase, **C++ authority** for core systems, shipping on all six platforms.
- **Scalability is a feature:** explicit tiers so the same game runs on mobile/mid-range and high-end.
- The whole world is a **hospital** — fully 3D, authentic, original art.
- STAT is **educational/entertainment only**, never clinical decision support; medical content is
  authored responsibly and validated.
- All player-facing text, tools, comments, and docs in English and localization-ready.
