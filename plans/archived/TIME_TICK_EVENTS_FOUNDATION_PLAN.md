# Time-Tick Events Foundation Plan

Date: 2026-08-17
Status: Closed (Runtime Foundation Complete; Deferred Follow-Ons Documented)

## Status Refresh (2026-08-24)

1. Closure remains valid; deterministic timer/tick foundation is still intact.
2. Deferred items remain intentionally deferred and are not closure regressions for this plan.
3. Downstream work is now phase-focused under the umbrella plan.

## Plan Hygiene Update (2026-08-19)

1. Runtime foundation closure remains valid; checklist/state below was normalized to match implemented code and validation evidence.
2. Tick advancement ownership is engine-managed in v1 (think-loop advances session tick each wake); host-driven pulse ownership is deferred.
3. Delta poll batch pacing refinement for Mild/Medium/Hot remains intentionally deferred as a follow-on and is not a blocker for this foundation closure.
4. Remaining non-blocking follow-ons are listed in "Remaining Deferred Items" near the end of this document.

## Purpose

Extract and execute the time-tick scheduling/event foundation from the umbrella roadmap as a focused, low-risk implementation track.

## Design Lock Snapshot (2026-08-17)

1. Core runtime primitive is a deterministic tick-deferred action queue (internally may still use scheduler naming).
2. Timer start path is `event -> subscription -> StartTimerAction -> timer entry created`.
3. Timer fire path is `tick due -> owner scope liveness check -> lookup bound action -> execute action`.
4. Timer fire does not fan out through producer-authored event subscriptions in v1.
5. Execution loop is single-threaded end-to-end for enqueue, due selection, cancellation, fire, and journaling.
6. Scope-bound lifetime validity is enforced just-in-time at fire checks for MVP.
7. Commands, events, and timers are independent think-loop activities; do not intermix or merge their processing/state payloads.
8. Think-loop ordering lock: timer processing runs after command and event processing.
9. Repeated-start conflict behavior is MVP-required with at least:
- `replaceExisting`
- `ignoreIfExists`

## Relationship To Umbrella

1. Parent plan: `plans/active/TIME_PASSAGE_AND_EVENTS_PLAN.md`.
2. This plan owns Topic 1 only: generalized time-tick-events.
3. Phase progression and milestone implementation were deferred until this plan closed; this dependency is now satisfied.

## Scope (This Plan)

1. Monotonic runtime tick model and execution boundaries.
2. Deterministic tick-deferred timer model that executes bound actions.
3. Time-oriented anchor surfaces needed by event payload mapping (including room-entry-relative anchors).
4. Anchor-origin timer semantics (what each timer is timed from) with explicit scope ownership.
5. Automatic schedule termination rules when anchor scope exits.
6. Replay determinism rules and lock decisions for scheduling behavior.
7. Focused tests and regression gates proving deterministic behavior.
8. Small subscription authoring UX enhancement to support clear intent naming when multiple subscriptions target the same event.

## Subscription Naming UX Enhancement (MVP)

1. Subscription display name defaults to subscribed `eventKey` on creation.
2. Subscription display name is user-editable after creation.
3. Multiple subscriptions at the same node may target the same `eventKey` with distinct display names.
4. Renaming a subscription does not change its subscribed `eventKey`.
5. Event dispatch behavior remains keyed by event identity, not by subscription display name.

## Out Of Scope (This Plan)

1. Phase authoring/validation policy and phase action behavior details.
2. Milestone gameplay condition implementation.
3. Full recurrence model beyond explicitly approved v1 slice.

## Starting Inputs Carried From Umbrella

1. Runtime scheduling is tick-based even when authored delay input is milliseconds.
2. Canonical conversion rule:
- `ticksFromNow = ceil(delayMs / tickIntervalMs)`
- positive non-zero delay enforces `ticksFromNow >= 1`.
3. Due work ordering target:
- `dueTick`
- lane/priority (if enabled)
- insertion sequence
- deterministic id fallback.
4. Dispatch and replay correctness are asserted at safe execution boundaries, not by poll chunk size.
5. For this v1 slice, due timers execute their bound target action directly through the existing action execution pipeline.

## Anchor-Bound Timer Contract (v1)

