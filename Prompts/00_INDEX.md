# STAT V3 Prompt Pack

Use this pack to build **STAT**, a standalone Windows PC open-world action game in Unreal Engine 5 with C++ as the authoritative implementation layer.

## How To Execute

1. Keep `01_MASTER_PROMPT.md` active throughout every implementation session.
2. Execute `02_BUILD_SEQUENCE.md` in order; each numbered section is a separate prompt and milestone.
3. Enforce `03_QUALITY_GATES.md` before accepting a stage.
4. End each stage with changed files, build/test/profile evidence, assumptions, known limits, and the next dependency.
5. Build the vertical slice first. Do not scale the city, systems, content, co-op, or optional modes before the slice proves core fun, performance, save integrity, and stability.

## Delivery Strategy

`Prototype -> Vertical Slice -> Production Systems -> Content Scale -> Shipping`

The vertical slice is one dense district, one safehouse, one complete mission chain, one pursuit, one combat space, one tuned hero vehicle, representative traffic/crowds, core save/load, and a stable packaged Windows build.

## Non-Negotiables

- Windows PC only.
- Unreal Engine 5 with C++ authority for core gameplay, simulation, save, mission, combat, vehicle, AI, economy, performance-sensitive, and online rules.
- Blueprints may compose, tune, animate, and expose content-facing values, but they cannot conceal untestable core architecture.
- Every system must define ownership, lifetime, threading, replication relevance, save impact, data contracts, validation, debug tooling, and performance budget.
- Player-facing text, editor tools, comments, and documentation stay in English and localization-ready.

## Stage Map

| Stage | Focus | Completion Signal |
|---|---|---|
| 01 | Project, modules, toolchain, budgets | Editor target compiles with CI/build guidance |
| 02 | Lifecycle and data architecture | Front-end-to-world loop and validated data contracts |
| 03 | Input, camera, character, interaction | Controllable character and accessible input foundations |
| 04 | Combat vertical slice | Readable combat arena with tests and profiling |
| 05 | Vehicle vertical slice | Tuned hero vehicle, entry/exit, damage, persistence |
| 06 | Dense district and streaming | One performant district with HLOD/streaming evidence |
| 07 | Traffic, crowds, civilians | Scalable population and reaction systems |
| 08 | Police heat and pursuit | Witness, dispatch, pursuit, search, cooldown loop |
| 09 | Mission framework | One complete mission with objectives and recovery |
| 10 | Factions/economy/consequences | Persistent district state without soft locks |
| 11 | Crew and safehouse | Useful crew, services, injuries, loyalty |
| 12 | Narrative/cinematic pipeline | Localization-ready dialogue and mission cinematics |
| 13 | Time/weather/audio/VFX/destruction | Atmosphere systems with explicit budgets |
| 14 | UI/map/phone/HUD/accessibility | CommonUI, settings, map, save/load, PC UX |
| 15 | Save/checkpoint/migration | Atomic versioned saves and restore tests |
| 16 | Progression/rewards/telemetry | Auditable rewards and privacy-aware telemetry |
| 17 | Optional co-op | Only after single-player slice is stable |
| 18 | VEX Protocol | Separate educational mode, independent saves |
| 19 | Performance/scalability/stability | Representative captures and stability evidence |
| 20 | Pipeline/QA/packaging/release | Reproducible Shipping build and release docs |

## Final Definition Of Done

STAT is done only when the packaged Windows build proves the dense-district vertical slice is fun, stable, performant, save-safe, accessible, and extensible; the optional VEX Protocol remains isolated; and every expansion system has validation, profiling, and recovery evidence.
