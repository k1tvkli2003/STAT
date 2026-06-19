# STAT Quality Gates

A stage is incomplete until every applicable gate has evidence. Do not accept claims without compile logs, automation results, packaged smoke tests, profiling captures, debug screenshots, or explicit manual verification notes.

## Build And Playability

- Every milestone ends in a controllable build; vertical-slice milestones require a packaged Windows build, not editor-only systems.
- Editor target compiles cleanly and relevant runtime/editor/test modules load.
- Build scripts and project setup are reproducible from documented commands.
- No private credentials, local-only paths, or undocumented machine state are required.

## Vertical Slice Discipline

- No city-scale expansion before combat, driving, pursuit, mission, save, UI, accessibility, and performance targets pass in one district.
- Systems used by the slice are production-shaped, not throwaway prototypes hiding missing ownership.
- Content scale follows measured budgets.

## C++ Authority And Data Integrity

- Core rules are implemented in testable C++; Blueprints configure, compose, tune, or present.
- Primary Data Assets validate IDs, tags, references, ranges, dependencies, localization keys, and migration versions.
- Save-affecting systems use stable IDs, versioned schemas, migration tests, and corruption recovery.
- Deterministic systems use stable event ordering and seeded randomness where required.

## Robustness And Recovery

- Missing assets, invalid data, streaming transitions, corrupted saves, input changes, low settings, paused state, package differences, disconnects, and optional-mode disablement recover gracefully.
- Debug commands and visualizations exist for mission state, heat, traffic, AI, save, streaming, economy, crew, and VEX where relevant.
- Failure states are player-readable and do not soft-lock the vertical slice.

## PC Quality And Accessibility

- Keyboard/mouse, gamepad, remapping, input glyphs, ultrawide, resolution scaling, window modes, subtitles, color alternatives, aim/drive assists, reduced camera shake, hold/toggle options, and text scaling pass.
- UI is localization-ready and does not depend on hard-coded English layout sizes.
- Low settings preserve essential gameplay feedback.

## Performance And Stability

- Measured captures meet documented CPU, GPU, frame-time, memory, streaming, IO, traffic, AI, physics, audio, UI, and package-size budgets on target hardware.
- Performance-sensitive work includes stat groups, trace markers, debug views, and scalability behavior.
- Long-running soak, streaming, combat, vehicle, pursuit, and mission journeys do not leak or hitch beyond accepted thresholds.

## Security, Privacy, And Online Boundaries

- Telemetry is consent-aware, privacy-minimizing, and disableable.
- Online authority, replication trust boundaries, anti-cheat posture, and secret handling are explicit where relevant.
- VEX Protocol is isolated from campaign state and clearly labeled as educational only.

## Delivery Evidence

- Changed files and assets are listed.
- Commands, automation tests, profiles, packaged checks, and manual verifications are reported exactly.
- Known limitations and deferred work are explicit.
- The next dependency is named so the following stage can continue cleanly.