1. Every scheduled timer must declare an anchor origin (for example `roomEntry`, `sessionStart`, or other approved anchor roots).
2. Anchor origin also defines schedule ownership scope unless explicitly overridden by contract.
3. Room-anchored timers are owned by the current room context instance.
4. When anchor scope exits, all pending schedules owned by that scope are automatically canceled.
5. Room-entry example lock:
- `delayMs` after `roomEntry` schedules inside the entered room scope.
- if player exits that room before due tick, pending schedules are terminated and never dispatch.
6. Timer-anchor dependency rule:
- a timer anchor is event-driven; if an anchor event does not yet exist, add the event emit point before enabling that timer anchor in authoring.
7. Timer action binding rule:
- each timer entry captures one target action reference at creation time.
- timer fire resolves and executes that bound action in inherited scope context.

## MVP Anchor/Event Candidates To Evaluate

1. Geography entry/exit anchors and events for each scope node:
- `planetEntered` / `planetExited`
- `countryEntered` / `countryExited`
- `areaEntered` / `areaExited`
- `roomEntered` / `roomExited`
2. Session/player anchors:
- `sessionStart`
- `playerJoined`
3. Object state-change anchor/event (new candidate):
- event fires when a specific object's specific `gameProperty` changes.
- optional filter for "changed to target value" (for example `isOpen` becomes `true`).
- timer definitions may anchor from this event (for example X ms after object property change).
4. Scope-bound termination lock for all scope-owned anchors:
- pending schedules are canceled when the owning scope exits/deactivates.

## MVP Timer Definition Fields (Locked)

1. `timerKey` / name.
2. `scheduleAfterMs`.
3. `fireMode` (`oneShot` or `repeating`).
4. `repeatMode` for repeating timers:
- `fixedInterval`
- `growingInterval`
- `shrinkingInterval`
5. `repeatProgressionMode` for grow/shrink timers:
- `linear`
- `exponential`
6. `repeatIntervalMs` (required when `fireMode=repeating`; used as base interval).
7. `repeatIntervalStepMs` (required for `growingInterval` and `shrinkingInterval`).
8. `repeatProgressionRate` (required for `exponential` progression; tunes change intensity over time).
9. `repeatIntervalMinMs` (required for `shrinkingInterval`; interval never drops below this value unless shrink-expiry is enabled).
10. `shrinkingExpiresUnderMs` (optional for `shrinkingInterval`; if next effective interval would be below this threshold, timer expires instead of clamping/continuing).
11. `onShrinkExpiryActionRef` (optional companion action executed once when shrink-expiry occurs; supports terminal behavior such as "explode").
12. `lifetimeOwnerType` (`session`, `player`, `planet`, `country`, `area`, `room`).
13. `targetActionRef` (action executed on normal timer fire).
14. `conflictBehavior` on repeated start with required MVP values:
- `replaceExisting`
- `ignoreIfExists`
15. `enabled` flag.
16. Explicit cancel support by `timerKey` via `CancelTimerAction`.

Notes:

1. Explicit authored "anchor origin" is not required in MVP timer definitions.
2. Timer start origin is implicit from the triggering event path (`event -> subscription -> StartTimerAction`).
3. Runtime may still capture triggering event key as internal diagnostics/replay metadata.

## Topic A: Tick And Scheduler Foundation

### A1. Runtime Tick State

- [x] Add monotonic `TickCounter` runtime/session state.
- [x] Ensure `TickCounter` is persistent across wakes and does not reset per wake loop.
- [x] Keep per-wake catch-up counters non-canonical and local only.

### A2. Schedule Record Model

- [x] Define runtime schedule record fields for deterministic replay:
  - source `delayMs`
  - effective `tickIntervalMs`
  - computed `ticksFromNow`
  - `scheduledAtTick`
  - `dueTick`
  - insertion sequence
  - dispatch sequence
- [x] Persist action binding metadata (`targetActionRef` and resolution root context) for deterministic fire behavior.
- [x] Define cancellation handle/id and canceled-state semantics.
- [x] Persist schedule ownership metadata (`anchorKind`, `ownerScopeKind`, `ownerScopeId`) for deterministic termination.
- [x] Implement timer-key conflict behavior for MVP:
  - `replaceExisting`
  - `ignoreIfExists`
- [x] Defer `failIfExists` unless needed by concrete authoring use case.

### A3. Execution Model

- [x] Engine/think-loop supplies tick advance pulses in v1; runtime remains single-threaded. (Host-driven pulse ownership deferred.)
- [x] Keep command processing, event processing, and timer processing as independent activity slices in the think loop.
- [x] Do not introduce command<->event<->timer data blending/merging dependencies.
- [x] Run timer processing last in the think-loop activity order.
- [x] Execute all due timer entries for the advanced tick horizon.
- [x] Process event/schedule chain to quiescence with guardrails for runaway chains.
- [x] Apply owner-scope liveness checks before dispatching due schedules.
- [x] Use just-in-time liveness checks as the correctness rule for MVP.
- [x] Keep cancellation paths idempotent across explicit cancel and scope-exit invalidation.

