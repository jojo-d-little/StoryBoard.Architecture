# Simulator Consolidation Post-Implementation Review

Date: 2026-07-12
Plan Reference: plans/active/simulator-consolidation.md

## Scope Audited
- Designer external run pipeline and setup persistence.
- Simulator startup argument bootstrap and per-project duplicate-launch arbitration.
- Runtime mapping touchpoints potentially affected by removal of embedded simulator UX.

## ProjectModelRuntimeSnapshotMapper Audit
Decision: Keep for now.

Rationale:
- The mapper is still used by active in-memory runtime session initialization paths in the designer host, not only tests.
- Current production usage remains in MainWindowViewModel.GameSimulator flow, which still initializes runtime snapshots from the authored project model.
- The mapper also remains heavily covered by existing runtime behavior tests and parity fixtures, indicating it is still a live contract in current architecture.

Evidence:
- StoryboardDesigner.App/ViewModels/MainWindowViewModel.GameSimulator.cs
- StoryboardDesigner.App/GameServices/ProjectModelRuntimeSnapshotMapper.cs
- Multiple runtime and regression tests in StoryboardDesigner.App.Tests referencing ProjectModelRuntimeSnapshotMapper.

## Cleanup Status
- Embedded simulator tab entry has been removed from main shell workspace tabs.
- External simulator workflow remains the primary entry via Run Simulator + Simulator Setup.

## Follow-Up Recommendation
- Open a dedicated follow-up to fully retire designer in-memory simulator command surfaces if desired.
- Re-run this mapper audit after that follow-up; mapper may become removable at that point depending on remaining host responsibilities.
