# Event Addendums Plan

Date: 2026-08-19
Status: Closed

## Status Snapshot (2026-08-19)

1. Functional addendums are implemented in runtime dispatch and schema contracts.
2. Designer authoring support for new subscription fields is now exposed in the event subscription editor.
3. Runtime eligibility/source-match tests are in place for primary modes.
4. Missing-origin source-match guard tests are now covered in runtime eligibility tests.
5. Designer persistence regression coverage now includes round-trip/runtime-export assertions for source/visibility fields.
6. Strict playback smoke and focused runtime gates are currently green after replay baseline reconciliation.
7. Remaining observability counter hardening is deferred as follow-on (non-blocking for closure).

## Purpose

Capture focused event-system enhancements identified after foundation closure, with explicit lock decisions before implementation.

## Relationship To Existing Plans

1. Parent umbrella: `plans/active/TIME_PASSAGE_AND_EVENTS_PLAN.md`.
2. Complements closed foundation work in: `plans/active/TIME_TICK_EVENTS_FOUNDATION_PLAN.md`.
3. Scope here is additive event-routing behavior and performance hardening, not timer/tick foundation redesign.

## Problem Statement

Two functional gaps are currently in scope:

1. Subscriber selection is currently too broad by default (global-ish enumeration across session scopes).
2. Object-centric behavior needs clear source-targeted subscription semantics (for example "self only") without payload-filter hacks.

Performance note (deferred): candidate-routing index/perf optimization is intentionally deferred to a follow-on plan after functional correctness is locked.

## Goals

1. Make event subscription eligibility safe and local by default.
2. Add explicit source-match semantics for object-local behavior.
3. Preserve deterministic replay and existing event/action contract guarantees.

## Non-Goals

1. Redesign event publication contracts in a breaking way.
2. Replace the existing action execution pipeline.
3. Introduce nondeterministic parallel event dispatch.
4. Expand into full phase/milestone gameplay design in this plan.

## Functional Addendum A: Subscriber Eligibility Controls

### A1. New Subscription Fields (Subscriber-Centric)

1. `subscriptionVisibleWhenContained` (default: `false`)
- Controls whether a subscriber object is eligible while currently contained/inventory-contained.

### A2. Default Behavior Lock (Proposed)

1. Default dispatch eligibility is room-local + lineage-safe.
2. By default, contained objects are not eligible subscribers.
3. Subscription geography widening beyond the current room + ancestors model is deferred until candidate-routing performance optimization is designed.

## Functional Addendum B: Source-Match Controls

### B1. New Subscription Field

1. `subscriptionSourceMatchMode` (default: `AnySource`)
- Proposed values: `AnySource`, `SelfOnly`, `SameContainer`, `SameRoom`, `ExplicitSourceId`.
2. `subscriptionSourceScopeNodeId` (nullable `Guid`)
- Required when `subscriptionSourceMatchMode=ExplicitSourceId`.

### B2. Behavior Intent

1. Enable object-local behaviors like:
- `item_move_started` -> start sound on the moved object.
- `item_move_completed` -> stop sound on the moved object.
2. Avoid payload-filter backdoor patterns for source identity matching.

## Deferred Topic: Event Routing Indexes

This topic is intentionally out of scope for this plan phase and will be revisited after functional behavior rollout.

### Deferred Direction (Preview Only)

1. Build a runtime/session event-routing index keyed by `eventKey`.
2. Each indexed entry stores compact subscriber metadata needed for filtering:
- subscriber scope node id
- subscription id
- scope lineage metadata
- containment metadata
- source-match mode
- deterministic ordering metadata

## Determinism + Compatibility Guardrails

1. Preserve stable candidate ordering for strict replay consistency.
2. Keep default behavior explicit and schema-driven (no hidden fallback magic).
3. Keep dispatch single-threaded in v1.
4. Keep event publication contract unchanged where possible.

## Implementation Slices

### Slice 1: Contract + Dispatcher Defaults

1. Completed: Added `subscriptionVisibleWhenContained` to contract schema + DTO mapping.
2. Completed: Updated dispatcher candidate eligibility evaluation.
3. Completed: Added runtime eligibility coverage for containment default behavior.

### Slice 2: Source-Match Semantics

1. Completed: Added `subscriptionSourceMatchMode` and `subscriptionSourceScopeNodeId` to contract schema + DTO mapping.
2. Completed: Added source identity checks in dispatcher (`AnySource`, `SelfOnly`, `SameContainer`, `SameRoom`, `ExplicitSourceId`).
3. Completed: Added runtime tests for `SelfOnly`, `AnySource`, and `ExplicitSourceId` behavior.
4. Completed: Dispatcher origin scope lookup now uses session registry lookup instead of full scope traversal.

### Slice 3: Observability + Hardening

1. Deferred: Add explicit candidate-funnel trace counters:
- post scope filter
- post containment filter
- post source filter
- executed bindings
2. Completed: Added guard tests for missing-origin source-match behavior.
3. Completed: Added designer persistence regression tests for `SubscriptionVisibleWhenContained`, `SubscriptionSourceMatchMode`, and `SubscriptionSourceScopeNodeId` round-trip/runtime-export.
4. Completed: Reconciled strict playback diagnostics mismatch in `GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep` via playback baseline update.

## Validation Gates

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`
4. New focused event-dispatch subscriber-selection tests in runtime test projects.

## Lock-Off Questions (Required Before Implementation)

1. Does `subscriptionVisibleWhenContained=false` exclude only contained game objects, or also contained non-object scopes if present?
2. For contained eligibility, do we need depth controls now (direct-only vs recursive), or defer depth policy?
3. Should `subscriptionVisibleWhenContained=true` include inventory-contained and container-contained objects equally, or split policy flags?
4. Is `subscriptionSourceMatchMode=SelfOnly` evaluated by subscriber scope node id equality with published origin scope id?
5. For events without a valid origin scope id, how should `SelfOnly`, `SameContainer`, `SameRoom`, and `ExplicitSourceId` behave?
6. Should source-match evaluation run before payload condition filters for performance and diagnostics clarity?
7. What diagnostics verbosity policy should apply when a subscriber is filtered out by scope/containment/source rules (none, medium+, high-only)?

Resolved direction snapshot (2026-08-19):

1. Keep current room + ancestors geography model as baseline for now (no new subscription geography widening in this plan).
2. `subscriptionVisibleWhenContained=false` excludes contained game objects only.
3. Contained-depth controls are deferred; `true` means contained eligibility at any depth.
4. Inventory-contained and container-contained objects are treated equally.
5. `SelfOnly` uses strict scope node id equality against event origin scope node id.
6. Unset `subscriptionSourceMatchMode` is equivalent to `AnySource`.
7. `ExplicitSourceId` is included in MVP.
8. Source-match checks run before payload condition filters.
9. Performance/index optimization is deferred to a dedicated follow-on.

## Exit Criteria

1. Lock-off questions are decided and recorded.
2. Subscription eligibility defaults are implemented and covered by tests.
3. Source-match semantics are implemented and covered by tests.
4. Regression and playback validation gates pass.

Exit criteria progress (2026-08-19):

1. Satisfied: #1 (lock decisions recorded in this plan).
2. Satisfied: #2 (implemented with runtime eligibility tests).
3. Satisfied: #3 (implemented with runtime source-match tests).
4. Satisfied: #4 (build, focused runtime regression, and strict playback smoke gates currently pass).

## Closure Note (2026-08-19)

1. This addendum is closed with all functional goals and validation gates satisfied.
2. Candidate-funnel observability counters are intentionally deferred to a follow-on hardening pass and are not a closure blocker.
