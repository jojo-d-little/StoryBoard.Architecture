# Shared Variable Reference Aliasing Plan

Status: Implemented
Owner: Storyboard.Shared runtime/state
Last updated: 2026-07-16

## Objective

Move shared variables from synchronization-based propagation to true reference aliasing, where all participants in a shared group read and write through the same backing reference and receive deterministic mutation notifications.

## In Scope

1. Runtime-state data structures in Storyboard.Shared.
2. Shared-variable initialization, read/write behavior, and mutation signaling.
3. Regression tests for alias identity, propagation behavior, and mutation revision outcomes.

## Out of Scope

1. Designer shared-variable JSON shape/schema changes.
2. Simulator/designer UX changes.
3. Non-shared variable semantic changes.

## Open Design Questions (Committed)

None.

## Locked Decisions

1. Naming: the canonical runtime value-holder reference type name is RuntimeValueRef.
2. Responsibility boundary: RuntimeValueRef is a minimal reference holder for current value plus mutation notification only.
3. Non-responsibilities: RuntimeValueRef does not enforce restrictions, validate values, or own property-level policy decisions.
4. Q-01 locked: RuntimeValueRef emits an internal low-level mutation signal that is funneled through GamePropertyState's existing mutation callback chain; no new public mutation payload contract is introduced.
5. Q-02 locked: RuntimeValueRef is subscribed directly by GamePropertyRuntimeValue; GamePropertyRuntimeValue exposes its own mutation event and GamePropertyState subscribes there to forward into its existing mutation callback chain.
6. Q-03 locked: RuntimeValueRef suppresses no-op writes (same value) to prevent churn; no mutation event is raised and no mutation revision increment should occur on no-op updates.
7. Q-04 locked: Shared reference initialization uses deterministic first-discovered participant precedence as winner; non-winning conflicting defaults are coerced to winner value and each conflict emits a diagnostic, without failing initialization.
8. Q-05 locked: Restriction mismatches are treated as runtime warnings only; runtime continues and uses least restrictive effective behavior for shared value writes.
9. Q-06 locked: Restriction enforcement remains canonical in GamePropertyRuntimeValue; GamePropertyState does not enforce restrictions and RuntimeValueRef remains restriction-agnostic.
10. Q-07 locked: Runtime share membership topology is static for the current feature set (designer-defined at design time and materialized at session initialization). Runtime add/remove/recreate participant lifecycle semantics are out of scope for this implementation.
11. Q-08 locked: Shared binding is eager at session initialization only, with no lazy/on-access rebinding path in the current implementation scope.
12. Q-09 locked: Runtime assumes single-threaded command execution; RuntimeValueRef mutation and event dispatch are not internally synchronized in current scope, and concurrency handling is deferred to a future plan if needed.
13. Q-10 locked: RuntimeValueRef is runtime-internal only and does not alter export/contract shapes; export continues emitting resolved values with no RuntimeValueRef identity or structure exposure.
14. Q-11 locked: GamePropertyRuntimeValue surface APIs remain unchanged (Name, Value, Lifetime, ValueRestriction, SharedVariableId, and TrySetValue signature/behavior), and RuntimeValueRef awareness is isolated to GamePropertyRuntimeValue internals only.
15. Q-12 locked: Signoff validation matrix is mandatory and includes solution build, full Storyboard.Shared.Tests run, focused StoryboardDesigner.App.Tests runtime guardrail filter run, replay regression smoke gate run, and targeted alias-identity/no-op-churn shared tests.
16. Q-13 locked: Diagnostics are non-fatal warnings for shared-default mismatch and restriction mismatch at initialization; write-rejection warnings are emitted only when an effective restriction actually rejects a candidate value.

## Exit Criteria

1. All committed design questions are answered and recorded in this plan.
2. A migration sequence is defined with low-risk implementation slices.
3. Required regression tests are identified before implementation starts.
4. Contract stability constraints are explicitly documented and accepted.

## Implementation Status

1. All design lock-off questions are resolved.
2. RuntimeValueRef aliasing implementation is complete.
3. Shared initialization, least-restrictive effective restriction behavior, and diagnostics are implemented.
4. Regression coverage for aliasing and no-op suppression is implemented.

## Validation Signoff (2026-07-16)

1. dotnet test .\Storyboard.Shared.Tests\Storyboard.Shared.Tests.csproj: passed.
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests": passed.
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep": passed.
4. dotnet build .\StoryboardDesigner.slnx: passed.

## Phased Implementation Plan

### Phase 1 - Add RuntimeValueRef Primitive

1. Add RuntimeValueRef as a minimal reference holder with value storage, no-op suppression, and internal mutation event.
2. Keep restriction and policy logic out of RuntimeValueRef.
3. Add focused unit tests for value update and no-op suppression behavior.
Status: Implemented

### Phase 2 - Refactor GamePropertyRuntimeValue Internals

1. Keep GamePropertyRuntimeValue public API unchanged.
2. Switch Value backing from direct field to RuntimeValueRef.
3. Subscribe in GamePropertyRuntimeValue to RuntimeValueRef mutation event and forward via GamePropertyRuntimeValue mutation event.
4. Preserve TrySetValue restriction behavior and return semantics.
Status: Implemented

### Phase 3 - Wire GamePropertyState Forwarding Chain

1. Subscribe GamePropertyState to GamePropertyRuntimeValue mutation events.
2. Funnel received mutations through existing _onMutated callback only.
3. Ensure subscription lifecycle is safe for variable creation and removal.
Status: Implemented

### Phase 4 - Replace Shared Sync with Shared Reference Aliasing

1. Build shared RuntimeValueRef instances per shared id during session initialization.
2. Bind all participant GamePropertyRuntimeValue instances to the same shared RuntimeValueRef.
3. Apply deterministic winner/default initialization and least-restrictive mismatch policy with warnings.
4. Keep export and contract behavior unchanged.
Status: Implemented

### Phase 5 - Diagnostics, Regression, and Signoff

1. Add and verify diagnostics for default mismatch, restriction mismatch, and write rejection when effective restriction rejects.
2. Add alias-identity and no-op-churn regression tests.
3. Run mandatory validation matrix and capture signoff results.
Status: Implemented
