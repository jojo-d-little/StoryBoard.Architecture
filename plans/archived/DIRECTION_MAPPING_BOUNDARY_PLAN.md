# Direction Mapping Boundary Plan

## Goal
Define a strict boundary where the engine owns canonical directions (10 total) and producers fully own command-word mappings to those directions.

## Scope
- In scope: command preprocessing, runtime export contract, runtime bootstrap mapping, mutation gateway token handling, diagnostics, tests.
- Out of scope: changing canonical direction enums or traversal physics.

## Small Plan
1. Add/verify runtime export support for global directional traversal mappings so producer mappings are persisted into runtime input.
2. Change preprocessor direction resolution to prefer producer mapping and remove hardcoded word fallback in strict mode.
3. Limit mutation gateway direction parsing to canonical tokens only (`N,NE,E,SE,S,SW,W,NW,UP,DOWN`) and fail non-canonical tokens with clear diagnostics.
4. Add compatibility switch (temporary) to stage rollout and emit warnings when legacy hardcoded aliases are used.
5. Add focused tests:
   - mapped token works (e.g. right -> East when mapped)
   - unmapped directional word does not resolve direction
   - move/rotate/navigate consume canonical direction output from preprocessor
   - runtime export contains producer global mappings

## Exit Criteria
- Directional command-word behavior is controlled by producer mappings only.
- Core runtime still owns the canonical 10-direction model.
- Focused runtime + command parsing tests pass.

## Closure
- Status: Closed
- Closed on: 2026-09-07
- Notes:
   - Producer-owned traversal mappings now flow through export and bootstrap for global and scoped nodes.
   - Command preprocessing no longer uses legacy hardcoded directional-word fallback for player input resolution.
   - Designer mapping authoring supports full Direction10 canonical targets including Up and Down.
   - Bootstrap legacy direction alias parsing shim was removed to enforce hard-cut behavior.
- Validation:
   - dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj
   - dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
   - dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