### A4. Schedule Invocation Policy

- [x] Enforce timer fire to execute one bound action via existing action executor pipeline.
- [x] Keep existing event->subscription->action path as the primary timer creation/cancellation authoring route.
- [x] Ensure timer fire preserves inherited scope context for action lookup and execution.

### A5. Session-Delta Journal Integration For Timers

- [x] Preserve timer journaling as a standalone completion stream shape (no payload/data merge with command or event completion entries).
- [x] Emit timer completion entries using timer-owned semantics and reason fields (for example canceled/fire/outcome/expiry).
- [x] Keep watermark advancement and replay continuity deterministic while preserving separation from command/event completion payloads.

## Topic B: Time Anchors And Anchor-Lifetime Semantics

### B1. Anchor Surface

- [x] Define time/session anchors exposed through canonical anchor provider path.
- [x] Include room-entry-relative anchor support (for example "since room entry").
- [x] Keep anchor lifetime/reset semantics explicit and deterministic.
- [x] Define an explicit anchor catalog allowed for timer origins in v1. (MVP origin remains implicit from event subscription path.)

### B2. Lifetime And Reset Rules

- [x] Room-entry-relative values reset on room transition.
- [x] Command/action lifetime values follow existing command/action scope boundaries.
- [x] Session-level tick anchors persist with session state.
- [x] Room-owned pending schedules terminate on room exit.

### B3. Mapping Integration

- [x] Route any new time-anchor payload mappings through existing resolver pipeline only.
- [x] Maintain non-fatal mapping failure behavior and diagnostics significance policy.

### B4. Authoring And Runtime Contract Alignment

- [x] Ensure designer-facing timer definition model captures anchor origin and scope ownership. (Anchor origin remains implicit from event subscription path in MVP; explicit ownership persisted via `lifetimeOwnerType`.)
- [x] Ensure runtime scheduling contract consumes that anchor/scope metadata without hidden fallback defaults.
- [x] Add diagnostics for rejected timer definitions when anchor/scope is invalid for context. (See `PROJ-017` timer owner-scope context validation rule.)
- [x] Ensure designer-facing timer definitions capture target action reference and timer key.
- [x] Add subscription display-name editing support with default-to-event-key behavior.
- [x] Ensure save/load round-trips preserve edited subscription display names.

## Design Lock-Off Questions (Pre-Implementation)

Use this section to ratify implementation defaults before coding begins. Each item should be marked with a final decision.

1. Tick interval ownership and bounds
- Question: Is `tickIntervalMs` engine-config only, or producer-authored with validation bounds?
- Proposed default: engine-config only for MVP.
- Decision: Approved - engine-config only for MVP; fixed for session lifetime.

2. Timer key uniqueness domain
- Question: Is `timerKey` uniqueness scoped to owner scope or global to session?
- Proposed default: unique within owner scope.
- Decision: Approved - unique within owner scope.

3. Repeated-start conflict behavior
- Question: Which conflict modes are required in MVP?
- Proposed default: `replaceExisting` and `ignoreIfExists`.
- Decision: Approved - MVP includes `replaceExisting` and `ignoreIfExists`; `failIfExists` deferred.

4. Timer fire dispatch path
- Question: Should due timers publish into producer subscription fanout, or execute bound action directly?
- Proposed default: execute one bound action directly through existing action executor.
- Decision: Approved - execute one bound action directly through existing action executor.

5. Lifetime invalidation semantics
- Question: Should scope-bounded invalidation occur eagerly on scope-exit events or at fire-time liveness checks?
- Proposed default: fire-time liveness check only for MVP.
- Decision: Approved - fire-time liveness check only for MVP; eager pruning deferred.

6. Repeating timer MVP inclusion
- Question: Include repeating timers in MVP, and if yes, what shape?
- Proposed default: include repeating timers with three modes:
  - `fixedInterval`
  - `growingInterval`
  - `shrinkingInterval`
- Proposed default progression for `growingInterval` and `shrinkingInterval`:
  - `linear` progression
  - `exponential` progression (with `repeatProgressionRate` tuning)
- Proposed default for shrinking lower bound:
  - if `shrinkingExpiresUnderMs` is configured and next interval would be below threshold, expire timer and run `onShrinkExpiryActionRef` (if provided).
  - otherwise clamp effective interval at `repeatIntervalMinMs` and continue repeating.
