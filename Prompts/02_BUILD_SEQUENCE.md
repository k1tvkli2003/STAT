# STAT Build Sequence

Each section below is an independent implementation prompt governed by `01_MASTER_PROMPT.md`. Apply `03_QUALITY_GATES.md` before accepting a stage.

## 01 - Project, Modules, Toolchain, And Budgets

Create the UE5 C++ project, runtime/editor/test modules, plugin boundaries, coding rules, target configs, source-control/LFS guidance, Asset Manager rules, Gameplay Tags, CI/build scripts, crash symbols, and explicit CPU, GPU, frame-time, memory, streaming, package-size, load-time, and minimum-spec budgets.

Deliverables:

1. Compiling Editor target and empty startup map.
2. Module/plugin structure for core, UI, campaign, world, combat, vehicles, AI, missions, save, audio, tools, tests, and VEX.
3. Asset naming rules, folder conventions, Primary Asset Types, Gameplay Tag hierarchy, and validation commands.
4. Build scripts for clean compile, tests, and packaged smoke build.
5. Architecture and performance-budget documentation.

Acceptance criteria:

- A new developer can build the Editor target from documented steps.
- CI/build scripts do not require private credentials.

## 02 - Core Lifecycle And Data Architecture

Define game flow, subsystem ownership, event contracts, clocks, deterministic random streams, save IDs, Primary Data Assets, validators, debug commands, and automation-test harnesses. Deliver an empty but robust front-end-to-world loop.

Deliverables:

1. GameInstance, World, LocalPlayer, and Player subsystem ownership map.
2. Front end, loading, new game, continue, world entry, pause, and quit flow.
3. Typed event bus or message contracts where needed.
4. Primary Data Asset base classes, validation framework, stable IDs, schema versions, and debug commands.
5. Automation tests for lifecycle flow, invalid data, deterministic RNG, and asset validation.

Acceptance criteria:

- The game can transition from menu to world and back without leaking state.
- Invalid data assets fail validation with actionable messages.

## 03 - Input, Camera, Character, And Interaction

Implement Enhanced Input for keyboard/mouse and gamepad, remapping, context switching, third-person camera collision, locomotion, traversal foundation, interaction scanning, prompts, accessibility assists, and deterministic interaction tests.

Deliverables:

1. Input actions, mapping contexts, glyph support, remapping, hold/toggle options, sensitivity, and inversion.
2. Character locomotion, camera orbit/collision, sprint, crouch, cover-ready movement, and traversal hooks.
3. Interaction scanner with prioritization, prompts, focus rules, and debug visualization.
4. Accessibility assists for aim, camera shake, hold actions, subtitles hooks, and input buffering.
5. Tests for mapping changes, context switching, interaction priority, and pause/menu transitions.

Acceptance criteria:

- Keyboard/mouse and gamepad are both fully controllable.
- Prompt text and glyphs update with input device changes.

## 04 - Combat Vertical Slice

Build weapon data, aiming, recoil, spread, reload, damage, armor, hit reactions, cover, takedowns, threat indicators, AI perception, combat states, encounter director, and a representative combat arena.

Deliverables:

1. C++ weapon, damage, health, armor, ammo, recoil, spread, reload, and hit-reaction systems.
2. Cover interaction, target acquisition, threat indicators, suppression hooks, and readable enemy telegraphs.
3. AI perception, combat StateTree/Behavior Tree, EQS where useful, and encounter director.
4. Combat arena map with debug spawns and designer-tunable Data Assets.
5. Automation tests and profiling for traces, projectiles, animation events, VFX, and AI tick cost.

Acceptance criteria:

- The arena supports a complete encounter loop: enter, engage, recover, win/fail.
- Combat remains readable at low settings.

## 05 - Vehicle Vertical Slice

Implement Chaos vehicle data, entry/exit, camera, assists, damage, repair, surface response, traffic collision, garage spawn, persistence, and one tuned hero vehicle.

Deliverables:

1. Vehicle pawn/component architecture with data-driven tuning.
2. Enter/exit flow, seat rules, vehicle camera, input assists, handbrake, reverse, horn/siren hooks, and damage.
3. Surface response, collision handling, repair, respawn, garage spawn, and save persistence.
4. One hero vehicle tuned for keyboard/mouse and gamepad at multiple frame rates.
5. Tests and profiling for physics stability, input latency, camera collision, and save restore.

Acceptance criteria:

- Driving feels heavy but controllable.
- Vehicle state survives save/load and world transitions.

## 06 - Dense District And Streaming

Create one production-quality district using World Partition, Data Layers, HLOD, level instances, navigation, occlusion, streaming sources, traversal metrics, lighting scenarios, and automated streaming walks.

Deliverables:

1. Slice district map with roads, alleys, interiors or entry points, safehouse location, combat space, pursuit routes, and landmarks.
2. World Partition/Data Layer setup, HLOD strategy, navigation, occlusion, streaming sources, and lighting profile.
3. Traversal metrics for foot, vehicle, chase, and mission routes.
4. Automated streaming walk/drive tests and Insights captures.
5. Content-density budget before expansion.

Acceptance criteria:

- The district streams without major hitches on target hardware.
- Content density feels intentional before map scale expands.

## 07 - Traffic, Crowds, And Civilian Reactions

Build scalable traffic lanes, intersections, parking, spawn budgets, pedestrian zones, reactions, panic, reporting, evacuation, pooling, LOD/significance, stuck recovery, and debug heatmaps. Use Mass only where profiling demonstrates value.

Deliverables:

1. Traffic lane/intersection data, vehicle spawners, parking, despawn, and stuck recovery.
2. Pedestrian zones, civilian state machine, panic, witness, reporting, evacuation, and avoidance.
3. Significance, LOD, pooling, budget controls, and debug heatmaps.
4. Tests for spawn limits, blocked routes, reporting events, and cleanup.
5. Performance captures for traffic/crowd density tiers.

Acceptance criteria:

- Traffic and civilians react believably without overwhelming CPU budget.
- Reporting can feed the police heat system.

## 08 - Police Heat And Pursuit Director

Implement witnessed crimes, evidence, district heat, dispatch, search areas, line-of-sight memory, escalation tiers, roadblocks, helicopter hooks for later, cooldown, disguises/vehicle recognition, arrest/failure, and anti-spawn-cheating rules.

Deliverables:

1. Heat model, crime events, witness/evidence pipeline, dispatch rules, and pursuit director.
2. Search area, last-known position, line-of-sight memory, escalation, roadblocks, cooldown, and escape rules.
3. Police AI behavior, vehicle pursuit hooks, arrest/failure flow, and player feedback.
4. Debug tools for heat, dispatch, search radius, spawn sources, and recognition state.
5. Tests for escalation, cooldown, escape, save/load, and invalid spawn conditions.

Acceptance criteria:

- A complete chase can start, escalate, search, and resolve.
- Police do not spawn unfairly in visible impossible locations.

## 09 - Mission Framework And Systemic Objectives

Create data-driven mission definitions, objective graph, triggers, checkpoints, fail/recovery policy, world-state conditions, dialogue hooks, rewards, replay, validation, and debug skipping. Ship one full mission that supports stealth, combat, and vehicle escape without bespoke engine hacks.

Deliverables:

1. Mission Data Asset schema with objectives, conditions, rewards, checkpoints, dialogue hooks, and validation.
2. Objective graph runtime with trigger handling, fail/recover policy, debug skip, and replay support.
3. Checkpoint save integration and mission-state restoration.
4. One complete mission chain using existing combat, vehicle, district, traffic, and police systems.
5. Tests for objective ordering, invalid data, checkpoint restore, failure recovery, and replay.

Acceptance criteria:

- One mission can be completed through multiple supported approaches.
- Mission logic uses systems, not one-off hacks.

## 10 - Factions, District State, Economy, And Consequences

Model reputation, territory pressure, prices, access, retaliation, informants, safehouses, mission availability, and persistent consequences. Keep economy sources/sinks auditable and prevent irreversible soft locks.

Deliverables:

1. Faction and district-state models with stable save IDs.
2. Economy ledger for money, favors, access, repairs, gear, safehouse upgrades, and mission rewards.
3. Consequence events that affect heat, prices, mission availability, crew, and district behavior.
4. Soft-lock prevention rules and recovery options.
5. Tests for economy transactions, save migration, reputation thresholds, and mission gating.

Acceptance criteria:

- Consequences are meaningful without permanently trapping the player.
- Economy changes are auditable and deterministic where scored.

## 11 - Crew And Safehouse

Implement recruitable crew roles, loyalty, injuries, availability, perks, banter hooks, assignment, relationship events, garage/loadout services, and safehouse upgrades.

Deliverables:

1. Crew member Data Assets, runtime state, loyalty, injury, availability, perks, and role rules.
2. Safehouse services for garage, loadout, planning, recovery, upgrades, and crew interactions.
3. Assignment system for missions and support roles.
4. Banter/event hooks with localization-ready text.
5. Tests for availability, injury recovery, save/load, assignment conflicts, and perk effects.

Acceptance criteria:

- Crew affects play without becoming spreadsheet micromanagement.
- Crew state restores accurately from saves.

## 12 - Narrative, Dialogue, And Cinematic Pipeline

Define narrative state, dialogue conditions, localization-ready text, subtitles, performance capture hooks, Sequencer conventions, skip/replay, camera safety, save interaction, and cinematic streaming. Ship a complete mission intro/outro pipeline.

Deliverables:

1. Narrative state model and dialogue condition system.
2. Subtitle and localization data contracts.
3. Sequencer conventions, camera safety rules, skip/replay behavior, and streaming policy.
4. Mission intro/outro implementation for the vertical slice.
5. Tests/checks for missing subtitles, invalid dialogue IDs, save during cinematic, and skipped sequences.

Acceptance criteria:

- Cinematics do not break save, input, or streaming.
- All spoken or important narrative content has subtitle support.

## 13 - World Time, Weather, Audio, VFX, And Destruction

Integrate time-of-day, authored weather transitions, wetness/visibility effects, MetaSounds ambience, music states, vehicle/combat mix, Niagara budgets, decals, breakables, and Chaos destruction only where gameplay value justifies cost.

Deliverables:

1. Time/weather state with mission and district integration.
2. Audio state system for ambience, music, combat, vehicle, pursuit, UI, and accessibility.
3. VFX/decal/breakable budgets and scalability rules.
4. Destruction hooks for authored gameplay moments only where performance allows.
5. Profiling captures and debug toggles for each system.

Acceptance criteria:

- Atmosphere improves gameplay readability rather than obscuring it.
- Low settings preserve essential feedback.

## 14 - UI, Map, Phone, HUD, And Accessibility

Build CommonUI front end, HUD, interaction prompts, minimap/map, mission log, crew/garage screens, phone surface, settings, save/load, input glyphs, ultrawide and resolution scaling.

Deliverables:

1. CommonUI shell with main menu, pause, settings, save/load, and profile surfaces.
2. HUD for health, armor, ammo, heat, mission objectives, vehicle state, interactions, and pursuit feedback.
3. Map/minimap, phone, mission log, crew, garage, and safehouse screens.
4. Accessibility options for subtitles, color alternatives, aim/drive assists, reduced camera shake, hold/toggle, remapping, text size, and audio mix.
5. Tests/manual checks for keyboard/mouse, gamepad, ultrawide, scaling, pause, and settings persistence.

Acceptance criteria:

- UI is readable and controllable across PC display modes.
- Accessibility options persist and affect gameplay.

## 15 - Save, Checkpoints, Migration, And Recovery

Implement versioned campaign snapshots, stable object IDs, asynchronous saves, atomic writes, checkpoint scope, streamed-world restoration, corruption fallback, migration tests, multiple slots, cloud-provider abstraction, and explicit autosave indicators.

Deliverables:

1. Save schema with stable IDs, versioning, migration, and subsystem ownership.
2. Async save/load, atomic writes, slot management, autosave indicators, and corruption fallback.
3. Checkpoint policy for missions, pursuits, combat, vehicles, crew, economy, and district state.
4. Cloud-provider abstraction without vendor lock-in.
5. Tests for migration, corruption, interrupted save, streamed restore, and multiple slots.

