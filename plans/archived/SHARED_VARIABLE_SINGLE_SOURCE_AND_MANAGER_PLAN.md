# Shared Variable Single-Source And Manager Plan

Status: Complete (M1-M4 complete)
Owner: StoryboardDesigner.App authoring + persistence + validation
Last updated: 2026-07-15

## Plan Maintenance (2026-07-15)
1. M1-M4 completion state is current and aligned with passing validation gates.
2. This plan is archive-ready with no remaining closure tasks.

## Objective

Lock a practical path to reduce shared-variable drift by making child variable links authoritative, while adding UX and tooling so users can inspect and repair shares directly.

## Background

The current project data can drift because share membership is effectively represented in two directions:
1. Child variables reference a shared variable id.
2. Project-level shared variable records also carry participant membership.

When those drift, users can get contradictory UX and warnings without a direct repair surface.

## Proposed Direction

1. Canonical membership source:
Child variable sharedVariableId links are authoritative.

2. Project-level shared variable records:
Retained as metadata records keyed by id (for example id + optional displayName), not as authoritative participant membership.

3. Participant lists:
Derived from child links for display and validation; never treated as independent authority.

4. UX:
Keep variable-centric editing and add explicit Leave Share support for the current variable.

5. Validation and repair:
Strengthen consistency validation and provide direct fix actions that operate through one mutation path.

## Scope

In scope:
1. Shared variable persistence shape and normalization strategy.
2. Variable editor shared-participation UX improvements, including remove-self.
3. Shared Variables Manager entry points and behavior.
4. Validation and repair action design.

Out of scope:
1. Runtime action script token changes.
2. Non-shared variable editing workflows.
3. Unrelated command-processing refactors.

## Constraints

1. One variable may participate in at most one share.
2. Existing projects must load without destructive migration.
3. Repair workflows must be deterministic and undoable.

## Candidate UX Entry Points

1. Variable editor shared section:
Show share name/id, participants, Add participant, Remove participant, Leave Share.

2. Tools menu:
Tools > Shared Variables Manager.

3. Project tree:
Shared Variables node under project root.

4. Validation deep-link:
Issue actions open manager filtered to the affected share.

## Design Questions To Resolve

1. SVSM-01: Final persisted JSON shape - do project-level shared records store only id and optional displayName, or also additional metadata fields?
2. SVSM-02: Migration policy - when legacy participant lists disagree with child links, which side wins and what diagnostics are emitted?
3. SVSM-03: Normalization timing - run reconciliation on load only, on save only, or both?
4. SVSM-04: Empty-share behavior - if no variables reference a share id, do we retain a metadata shell or delete it automatically?
5. SVSM-05: Singleton-share behavior - when one participant remains, keep share metadata or auto-collapse to non-shared?
6. SVSM-06: Leave Share UX - should leaving share require confirmation in all cases or only when share cardinality is small?
7. SVSM-07: Type/restriction consistency - should mixed value restrictions inside a share be blocked, warned, or auto-repaired?
8. SVSM-08: Scope policy - are cross-scope shares unrestricted, restricted by scope family, or policy-configurable?
9. SVSM-09: Name model - displayName uniqueness scope (project-wide or none) and rename conflict handling.
10. SVSM-10: Manager capabilities for V1 - read/rename only, or full participant add/remove/merge/split operations?
11. SVSM-11: Repair command design - should validation fixes be one-click auto-fix, preview-and-apply, or both?
12. SVSM-12: Auditability - what user-facing change log/undo labels are required for share membership and name edits?

## Locked Decisions (Accepted)

1. SVSM-01 (Persistence shape):
Project-level shared records persist metadata only: required id + optional displayName. No participant membership is persisted as authority.

2. SVSM-02 (Conflict winner):
Child sharedVariableId links are authoritative. Legacy participant metadata is reconciled from child links with non-blocking warnings.

3. SVSM-03 (Normalization timing):
Normalize on both load and save.

4. SVSM-04 (Empty shares):
Retain empty share metadata records. Show them in manager/editor and require explicit user-confirmed drop to delete.

5. SVSM-05 (Singleton shares):
Retain singleton shares. Flag as "Single Participant Share" for user review and optional user-driven cleanup.

6. SVSM-06 (Leave Share confirmation):
Leave Share always requires confirmation.

