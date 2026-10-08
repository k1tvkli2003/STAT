# STAT — Critical Shift

STAT: Critical Shift is a story-driven, systems-driven 3D game set entirely in a fictional teaching hospital. Over one critical shift, the player manages patients, ward capacity, staff energy, equipment, time, and incomplete information at once. Target feel is cinematic AAA realism in a living hospital — no copy of any existing game, brand, or asset.

This is entertainment/fiction. Not medical training, treatment advice, or a clinical decision tool.

## Design direction

- **Space:** one multi-layer hospital — ambulance bay, triage, ED, ICU, OR, imaging, lab, pharmacy, service corridors, control room, rooftop.
- **Core loop:** observe, prioritize, command or act, face systemic consequences, review and improve.
- **Player:** a fixed fictional operations coordinator in third person — moves through the space, reads the situation, registers allowed intents, suggests and delegates to staff. Full abstract skill sequences exist only in Practice/Free Shift modes.
- **Modes:** single-player, optional online co-op, accessibility modes. No pay-to-win, no gambling mechanics.
- **Architecture rule (kept from the earlier package):** simulation, UI, and durable data stay independent and contract-driven; the UI is never the source of truth for game rules.
- **Engine decision:** Unreal Engine 5.x, C++, Gameplay Framework, Enhanced Input, CommonUI, Niagara, MetaSounds, World Partition/level streaming. React/TypeScript only for internal tools, production dashboards, or a companion site — never for render or gameplay.

## What's in the folder

- `Data/` — 25 numbered design registries (patients, staff, zones, equipment, systems cascade, UI/accessibility, audio/VFX, progression, release ops, narrative agency, QA/failure recovery, asset/dialogue/localization catalogs, balance, routes, permissions, events, fixtures) plus compiled output and scenario drafts.
- `Prompts/` — 100+ production prompts behind `00_Master_Production_Protocol.md`: game loop, time management, vitals simulation, triage/bed management, arrest system, world blockout, animation/audio/VFX manifests, save/CI/privacy/release registries. Read the master protocol first; short prompts alone do not authorize execution.
- `Tools/` — content, migration, native-contract, and validation tooling.
- `Evidence/` — migration evidence; `work/` holds orchestration and verification notes.
- `E/` — secondary evidence tree.

Per `OVERVIEW.md`, no platform counts as supported until a real build/package/install/smoke test on matching hardware says so. Active targets are Windows, Linux, Android, installable PWA (for Apple devices, per `ADR_PWA_001` — no native iOS/macOS deliverables), and PlayStation (console path only in an authorized environment).

## Tech stack

Unreal Engine 5.x / C++ (game runtime), Markdown design registries and prompt packs (pre-production), validation/migration tooling.

## Status

Pre-production: design registries, production protocol, and prompt packs exist. No engine build or platform evidence is claimed in-tree — quality, frame-rate, and platform claims require measured matrices and real device tests first.