Acceptance criteria:

- Save/load restores the vertical slice accurately.
- Corrupt saves fail safely with user-visible recovery.

## 16 - Progression, Rewards, Challenge, And Telemetry

Create progression and unlocks that broaden play rather than inflate numbers, mission grading, difficulty assists, challenge definitions, achievements, privacy-aware telemetry, economy dashboards, and anti-farming rules.

Deliverables:

1. Progression ledger with idempotent reward events and source attribution.
2. Unlocks, challenges, achievements, mission grading, assists, and difficulty settings.
3. Telemetry schema with consent, PII minimization, offline buffering, and debug viewer.
4. Economy dashboard for balance inspection.
5. Tests for duplicate rewards, challenge completion, difficulty effects, and telemetry consent.

Acceptance criteria:

- Rewards cannot be farmed through retries or sync duplication.
- Telemetry can be disabled without breaking gameplay.

## 17 - Optional Co-op Architecture

Only after the single-player slice is stable, define server authority, session flow, replication graph, relevancy, prediction boundaries, join/leave, mission ownership, vehicle seats, crew roles, disconnect recovery, anti-cheat posture, and network profiling.

Deliverables:

1. Co-op feasibility ADR and scope boundary.
2. Session, authority, replication, relevancy, prediction, and ownership contracts.
3. Narrow prototype for one mission support role if approved by slice stability.
4. Disconnect/reconnect and save compatibility plan.
5. Network profiling and anti-cheat risk notes.

Acceptance criteria:

- Co-op does not destabilize single-player architecture.
- Systems are not blindly retrofitted without ownership decisions.

## 18 - VEX Protocol Mode

Implement a separate module and game flow for educational clinical cost-duel cases.

Deliverables:

1. Independent VEX module, menu entry, save namespace, settings reuse, and disclaimer.
2. Versioned case Data Assets with budgeted investigations, delayed results, diagnosis commitment, scoring, debrief references, reviewer metadata, and correction history.
3. Deterministic event log for purchased tests, results, decisions, timing, safety penalties, and final scoring.
4. UI for case briefing, budget, test shop, results timeline, final diagnosis, debrief, and replay.
5. Tests for scoring, invalid cases, save isolation, content validation, and disclaimer visibility.

Acceptance criteria:

- VEX never mutates campaign state or fiction.
- It is clearly educational and not clinical decision support.

## 19 - Performance, Scalability, And Stability

Profile CPU, GPU, memory, IO, shader compilation, streaming, hitches, AI, traffic, physics, audio, UI, save, and loading with representative captures. Create scalability tiers, PSO strategy, automated soak routes, leak checks, crash recovery, and minimum/recommended-spec evidence.

Deliverables:

1. Unreal Insights captures for vertical-slice journeys.
2. Scalability tiers and PC settings recommendations.
3. Automated soak tests and streaming/traffic/combat/pursuit routes.
4. PSO/shader strategy, hitch reports, memory budget, and crash triage process.
5. Stability report with known issues and acceptance thresholds.

Acceptance criteria:

- Performance data is measured, not guessed.
- Representative minimum-spec targets are documented.

## 20 - Content Pipeline, QA, Packaging, And Release

Add validators, naming rules, maps/checklists, automated functional tests, save compatibility matrix, localization pipeline, BuildGraph/CI, signed Windows packaging, patch/chunk strategy, crash reporting, store integration boundaries, release documentation, and a reproducible Shipping build.

Deliverables:

1. Content validation suite and asset QA checklist.
2. Functional test plan covering vertical slice, saves, missions, vehicles, police, VEX, UI, and settings.
3. Localization pipeline and text audit.
4. BuildGraph/CI packaging scripts, signing notes, patch/chunk plan, and crash reporting hooks.
5. Reproducible packaged Shipping build and release README.

Acceptance criteria:

- A packaged Windows build can be produced from documented steps.
- QA has a clear acceptance matrix and known-limitations report.
