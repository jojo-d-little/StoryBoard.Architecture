# Player Fallback and Synonym Diagnostics Plan

Status: Complete
Owner: Storyboard.Shared command processing and action execution
Last updated: 2026-07-20

## Scope

1. Add explicit diagnostics to distinguish active-player-container fallback failure reasons.
2. Add dedicated `PutObjectInContainer` result code for capacity failure.
3. Add focused fixture coverage for `get key` synonym resolution + player-container fallback diagnostics.

## Completed Work

1. Added detailed player fallback diagnostics in put-container resolution.
2. Added `CapacityExceeded` result code and token mapping for `PutObjectInContainer`.
3. Added `get key` fixture test asserting synonym and player fallback diagnostics.
4. Updated runtime result-code registry test expectations.
5. Validated via focused and runtime-focused test runs.

## Validation

1. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameCommandProcessorFixtureTests.PutObjectInContainer_GetKey_UsesSynonymAndFallsBackToActivePlayerContainer|FullyQualifiedName~RuntimeActionResultCodeRegistryTests.PutObjectInContainer_ResultCodeSet_UsesTypedEnumAndExpectedTokens"`
2. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`