7. SVSM-07 (Type/restriction mismatch):
Mixed value restrictions/types in a share are blocked as errors.

8. SVSM-08 (Scope policy):
Membership is allowed within the same area and up the same ancestor scope chain (country, planet, globals).

9. SVSM-09 (Name uniqueness):
displayName duplicates are allowed and flagged as warnings; id remains canonical identity.

10. SVSM-10 (Manager V1 scope):
Ship staged V1 core operations; defer merge/split workflows to V2.

11. SVSM-11 (Repair UX):
Support both one-click quick fixes for safe deterministic repairs and preview-and-apply for wider-impact repairs.

12. SVSM-12 (Auditability):
Require clear undo labels and concise operation summaries; no persistent project audit log in V1.

## Committed Implementation Slice

Slice ID: SVSM-S1
Goal: ship deterministic single-source reconciliation and guardrail validation before UI manager work.

In scope (S1):
1. Persistence/reconcile core that treats child sharedVariableId links as authority.
2. Load-time and save-time normalization using the same mutation path.
3. Metadata-shell retention for empty shares and singleton-share flag computation.
4. Validation diagnostics for:
	- stale participant metadata drift,
	- type/restriction mismatch (error),
	- duplicate displayName (warning),
	- orphan metadata shells (warning/info).
5. Deterministic output ordering and regression tests for equivalent link state.

Out of scope (S1):
1. Shared Variables Manager UI surface.
2. Variable-editor Leave Share command UX.
3. Preview/Apply repair dialog UX (only backend-ready diagnostics/actions wiring if low risk).

Deliverables (S1):
1. Shared reconcile service in StoryboardDesigner.App model/persistence path.
2. Save pipeline updated to emit metadata-only shared records (id + optional displayName).
3. Validation rule updates with actionable issue payloads for next-slice UI binding.
4. Targeted tests covering migration drift, empty/singleton retention, mismatch blocking, and deterministic save.

Exit criteria (S1):
1. Loading then saving divergent legacy fixtures yields normalized metadata consistent with child links.
2. Re-running save without semantic changes produces byte-stable shared-variable sections.
3. Validation reports expected severity for mismatch/error/warning scenarios.
4. Solution build and focused regression gates pass.

Validation gates (S1):
1. dotnet build .\\StoryboardDesigner.slnx
2. dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj
3. dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"

## Validation Strategy (Planned)

1. Add consistency rules for dangling ids, stale metadata, and unresolved participants.
2. Add regression fixtures with intentionally divergent data (like asymmetric share membership).
3. Add tests that prove deterministic reconcile output from the same input.

## Milestones

1. M1: Lock data model and migration rules. (Complete)
2. M2: Implement Leave Share and variable-editor updates. (Complete)
3. M3: Implement Shared Variables Manager and deep-link entry points. (Complete)
4. M4: Implement validation fixes and finalize docs. (Complete)

## Current Delivery Status

Completed:
1. Single-source shared reconciliation on load/save.
2. Metadata-only shared persistence shape (id + optional displayName).
3. Deterministic shared section save regression coverage.
4. Shared validation diagnostics:
	- legacy participant drift warning,
	- orphan metadata shell warning,
	- duplicate displayName warning,
	- restriction mismatch error.
5. Leave Share command path with mandatory confirmation.
6. Shared relationship callouts for singleton and empty-shell states.
7. Shared manager deep-link seam and manager actionable operations (rename display name, explicit drop-empty with confirmation, reload).
8. Focused regression coverage for shared manager action handlers.
9. Navigation shared-variable token expectations aligned with single-source descriptors.
10. Final completion gate passed: dotnet test .\\StoryboardDesigner.slnx (793/793 passing).

Remaining to close plan:
1. None.

## Phase 4 Completion Gate

After M4 is implemented, run the full test sweep before marking this plan complete.

Required command:
1. dotnet test .\\StoryboardDesigner.slnx

Completion rule:
1. Do not mark the plan complete until the full sweep passes.

## Success Criteria

1. No contradictory share membership views across editor surfaces after normalize.
2. Users can remove themselves from a share without hand-editing files.
3. Validation issues for shared-variable drift include direct repair actions.
4. Project save output is deterministic for equivalent shared-link state.
