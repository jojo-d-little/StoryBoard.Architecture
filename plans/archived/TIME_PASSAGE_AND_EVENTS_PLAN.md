# Time Passage and Events Umbrella Plan

Date: 2026-08-15
Status: Ready For Archive

## Status Refresh (2026-08-24)

1. Event foundation remains closed.
2. Time-tick events foundation remains closed.
3. Phase foundation closeout baseline is complete and archive-ready.
4. Remaining umbrella items are optional post-archive hardening only (see child plan notes).
5. Deprecated phase ambience action-type cleanup completed: removed `SetupPhaseAmbienceEffectTimers` and `PlayPhaseAmbienceEffect` to align with direct narrative ambient action flow.
6. Closeout validation rerun completed this pass:
- focused runtime-boundary gate passed (70/70)
- replay smoke gate passed (9/9)

## Purpose

Coordinate the full progression roadmap across phases, event foundation, tick scheduling, and milestone integration while keeping implementation staged and low-risk.

## Current Focus

1. Active implementation and lock decisions for event foundation are tracked in:
- `plans/active/EVENTS_FOUNDATION_PLAN.md`
2. Event foundation is closed.
3. Active implementation and lock decisions for time-tick events are now tracked in:
- `plans/active/TIME_TICK_EVENTS_FOUNDATION_PLAN.md`
4. Time-tick foundation is closed (runtime foundation complete; deferred follow-ons documented in the child plan).
5. Phase progression and milestone implementation are now eligible to proceed under this umbrella sequence.
6. This umbrella plan remains the parent tracker for sequence/dependencies across the next implementation stage.

## Core Direction

1. Game progression is organized into ordered phases.
2. Events provide a generic trigger mechanism for invoking behavior.
3. Actions remain the primary behavior unit.
4. Tick time and schedules are introduced after events and phase control exist.
5. Milestones are represented as event-producing gameplay facts and can drive phase changes.

## Phase Model (v1)

1. Add a simple integer `PhaseId` in runtime state.
2. `PhaseId` starts at `0`.
3. Phase advancement increments by `+1`.
4. A set operation can jump to any valid phase id (forward or backward).
5. Phase definition supports:
- Required: `PhaseId` (sequence + identifier)
- Optional: `Name`

## Action Additions (v1)

1. `AdvancePhaseAction` increments current phase by one.
2. `SetPhaseAction` sets phase to explicit integer target.
3. Keep both actions callable from command execution and event execution.

## Tick Time Foundation (v1)

1. Introduce monotonic `TickCounter` in runtime state.
2. Engine/think-loop advances tick pulses in v1; runtime remains single-threaded and deterministic. Host-driven pulse ownership is deferred.
3. Add minimal scheduler with:
- Schedule action/event at tick offset.
- Cancel scheduled work by handle/id.
- Execute due work when ticks advance.

## Deterministic Scheduling Lock (Replay-Critical)

1. Producer-authored schedule delays are milliseconds (`delayMs`), but runtime scheduling is tick-based only.
2. Runtime converts milliseconds to ticks using one canonical formula:
- `ticksFromNow = ceil(delayMs / tickIntervalMs)`
- for positive non-zero delays, enforce `ticksFromNow >= 1`.
3. Runtime stores and evaluates schedule deadlines using integer tick values only:
- `scheduledAtTick`
- `dueTick`
4. Event/schedule dispatch decisions must not use wall-clock comparisons; wall clock may only drive how many ticks to advance.
5. Same-`dueTick` ordering is stable and deterministic via tie-break chain:
- `dueTick`
- scheduler lane/priority (if configured)
- insertion sequence number
- deterministic id fallback
6. `TickCounter` is persistent runtime/session state and must not reset per wake invocation.
7. Per-wake local catch-up loop counters are pacing controls only; they are not canonical game-time values.

## Replay Determinism Guarantees (v1)

1. Replay correctness is evaluated at safe execution boundaries, not by transport chunk size.
2. Delta batch/chunk composition may vary by pacing/profile, but applied boundary state and ordering must remain stable.
3. Scheduler record/replay data must capture enough to re-drive ordering decisions:
- `delayMs`
- `tickIntervalMs` used for conversion
- `ticksFromNow`
- `scheduledAtTick`
- `dueTick`
- insertion sequence number
- dispatch sequence number
4. Tests should assert deterministic boundary outcomes and watermark continuity, not exact per-poll chunk cardinality.
5. Runtime must avoid nondeterministic inputs in scheduling paths (for example ad-hoc randomization or direct `DateTime.Now` ordering decisions).
6. Replay assertions use two explicit modes:
- `strict`: exact dispatch order and boundary sequence must match.
- `semantic`: boundary state/output/watermark continuity must match while allowing benign transport chunking variance.

