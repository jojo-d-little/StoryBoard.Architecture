# RoomLink Course Correction Plan

## Goal
Clarify and formalize traversal representation boundaries so designer authoring remains connection-centric while runtime execution remains directional-leg-centric, with first-class runtime game property participation on traversal legs.

## Why
- Current state is mixed: designer is effectively TraversalConnections-first, runtime execution is Links-first.
- Runtime Links now carry traversal leg passability through canonical `gameProperties` (`isPassable`) with shared-variable semantics on the property itself.
- This limits future extensibility and scriptability for traversal properties.

## Target Architecture
1. Designer contract shape is canonical on `TraversalConnections` and does not rely on `Links` as an authored source.
2. Runtime contract shape is canonical on `Links` and does not rely on `TraversalConnections` for execution.
3. Runtime traversal leg state uses proper `gameProperties` on `RuntimeRoomLinkDto`.
4. `isPassable` is treated as a normal runtime game property participant (including shared-variable semantics), not only as a special-case lookup field.

## Scope
- In scope:
  - Schema/contract boundary lock-off for traversal representation ownership.
  - Runtime `RuntimeRoomLinkDto` contract evolution toward first-class game properties.
  - Export mapping updates from designer `TraversalConnections` to runtime `Links` + leg game properties.
  - Runtime bootstrap/mapping updates to consume link game properties as execution truth.
  - Compatibility strategy for old exports and fallback behavior.
  - Simulator/runtime-tree visibility expectations for traversal leg properties.
- Out of scope:
  - New traversal gameplay features unrelated to representation correction.
  - Command grammar or parser vocabulary changes.

## Current State Summary
1. Designer stores and edits `TraversalConnections` and related leg state/variables.
2. Runtime export writes both `TraversalConnections` and `Links` on runtime area payloads.
3. Runtime execution currently consumes `Links` and ignores runtime `TraversalConnections` for traversal decisions.
4. Runtime passability resolves from canonical link `gameProperties` (`isPassable`) and associated shared-variable binding when present.

## Directional Principles
1. Keep the runtime host efficient: directional links are execution-native.
2. Keep designer authoring expressive: bidirectional connection modeling stays in designer.
3. Preserve shared-variable semantics through standard runtime game property mechanisms.
4. Prefer additive compatibility-first migration, then remove transitional aliases.

## Prerequisite Coverage Gate (Before Lock-Off Questions)
1. Add a playback regression test in the playback test suite that replays:
  - `C:\work\HobbyStuff\storyboarding\Samples\WorkshopTutorial\Recordings\BlockedDoors.sbe.sim.json`
2. Validate that the recording proves a non-passable traversal blocks movement as expected.
3. Treat this playback test as a hard prerequisite for this plan: keep it green before and during all RoomLink course-correction changes.

## Design Lock-Off Questions (Commit Gate)
Lock-off question count: 16

1. Should runtime contracts fully deprecate `RuntimeTraversalConnectionDto` from `RuntimeAreaDto` once link game properties are in place, or keep it as optional non-executed metadata?
2. Should designer contracts fully deprecate area `Links` from authored persistence once `TraversalConnections` is confirmed canonical, or keep an empty compatibility placeholder?
3. Should `RuntimeRoomLinkDto` add `gameProperties` as required, optional, or optional-with-default-empty semantics?
4. Should `isPassable` be required in runtime link `gameProperties`, or can runtime synthesize it when missing for compatibility?
5. Should top-level `RuntimeRoomLinkDto.SharedVariableId` and `RuntimeRoomLinkDto.DefaultIsPassable` remain as compatibility aliases during migration?
6. If aliases remain temporarily, what precedence is canonical at runtime when both alias fields and `gameProperties.isPassable` are present?
7. Should `OpenStatePolicy`/`OpenStateBindingMode` remain top-level link fields, or move into game properties/metadata with alias support?
8. What is the compatibility rule for old exports that have no link `gameProperties` but do have `SharedVariableId` and `DefaultIsPassable`?
9. What is the compatibility rule for exports that have `TraversalConnections` only and missing/empty `Links`?
10. Should runtime bootstrap materialize traversal leg game properties into room-adjacent runtime state nodes for simulator tree visibility?
11. If traversal leg state is room-adjacent in memory, what is the canonical identity key for a leg (fromRoomId + direction, or deterministic leg id)?
12. For `OpenStateBindingMode.Together`, should effective passability continue to require both paired legs true even when both legs share the same shared variable id?
13. What script/procedure access contract is targeted for traversal properties (for example `room.Traversal.East.isPassable`) and what canonical resolver path should back it?
14. What diagnostics are required when link `gameProperties` are malformed or `isPassable` has invalid restriction/default?
15. Which sample/runtime artifact regeneration set is mandatory in the same merge when contract shape changes?
16. Which validation gates are mandatory before merge for this course correction slice?

