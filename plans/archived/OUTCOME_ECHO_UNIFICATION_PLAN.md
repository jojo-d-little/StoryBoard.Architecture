# Outcome Echo Unification Plan

Status: Completed
Owner: Storyboard.Shared runtime action execution
Last updated: 2026-07-21

## Objective

Unify runtime outcome echo handling so each action execution resolves and emits outcome echo from one centralized post-action path.

## Why

1. Current runtime has two echo pathways:
- Action-specific emit in executable actions (self-handled).
- Shared post-core emit path.
2. This creates duplicate script lookups and debugger confusion.
3. Single-path emission should reduce complexity and make failures easier to trace.

## Current State Snapshot

1. Shared post-core path exists in RuntimeCommandActionExecutor.ExecuteOutcomeEcho.
2. Several action types are marked self-handled and emit inside their executable action classes.
3. RuntimeActionPayloadAccessors.GetOutcomeEchoScript may be called more than once per command because of split paths.

## Proposed Target State

1. Every action returns a structured runtime outcome.
2. Structured outcome carries:
- final result code token
- logical success
- action-scoped resolved echo variables (if any)
3. One centralized post-action method performs outcome script resolution and output emission exactly once.
4. Self-handled action list is removed after parity validation.

## Small Execution Plan

### Phase 1: Guardrail Baseline

1. Add/confirm focused tests for:
- put/remove/open/close/lock/unlock/navigate/composite success+failure echo behavior
- token-specific fallback behavior (exact result code token -> baseline token)
- no duplicate output emission
2. Capture current behavior expectations before refactor.

Exit criteria:
1. Focused tests pass on current implementation.
2. Test names clearly identify behavior that must remain unchanged.

### Phase 2: Outcome Context Shape

1. Extend runtime outcome contract to optionally carry merged action echo reference values.
2. Update executable actions to populate outcome context instead of directly emitting echo.
3. Keep current self-handled emit logic temporarily behind compatibility wiring until tests pass.

Exit criteria:
1. All affected actions can return context needed for echo interpolation.
2. No behavior regressions in focused tests.

### Phase 3: Centralize Emission

1. Move final emit decision and script evaluation to one shared post-core path.
2. Remove self-handled branch gate once all migrated actions pass parity tests.
3. Ensure GetOutcomeEchoScript is called once per action execution path.

Exit criteria:
1. Single-path output emission in runtime action executor.
2. No duplicate output lines.
3. Focused and broad regression tests pass.

## Validation

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\Storyboard.Shared.Tests\Storyboard.Shared.Tests.csproj
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameCommandProcessorFixtureTests|RuntimeActionPayloadAccessorsTests|RuntimeActionResultCodeRegistryTests"
4. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"

## Completion Notes

1. Runtime action outcomes now optionally carry an outcome invocation context for centralized echo interpolation.
2. Self-handled outcome echo gate was removed from shared executor flow.
3. Previously self-emitting actions now return result token + context and rely on one shared post-core outcome emit path.
4. Validation sequence in this plan passed after the refactor.
