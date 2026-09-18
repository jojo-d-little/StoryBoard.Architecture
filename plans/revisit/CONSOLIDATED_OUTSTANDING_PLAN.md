# Consolidated Outstanding Plan

Status: Active (Bucket B primary; Bucket A closeout pending; Bucket C deferred)
Owner: Cross-plan backlog triage
Last updated: 2026-07-08

## 1. Purpose

Provide one clear view of what remains after reviewing future plans and closing completed scopes.

## 2. Source Plan Disposition

1. Action Runtime Refactor Plan: Closed.
2. Improve Instance Object Handling Plan: Open with narrow remainder.
3. Navigation Traversal Enhancement Plan: Open and primary remaining body of work.
4. Traversal Assistance Plan: Complete for current A/B/C scope; only optional v2 items remain.

Update 2026-07-08:

1. `plans/archived/REFOCUS_ON_OBJECT_IDS_PLAN.md` is Closed (archived).
2. `plans/archived/SCOPEKIND_SINGLE_ID_COURSE_CORRECTION_PLAN.md` is Completed (archived).
3. `plans/future/IMPROVE_INSTANCE_OBJECT_HANDLING_PLAN.md` remains Near Complete with one deferred closeout review item.
4. `plans/future/NAVIGATION_TRAVERSAL_ENHANCEMENT_PLAN.md` remains the primary remaining implementation body (Phases 4-6).

Disposition update:

1. Bucket A is not closed yet; only a narrow item-6 review/signoff remainder is open.
2. Bucket B remains the dominant active program scope.
3. Bucket C remains intentionally deferred behind Bucket B completion.

## 3. Remaining Work Buckets

## 3.1 Bucket A: Instance Handling Final Closeout (Small)

Source:

1. `plans/future/IMPROVE_INSTANCE_OBJECT_HANDLING_PLAN.md`.

Current status:

1. Near Complete (single deferred review/signoff remainder).

Remaining scope:

1. Execute Section 22 item 6 review (deferred by request).
2. Record keep/refactor decisions and final signoff.
3. Flip plan status from Near Complete to Closed.

Suggested validation:

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "QuantifiableRuntimeMaterializationTests|GameCommandProcessorFixtureTests|GameSimulatorPlaybackRegressionTests"

Exit criteria:

1. Section 22 item 6 decision recorded.
2. Plan marked Closed.

## 3.2 Bucket B: Navigation Traversal Program (Primary)

Source:

1. `plans/future/NAVIGATION_TRAVERSAL_ENHANCEMENT_PLAN.md`.

Current status:

1. In Progress (Phases 1-4 partially implemented; Phases 5-6 not started).

Remaining scope summary:

1. Phase 4 completion pass:
2. Full traversal-connection migration finish.
3. Scoped override UX completeness.
4. Shared-property UX and diagnostics acceptance matrix closeout.
5. Phase 5 execution:
6. Canonical export contract finalization.
7. Schema/version transition record and migration notes.
8. Deterministic snapshot normalization pass.
9. Phase 6 execution:
10. Legacy-path cleanup and hardening after phase 5 stabilization.

Suggested sequencing:

1. Finish Phase 4 acceptance checklist first.
2. Lock and execute Phase 5 contract finalization second.
3. Run full cleanup/hardening in Phase 6 last.

Suggested validation gate (per slice):

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"

Exit criteria:

1. Phase 4-6 marked Completed.
2. Export schema/version artifacts and migration notes committed.
3. Runtime and architecture guardrails green.

## 3.3 Bucket C: Traversal Assistance v2 Backlog (Optional)

Source:

1. `plans/archived/TRAVERSAL_ASSISTANCE_PLAN.md` Topic B deferred/open-question set.

Current status:

1. Complete for current scope; v2 backlog intentionally parked.

Deferred candidates:

1. Replace mode strategy (row-level in wizard vs outside wizard).
2. Batch scope expansion (room subset/area-wide runs).
3. Additional shared-property templates beyond door isOpen <-> leg isPassable.
4. Optional door reuse/discovery behavior.

Decision:

1. Keep deferred until Bucket B is complete, unless a user-facing pain point escalates priority.

Exit criteria:

1. Either formal v2 plan is spun up, or deferred list remains explicitly parked here.

## 4. Recommended Next Execution Order

1. Bucket A closeout (small, fast, reduces plan noise).
2. Bucket B Phase 4 completion and Phase 5 contract lock.
3. Bucket B Phase 6 cleanup.
4. Bucket C only if needed.

## 5. Definition of Done for This Consolidated Tracker

1. Bucket A closed.
2. Bucket B closed.
3. Bucket C either completed or explicitly deferred with owner and rationale.
4. This consolidated plan is then marked Closed and archived.

## 6. What Is Left and How To Resume

This tracker remains active. Use this section as the restart board.

### 6.1 What Is Left (Open)

1. Bucket A: finish `IMPROVE_INSTANCE_OBJECT_HANDLING_PLAN` Section 22 item 6 review, record keep/refactor decision, and close that plan.
2. Bucket B: complete Phase 4 acceptance checklist items, then execute Phase 5 export contract finalization, then Phase 6 cleanup/hardening.
3. Bucket C: keep deferred unless a concrete user pain point escalates it ahead of Bucket B.

### 6.2 How To Resume (Session Checklist)

1. Start with Bucket A closeout because it is small and unblocks cleaner backlog state.
2. After Bucket A is closed, move directly to Bucket B Phase 4 acceptance closure and keep slices narrow.
3. Do not start Bucket C unless Bucket B is done or an explicit reprioritization decision is recorded.
4. Update this consolidated tracker at the end of each slice with bucket status deltas only.

### 6.3 Resume Validation Runbook

1. `dotnet build .\StoryboardDesigner.slnx`
2. Bucket A focused gate:
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "QuantifiableRuntimeMaterializationTests|GameCommandProcessorFixtureTests|GameSimulatorPlaybackRegressionTests"`
4. Bucket B focused runtime gate:
5. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`
6. Replay smoke gate:
7. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`

### 6.4 Not-Ready-To-Archive Conditions

1. Bucket A still open.
2. Bucket B still open.
3. Bucket C remains deferred by design and does not block archive only after A and B are closed.