## Lock-Off Decisions (Agreed)
1. Runtime contract authority excludes RuntimeTraversalConnectionDto for execution; keep only short-lived compatibility metadata and remove at end-state.
2. Designer authored persistence is TraversalConnections-only for traversal authoring; Links are runtime-derived and not authored.
3. RuntimeRoomLinkDto.gameProperties enters with migration support, but samples are migrated in place and hard-cut enforcement follows; missing gameProperties becomes a load warning during transition and an error after hard cut.
4. isPassable is required in canonical runtime link gameProperties; legacy synthesis is load-time compatibility only with emitted warning.
5. SharedVariableId and DefaultIsPassable are temporary compatibility aliases only, never canonical write targets, and are removed entirely at end-state.
6. When canonical gameProperties and legacy aliases are both present, gameProperties wins; emit a load warning for duplicate/conflicting passability sources.
7. OpenStatePolicy and OpenStateBindingMode remain top-level runtime link semantics, not gameProperties.
8. For legacy exports with alias passability fields and no link gameProperties, loader can normalize during transition with warnings; after in-place sample migration and hard cut, missing canonical gameProperties is a load error.
9. Runtime does not synthesize Links from TraversalConnections at load. If Links are missing or empty, load fails with migration-required diagnostics. Any synthesis is migration-tool-only, not runtime fallback.
10. Each traversal side is represented by its own RoomLink and its own isPassable gameProperty; lock-together behavior is represented by shared binding to the same backing value, independent behavior by separate bindings.
11. Canonical runtime traversal-leg identity is fromRoomId plus direction; deterministic leg id may exist only as non-authoritative diagnostics metadata.
12. OpenStateBindingMode is honored during setup of value-sharing topology. After setup, runtime move checks read attempted-side link isPassable directly; no extra paired-leg conjunction logic is required.
13. Traversal property access shape is staged by this plan (room-adjacent, direction-addressable, RoomLink-backed), but full script/procedure feature completeness is out of scope for this slice.
14. Canonical shape violations are hard load errors; transition-only compatibility behavior emits warnings. After hard cut, legacy alias-dependent passability data is load-fail.
15. Any traversal/RoomLink contract-shape merge must include runtime contract regeneration, lock/DTO refresh, in-place sample migration, dependent artifact refresh, targeted snapshot updates, and the BlockedDoors playback gate.
16. Mandatory merge gates: solution build, shared tests, app tests, focused runtime-boundary test filter, blocked traversal playback gate, and no legacy alias usage in migrated samples after hard cut.

## Proposed Execution Slices
1. Lock-Off Slice
- Completed: all 16 design questions are resolved and recorded in this plan.
- Enforce these decisions as commit gate constraints for implementation slices.

2. Contract Slice
- Update runtime room-link schema to include first-class link game properties.
- Decide and implement alias compatibility fields policy.
- Update designer/runtime area schema ownership split as approved.

3. Export Mapping Slice
- Map designer traversal leg variables to runtime link game properties.
- Keep deterministic ordering and stable serialization.

4. Runtime Consumption Slice
- Update runtime traversal evaluation to read passability from link game property state first.
- Keep fallback compatibility path only as long as approved.

5. Runtime State Visibility Slice
- Materialize traversal leg property state in runtime/session structures suitable for simulator tree display.

6. Compatibility Cleanup Slice
- Remove transitional aliases and deprecated duplicated fields after migration criteria are met.

## Validation Plan
1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\Storyboard.Shared.Tests\Storyboard.Shared.Tests.csproj`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`
4. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`
5. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`

## Exit Criteria
1. Designer/runtime traversal ownership split is explicitly locked and implemented.
2. Runtime link passability/shared behavior flows through first-class game properties.
3. Compatibility behavior is intentional, tested, and documented.
4. Simulator/runtime tree can expose traversal leg property state with stable identity.
5. Build + required test gates are green.

## Status Update (2026-08-07)
1. Compatibility Cleanup Slice is complete for RoomLink passability aliases.
2. RuntimeRoomLink contract lock is complete after schema removal of legacy `defaultIsPassable` and top-level `sharedVariableId` aliases.
3. Staged and accepted RuntimeRoomLink DTO now retain canonical `gameProperties` and remove deprecated alias members.
4. Post-lock validation passed:
  - `dotnet build .\StoryboardDesigner.slnx`
  - `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "JsonExportServiceRuntimeExportTests|SampleRuntimeLoadMatrixTests|GameSimulatorPlaybackRegressionTests"`
  - `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests|SampleRuntimeLoadMatrixTests"`
  - `dotnet test .\StoryboardDesigner.slnx`
5. Full solution test count after lock: 1156 passed, 0 failed.