## Schedule Invocation Policy (v1)

1. Schedules execute one bound action directly through the runtime action executor in v1.
2. Timer creation/cancellation remains authored through the event -> subscription -> action path.

## Milestone Relationship (initial stance)

1. Milestones are gameplay conditions that emit events when satisfied.
2. Milestones do not need to be implemented before event/tick scaffolding.
3. Once milestones exist, they can:
- Emit milestone events.
- Trigger phase-change actions.
- Start/cancel schedule chains via emitted events.

## Dependency Gates

1. Event foundation plan must be locked and signed off before scheduler and milestone integration work begins.
2. Phase actions can proceed in parallel if they do not alter event contract decisions.
3. Tick scheduler work depends on stable event dispatch semantics from the event foundation plan.

## Guardrails

1. Keep runtime logic in `Storyboard.Shared`.
2. Keep hosts (`StoryboardDesigner.App`, `Storyboard.Simulator`) as composition/pump layers only.
3. Do not introduce runtime dependency from `Storyboard.Simulator` to `StoryboardDesigner.App`.
4. Keep changes additive; avoid broad refactors in same pass.

## Validation Baseline

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`

## Open Umbrella Decisions

1. Phase validity constraints for `SetPhaseAction` (allow negative? upper bound?).
2. How to represent schedule ownership for bulk cancellation at umbrella-level orchestration seams (`phase`/`milestone`/`session`).

## Resolved Carry-Forward Decisions (From Topic 1 Closure)

1. Tick interval ownership for v1 is engine-config only; host-driven pulse ownership remains deferred.
2. Same-due-tick deterministic ordering is `(dueTick ASC, insertionSequence ASC, scheduleId ASC)`; lane/priority is deferred.
3. Replay CI default mode is `strict`; `semantic` is opt-in for explicitly designated playback cases.
4. Schedule invocation in v1 executes one bound action directly through the runtime action executor; timer creation/cancellation remains authored via event -> subscription -> action.
5. Repeating timers are included in v1 foundation (`fixedInterval`, `growingInterval`, `shrinkingInterval`) with shrink-expiry support.

## Determinism Lock Questions (Umbrella Follow-Ons)

1. Tick interval ownership:
- Resolved for Topic 1 v1: engine-config only.
- Follow-on question (future scope only): if producer-authored is introduced later, what validation bounds apply (min/max) and when can it change?

2. Same-due-tick tie-break semantics:
- Resolved for Topic 1 v1: stable ordering is `(dueTick, insertionSequence, scheduleId)`.
- Follow-on question (future scope only): if scheduler lane/priority is introduced, where should it insert in precedence semantics?

3. Replay assertion mode:
- Resolved for Topic 1 v1: CI default is `strict`; `semantic` is opt-in by case.
- Follow-on question: define any additional cases that should be permanently marked `semantic`.

## Exit Criteria

1. Event foundation plan is completed and signed off.
2. Runtime can process explicit phase actions.
3. Runtime can advance ticks and execute scheduled event emissions.
4. Composed scenario demonstrates event + phase + schedule interplay.
5. Build and focused regression tests pass.

Exit criteria progress (2026-08-24):

1. Satisfied: #1 (event foundation complete).
2. Satisfied: #2 (explicit phase actions implemented and covered in focused tests).
3. Satisfied: #3 (tick/schedule foundation complete under Topic 1).
4. Satisfied for closeout baseline: #4 (composed event + phase + ambient reconciliation coverage is in place; explicit replay assertion expansion is optional hardening).
5. Satisfied: #5 (focused regression and playback smoke gates reran green this pass).

## Remaining To Close

1. No blocking umbrella closeout items remain.
2. Optional post-archive hardening:
- explicit phase-transition replay assertions.
- additional designer phase validation-rule hardening if new gaps are discovered.

## Progress Note (2026-08-19)

1. Topic 1 time-tick events foundation is complete and closed in `plans/active/TIME_TICK_EVENTS_FOUNDATION_PLAN.md`.
2. Focused runtime regression and playback smoke gates passed after final playback rebaseline.
3. Remaining time-related items are explicitly deferred follow-ons (for example delta poll profile pacing refinement), not closure blockers for Topic 1.
4. Event addendum scope is functionally complete and closed in `plans/active/EVENT_ADDENDUMS_PLAN.md`; optional dispatch candidate-funnel observability counters are deferred as non-blocking hardening follow-on.