- Decision: Approved - include fixed/growing/shrinking; support shrink-expiry threshold with optional terminal action; otherwise clamp at repeatIntervalMinMs.

7. Owner scope defaulting
- Question: If `StartTimerAction` does not explicitly override ownership, what is the default owner scope?
- Proposed default: inherit current subscription/action execution scope context.
- Decision: Approved - inherit owner scope from the action execution context that starts the timer when explicit ownership is not set.

8. Explicit cancel targeting
- Question: Should `CancelTimerAction` target by `timerKey` only, or `timerKey` plus optional scope qualifier?
- Proposed default: `timerKey` with optional scope qualifier for precision.
- Decision: Approved - when qualifier is omitted, cancel any scheduled instance matching `timerKey`/timer name across active owned scopes; when qualifier is provided, cancel only matching instances within that qualifier.

9. Required timer lifecycle journaling
- Question: Which lifecycle transitions are contract-required for host delta/replay continuity?
- Proposed default: timer lifecycle journaling is standalone in the think loop (no merge/intermix dependency with command/event completion payloads); timer processing occurs last, and timer entries use timer-owned reason fields for continuity.
- Decision: Approved - keep command/event/timer journaling independent; timer processing and timer completion entries are last in loop order.

10. Subscription naming UX contract
- Question: Should subscription display name be independent from event key while preserving event binding?
- Proposed default: default display name to `eventKey`, allow rename/edit, persist round-trip, keep dispatch keyed by event identity.
- Decision: Approved - default display name to `eventKey`, allow rename/edit, persist round-trip, keep dispatch keyed by event identity.

## Implementation Slice 1 (Lock Translation)

Goal: Translate approved lock decisions into the first runnable runtime slice before broader UX and recurrence expansion.

### S1-A. Runtime Timer Contracts And State

- [x] Add timer definition contract fields for locked MVP shape:
  - `timerKey`, `scheduleAfterMs`, `fireMode`, `repeatMode`
  - `repeatIntervalMs`, `repeatIntervalStepMs`, `repeatIntervalMinMs`
  - `shrinkingExpiresUnderMs`, `onShrinkExpiryActionRef`
  - `lifetimeOwnerType`, `targetActionRef`, `conflictBehavior`, `enabled`
- [x] Add runtime scheduled-timer state record:
  - immutable schedule identity
  - owner scope identity (`ownerScopeKind`, `ownerScopeId`)
  - due-tick and insertion sequence data
  - bound action reference + execution context root
- [x] Enforce `timerKey` uniqueness within owner scope.
- [x] Implement conflict modes:
  - `replaceExisting`
  - `ignoreIfExists`

### S1-B. Timer Actions

- [x] Implement `StartTimerAction` executor:
  - resolve timer definition by key
  - apply enabled/validation checks
  - capture action execution context as default owner scope when explicit override is absent
  - enqueue scheduled timer entry
- [x] Implement `CancelTimerAction` executor:
  - cancel by `timerKey`/timer name
  - when scope qualifier omitted, cancel all matching active instances
  - when qualifier provided, cancel only matching scoped instances

### S1-C. Think Loop Ordering And Isolation

- [x] Enforce single-thread think-loop processing for command/event/timer activity slices.
- [x] Keep command, event, and timer paths independent (no payload/state blending between slices).
- [x] Lock activity order with timer processing last.
- [x] Apply fire-time owner-scope liveness check before timer dispatch.

### S1-D. Timer Fire And Recurrence

- [x] On due tick, execute one bound `targetActionRef` through existing action executor.
- [x] Preserve inherited/bound execution context for action lookup.
- [x] Implement recurrence behaviors:
  - `fixedInterval`
  - `growingInterval`
  - `shrinkingInterval`
- [x] Implement shrink-expiry behavior:
  - if next interval drops below `shrinkingExpiresUnderMs`, expire timer
  - run `onShrinkExpiryActionRef` once when configured
  - otherwise clamp to `repeatIntervalMinMs`

### S1-E. Session-Delta Journal Integration

- [x] Emit timer completion entries as standalone timer-owned journal records.
- [x] Keep command/event/timer journal streams independent while preserving deterministic watermark continuity.
- [x] Mark timer completion source explicitly as `timerCompletion`.
- [x] Include timer reason detail fields (cancel, fire, outcome, expiry) without introducing cross-stream merge behavior.

### S1-F. Slice 1 Validation

- [x] Add focused unit tests for:
  - uniqueness + conflict behavior (covered in `RuntimeSessionTimerQueueTests` and `RuntimeTimerActionTests`)
  - owner-scope default capture from action execution context (covered in `RuntimeTimerActionTests`)
  - fire-time liveness invalidation (covered in `RuntimeTimerActionTests`)
  - timer-last loop ordering (added in `GameThinkLoopTests`)
  - recurrence (fixed/growing/shrinking) and shrink-expiry terminal action (covered in `RuntimeTimerActionTests`)
  - cancel semantics with/without qualifier
  - timer completion journal entry shape and watermark continuity (covered in `SessionDeltaPollingContractBehaviorTests`)
- [x] Run baseline and focused validation gates after Slice 1 implementation.

## Determinism Locks (Sign-Off Required)

Locked direction (implementation start):

1. Single-thread execution is non-negotiable for v1 deterministic behavior.
2. Fire-time owner-scope liveness check is canonical correctness gate for bounded lifetimes.
3. Timer fire executes bound action directly (not producer-level subscription fanout on a generic timer-fired key).

1. Tick interval ownership:
- engine-config only vs producer-authored.
- if producer-authored, min/max bounds and change timing rules.
- Decision: Approved - keep tick interval engine-config owned for v1 (`RuntimeEngineTuningConfig.EngineLoopTickIntervalMs`), producer-authored tick sizing deferred.

2. Same-due-tick ordering:
- confirm final precedence chain.
- confirm whether lane/priority is v1-required or deferred.
- Decision: Approved - deterministic precedence is `(dueTick ASC, insertionSequence ASC, scheduleId ASC)` as implemented in `RuntimeSessionTimerQueue.TryDequeueDue`; lane/priority deferred.

3. Replay assertion mode policy:
- `strict` for exact dispatch/boundary order.
- `semantic` for stable boundary state/output/watermark continuity with chunk tolerance.
- choose default CI mode and explicit strict-required cases.
- Decision: Approved - CI default is `strict`; `semantic` is opt-in per playback case using `strict:false` in `playback-test-cases.json` and only enforces output-line continuity.

## Validation Gates

Baseline gates:

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`

Focused additions for this plan:

1. Scheduler conversion and due-tick ordering tests.
2. Tick persistence across wakes and watermark continuity tests.
3. Room-entry-relative anchor reset and payload mapping tests (covered in `RuntimeScopeLifetimeTransitionTests` and `RuntimeEventPayloadBuilderTests`).
4. Scope-exit cancellation tests (room-exit kills room-owned pending timers; covered by OwnerNotAlive timer completion in `SessionDeltaPollingContractBehaviorTests`).
5. Direct timer-fire action execution and inherited-context lookup tests.
6. Timer lifecycle journaling and watermark-continuity tests.
7. Repeated-start conflict behavior tests (`replaceExisting`, `ignoreIfExists`).
8. Replay strict/semantic assertion-mode behavior tests.
9. Subscription naming UX tests (same event key, multiple subscriptions, distinct edited names, save/load persistence).

Validation evidence recorded 2026-08-18:

1. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"` passed (9/9).
2. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"` passed (65/65).

Validation evidence refresh 2026-08-19:

1. `dotnet build .\\StoryboardDesigner.slnx` passed after timer milestone/core-flow and delta retention-path triage updates.
2. Focused runtime/timer/delta validation remained green during post-triage rebuild loop (see latest session build/test logs).

## Remaining Deferred Items

1. Delta polling profile pacing refinement (Mild/Medium/Hot watermark-advancement budgeting) remains deferred by design and tracked in runtime code TODO:
- `Storyboard.GameEngine/GameManager/GameManager.cs` TODO marker: `TODO(events-foundation, high-priority)`.
2. Host-driven tick pulse ownership remains deferred; v1 canonical behavior is engine-managed think-loop tick advancement.
3. Future semantic expansion from room-centric to broader scope-centric pruning (for example area-centric) is intentionally out of this closed foundation plan.

## Exit Criteria

1. Determinism locks are resolved and recorded in this plan.
2. Tick and schedule foundation behaviors are implemented and tested.
3. Time-anchor semantics (including since-room-entry behavior) are implemented and tested.
4. Scope-owned timer termination behavior is implemented and tested.
5. Direct timer-fire action execution behavior is implemented and tested.
6. Timer lifecycle journaling/delta continuity behavior is implemented and tested.
7. Focused regression and playback smoke gates pass.
8. Umbrella plan can move to Phase/Milestone implementation track with this foundation treated as closed.
