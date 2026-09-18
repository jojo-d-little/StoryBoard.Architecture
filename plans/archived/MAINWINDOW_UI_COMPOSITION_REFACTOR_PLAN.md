# MainWindow UI Composition Refactor Plan

Status: Phase 6 Complete (Lock-Off Checkpoint Reached)
Owner: StoryboardDesigner.App authoring UX
Last updated: 2026-07-10

Current progress snapshot:

1. Phase 0 decision lock-off completed for all items except deferred item 6.
2. Bite 0 scaffolding completed with new dedicated smoke test project.
3. Initial Bite 0 launch smoke test is passing locally.
4. Bite 1 shell AutomationIds added to MainWindow anchors and read-only presence smoke test is passing locally.
5. Bite 2 safe interaction smoke test added: select Game Simulator tab and verify command controls are discoverable.
6. Bite 3 happy-path smoke test added: load canonical fixture, initialize simulator in-memory, submit command, and verify console output growth.
7. Bite 4 baseline suite is green locally with temporary tolerance on hierarchy double-click editor-open assertions pending the known Create New Project workflow bug fix follow-on.
8. Slice A completed: startup object graph creation extracted to a dedicated Designer composition factory with behavior parity.
9. Slice A boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
10. Slice B completed: added shell orchestrator contracts, standardized workflow result envelope, immutable shell state snapshot, and initial MainWindowViewModel adapter wiring in Designer composition.
11. Slice B boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
12. Slice C completed: Create/Open/Save/Save As command handlers route through shell orchestrator contracts with parity fallback for non-composed harness contexts.
13. Slice C completed: unsaved-changes policy split implemented with dedicated save-guard policy service applied by orchestrator close workflow.
14. Slice C current boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
15. Slice C validation: build, focused regression gate, Bite5 create workflow smoke, and full smoke suite are green after orchestrator routing changes.
16. One-file-one-class/object requirement applied to shell workflow request/response contracts by moving the family into Orchestration/Workflows/Contracts with one type per file.
17. One-file-one-class/object requirement also applied to workflow result family by splitting generic result into its own file.
18. Slice D completed: orchestrator now owns baseline validation trigger policy after successful create/open workflows while focused validations remain available through existing feature paths.
19. Slice D boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
20. Slice D validation: solution build, focused regression gate, and Bite5 create workflow smoke are green.
21. Slice E completed: added diagnostics sink abstraction and routed orchestrator workflow diagnostics into the existing output console path.
22. Slice E boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
23. Slice E validation: solution build, focused regression gate, and Bite5 create workflow smoke are green.
24. Slice F progress: centralized global file command can-execute behavior from shell busy state via orchestrator state change notifications.
25. Slice F progress: removed synchronous awaiter blocking from orchestrator-backed file commands (async command handlers now await orchestrator tasks).
25. Slice F boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
26. Slice F validation (current step): solution build, focused regression gate, Bite5 create workflow smoke, and architecture guardrail coverage for no-blocking command handlers are green.
27. Slice G progress: added architecture guardrails for orchestrator command routing, diagnostics sink composition wiring, and one-type-per-file enforcement under the orchestration folder.
28. Slice G boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
29. Slice G validation (current step): ArchitectureSeparationGuardrailsTests + focused regression gate are green; solution build is green.
30. Close workflow backend validation: solution build, ArchitectureSeparationGuardrailsTests, focused regression gate, and Bite5 create workflow smoke are green.
31. Save-guard policy split validation: solution build, ArchitectureSeparationGuardrailsTests, focused regression gate, and Bite5 create workflow smoke are green.
32. Slice H progress: removed obsolete MainWindowViewModel.CreateNewProject workflow path after orchestrator routing completion.
33. Slice H validation (current step): solution build, ArchitectureSeparationGuardrailsTests + focused regression gate, and Bite5 create workflow smoke are green.
34. Slice H progress: simplified close workflow backend signature by removing redundant saveIfDirty parameter after save-guard policy split.
35. Slice H validation (current step): solution build, ArchitectureSeparationGuardrailsTests + focused regression gate, and Bite5 create workflow smoke are green.
36. Pre-phase close-out validation: solution build, full StoryboardDesigner.App.Tests suite, focused playback regression gate, and full smoke suite are all green.
37. Slice A-H tranche status: complete for pre-phase objectives and ready to transition into Phase 1 UI extraction work.
38. Phase 1 bullet 1 completed: extracted OutputConsolePane and SettingsAccessPane into dedicated controls with MainWindow host composition parity.
39. Phase 1 bullet 1 boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
40. Phase 1 bullet 1 validation: solution build, full StoryboardDesigner.App.Tests suite, and full smoke suite are green.
41. Phase 1 bullet 1 timing note: estimated 30 minutes, actual 23 minutes, variance -7 minutes (faster than estimate).
42. Phase 2 bullet 1 completed: extracted RoomDesignerWorkspace host control into a dedicated control and replaced inline MainWindow tab content.
43. Phase 2 bullet 1 boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
44. Phase 2 bullet 1 validation: solution build, full StoryboardDesigner.App.Tests suite, and full smoke suite are green.
45. Phase 2 bullet 2 completed: extracted reusable RoomImageSlotCard control and replaced repeated directional slot card markup in RoomDesignerWorkspace.
46. Phase 2 bullet 2 boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
47. Phase 2 bullet 2 validation: solution build, full StoryboardDesigner.App.Tests suite, and full smoke suite are green.
48. Phase 2 bullet 3 completed: extracted Default Room View preview into RoomDefaultViewPreview and replaced inline center preview markup.
49. Phase 2 bullet 3 boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
50. Phase 2 bullet 3 validation: solution build, full StoryboardDesigner.App.Tests suite, and full smoke suite are green.
51. Phase 2 bullet 4 completed: extracted reusable vertical look slot control for Up/Down and replaced inline look slot markup in RoomDesignerWorkspace.
52. Phase 2 bullet 4 boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
53. Phase 2 bullet 4 validation: solution build, full StoryboardDesigner.App.Tests suite, and full smoke suite are green.
54. Phase 3 bullet 1 completed: extracted AreaMapDesignerWorkspace host control and replaced inline MainWindow area-designer tab content.
55. Phase 3 bullet 1 boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
56. Phase 3 bullet 1 validation: solution build, full StoryboardDesigner.App.Tests suite, and full smoke suite are green.
57. Phase 3 bullet 2 completed: extracted AreaMapToolbar control and replaced inline toolbar markup in AreaMapDesignerWorkspace.
58. Phase 3 bullet 2 boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
59. Phase 3 bullet 2 validation: solution build, full StoryboardDesigner.App.Tests suite, and full smoke suite are green.
60. Phase 3 bullet 3 completed: extracted AreaMapCanvas control and AreaRoomPlacementCard control; replaced inline canvas and room-placement templates in AreaMapDesignerWorkspace.
61. Phase 3 bullet 3 boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
62. Phase 3 bullet 3 validation: solution build, full StoryboardDesigner.App.Tests suite, and full smoke suite are green.
63. Phase 4 bullet 1 completed: extracted ProjectHierarchyPane control and replaced inline MainWindow hierarchy pane markup with host control composition parity.
64. Phase 4 bullet 1 boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
65. Phase 4 bullet 1 validation: solution build, full StoryboardDesigner.App.Tests suite, and full smoke suite are green.
66. Phase 4 bullet 2 completed: moved hierarchy tree templates/styles into ProjectHierarchyPane control-level resources and removed hierarchy-specific resource ownership from MainWindow.
67. Phase 4 bullet 2 boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
68. Phase 4 bullet 2 validation: solution build, full StoryboardDesigner.App.Tests suite, and full smoke suite are green.
69. Phase 4 bullet 3 completed: added deterministic selected-editor automation signals for Room/Area workspaces and updated Bite4 hierarchy smoke assertions to verify editor selection via those signals.
70. Phase 4 bullet 3 boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
71. Phase 4 bullet 3 validation: targeted Bite4 hierarchy smoke, full solution build, full StoryboardDesigner.App.Tests suite, and full smoke suite are green.
72. Phase 5 bullet 1 completed: extracted GameSimulatorWorkspace control and replaced inline MainWindow Game Simulator tab content with host control composition parity.
73. Phase 5 bullet 1 boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
74. Phase 5 bullet 1 validation: solution build, full StoryboardDesigner.App.Tests suite, and full smoke suite are green.
75. Phase 5 scope decision: keep GameSimulatorWorkspace as a single cohesive standalone control for now; defer internal simulator subcomponent extraction until post-rework requirements are defined.
76. Phase 5 deferment rationale: avoid premature decomposition in a known future-change area while still removing simulator complexity from MainWindow.
77. Phase 6 cleanup progress: moved simulator-specific view mechanics (tree double-click variable edit, command Enter submit, console copy shortcut/context menu handling) from MainWindow into GameSimulatorWorkspace code-behind.
78. Phase 6 cleanup boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
79. Phase 6 cleanup validation: solution build, full StoryboardDesigner.App.Tests suite, and full smoke suite are green.
80. Phase 6 cleanup progress: moved hierarchy right-click selection/context-menu construction and hierarchy double-click editor-routing mechanics from MainWindow into ProjectHierarchyPane code-behind.
81. Phase 6 cleanup boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
82. Phase 6 cleanup validation: solution build, full StoryboardDesigner.App.Tests suite, and full smoke suite are green.
83. Phase 6 cleanup progress: moved hierarchy drag gesture mechanics (preview left-down/key gesture marking and room drag-drop initiation state) from MainWindow into ProjectHierarchyPane code-behind.
84. Phase 6 cleanup boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
85. Phase 6 cleanup validation: solution build, full StoryboardDesigner.App.Tests suite, and full smoke suite are green.
86. Phase 6 cleanup progress: moved area map pan/scroll mechanics and room-drop-on-canvas workflow from MainWindow into AreaMapCanvas, and moved room-placement drag/move workflow from MainWindow into AreaRoomPlacementCard.
87. Phase 6 cleanup boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
88. Phase 6 cleanup validation: solution build, full StoryboardDesigner.App.Tests suite, and full smoke suite are green.
89. Phase 6 cleanup progress: moved room placement context actions (Traversal Wizard and Manage Traversals) and the room traversal review/edit dialog workflow from MainWindow into AreaRoomPlacementCard.
90. Phase 6 cleanup progress: moved navigation arrow interaction mechanics (clear pending destination mode, edit traversal on double-click, and activate change-destination mode) from MainWindow into AreaMapCanvas, leaving only minimal pending-destination coordination state in MainWindow.
91. Phase 6 cleanup boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
92. Phase 6 cleanup validation: solution build, full StoryboardDesigner.App.Tests suite, and full smoke suite are green.
93. Phase 6 completion note: MainWindow now primarily contains shell/composition concerns (window lifecycle, pane sizing/state restore, high-level selection synchronization, global edited-state hooks, and minimal cross-surface coordination helpers).
94. Phase 6 acceptance summary: MainWindow complexity materially reduced, behavior parity preserved under current UX baseline, and required build/tests/smoke validation gates are green.
95. Full status review checkpoint completed (2026-07-10): active plan remains complete through Phase 6 with no open Phase 6 implementation slices.
96. Deferred scope confirmation: Phase 5 bullets 2 and 3 (internal simulator command/console and debug popup decomposition) remain intentionally deferred pending future simulator rework requirements.
97. Fresh validation checkpoint (2026-07-10): `dotnet build .\StoryboardDesigner.slnx -p:RestoreIgnoreFailedSources=true` is green.
98. Fresh validation checkpoint (2026-07-10): `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj -p:RestoreIgnoreFailedSources=true` is green (590 passed, 0 failed, 0 skipped).
99. Fresh validation checkpoint (2026-07-10): `dotnet test .\StoryboardDesigner.App.SmokeTests\StoryboardDesigner.App.SmokeTests.csproj -p:RestoreIgnoreFailedSources=true` is green (7 passed, 0 failed, 1 skipped deep opt-in test).
100. Next-step status: no defined Phase 7 in this plan yet; next planning action is to define post-refactor hardening/guardrail scope or close this plan as complete.
101. Lingering coupling item addressed: pending-destination arrow state ownership moved out of MainWindow and localized to area-map control surfaces (AreaMapCanvas via AreaMapDesignerWorkspace delegation for Escape cancellation).
102. Lingering coupling boundary check note: Designer-only changes, no Storyboard.Shared edits, no Storyboard.Simulator edits, no JSON format changes.
103. Lock-off validation checkpoint (2026-07-10): `dotnet build .\StoryboardDesigner.slnx -p:RestoreIgnoreFailedSources=true` is green.
104. Lock-off validation checkpoint (2026-07-10): `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj -p:RestoreIgnoreFailedSources=true` is green (590 passed, 0 failed, 0 skipped).
105. Lock-off validation checkpoint (2026-07-10): `dotnet test .\StoryboardDesigner.App.SmokeTests\StoryboardDesigner.App.SmokeTests.csproj -p:RestoreIgnoreFailedSources=true` is green (7 passed, 0 failed, 1 skipped deep opt-in test).
106. Effort lock-off note: MainWindow pending-destination state seam removed; Escape cancellation parity preserved through AreaMapDesignerWorkspace -> AreaMapCanvas delegation, with overall refactor objectives met for current phase scope.
107. Lock-off hardening: added architecture guardrail test ensuring MainWindow does not re-own pending-destination state and that cancellation delegation remains AreaMapDesignerWorkspace -> AreaMapCanvas.
108. Final validation checkpoint (2026-07-10): `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj -p:RestoreIgnoreFailedSources=true` is green (591 passed, 0 failed, 0 skipped).
109. Final closure checkpoint (2026-07-10): solution build and smoke suite remain green after guardrail hardening; plan is ready to close unless a new Phase 7 scope is introduced.

## 0. MainWindow Composition Refactor Contract (Locked)

Status: Locked before implementation
Decision date: 2026-07-10
Checkpoint: 30 of 30 lock-off questions completed

Cross-cutting design rule addendum:

1. Prefer explicit return/result types over ref/out parameters.
2. Ref/out is exception-only and must be justified when used.
3. One file per class/object is the default codebase rule. Class/object families should be organized in folders, with one type per file.

Locked decisions:

1. Use a single shell orchestrator as the app-level workflow coordinator.
2. Multi-step app workflows are orchestrator-owned.
3. Unsaved-changes policy is split: orchestrator flow plus dedicated save-guard policy service.
4. Feature ViewModels may perform approved read-only queries; all writes route through orchestrator workflows.
5. Cross-feature communication is orchestrator intent/event driven only; no direct ViewModel-to-ViewModel calls.
6. Startup object graph creation moves to a dedicated composition module/factory.
7. Orchestrator exposes explicit strongly-typed use-case operations (not generic string dispatch).
8. Validation policy is hybrid: orchestrator baseline validation after create/open plus feature-requested focused validation.
9. Validation services return structured data only; UI presentation remains outside validation services.
10. Busy/cancel behavior is unified under orchestrator-managed contracts.
11. Feature ViewModels use minimal dependency surfaces plus minimal orchestrator interfaces.
12. Orchestrator workflows are async-first with cancellation-ready contracts.
13. Global UI signals (title/status/banner) derive from one orchestrator-maintained shell state model.
14. State strategy is hybrid: immutable shell/global snapshots; mutable feature/domain editing state.
15. Workflow failures map to a standardized error contract.
16. Structured internal workflow events are required; status console key activity echo remains and receives routed diagnostics.
17. Add a diagnostics sink abstraction and route selected diagnostics to status console echo.
18. Dialog interactions are brokered through a single orchestrator-facing dialog gateway.
19. Global command enable/disable rules are centralized and shell-state driven.
20. Undo/redo internals remain feature/domain-owned; orchestrator coordinates workflow boundaries only.
21. Persistence boundaries are explicit and enforced so authored content and app/session state paths cannot be mixed.
22. Export domain logic remains in dedicated services; orchestrator coordinates entry/exit only.
23. Simulator integration remains a strict boundary call; no dependency on simulator host internals.
24. Orchestrator workflow contracts use explicit request/response DTOs.
25. No-blocking-UI rule is phased: enforce now for file/validation/export workflows, then expand.
26. Use one standard workflow result envelope for orchestrator operations.
27. Dependency lifetime rules are explicit and codified.
28. Add architecture guardrail tests specific to this refactor (phased rollout allowed).
29. All tests must pass for merge; no temporary bypass flags.
30. This written contract is the source of truth before implementation starts.

Hard boundary and approval gates (added):

1. This is a StoryboardDesigner.App UI composition refactor.
2. No code changes are allowed in Storyboard.Shared for this refactor.
3. No code changes are allowed in Storyboard.Simulator for this refactor.
4. If a legitimate need appears to change Storyboard.Shared or Storyboard.Simulator, work must pause and request explicit approval before any code change in those projects.
5. No JSON file format or schema changes are allowed in this refactor (native project files, sidecar files, clean export files, or related contract DTO shapes).
6. If a legitimate need appears to change any JSON format/schema/contract, work must pause and request explicit approval before code changes.
7. Until explicit approval is given, proceed with Designer-only implementation that preserves existing file format behavior.

Implementation execution rules (added):

1. Each implementation PR/slice must include a short boundary check note confirming: Designer-only changes, no Shared/Simulator edits, no JSON format changes.
2. Any detected boundary pressure is logged as "Approval Required" and excluded from the current slice.
3. Refactor completion is blocked if boundary checks are missing.

## 1. Purpose

Reduce MainWindow UI complexity by extracting cohesive regions into dedicated controls while preserving existing behavior, MVVM boundaries, and designer/runtime separation.

## 2. Why This Plan Exists

MainWindow currently carries a large amount of template, layout, and interaction surface area.

Symptoms:

1. High merge conflict probability on parallel UI work.
2. Elevated regression risk when changing room and map authoring surfaces.
3. Slower review and debugging because unrelated concerns are co-located.
4. Repeated markup sections increase maintenance cost.

## 3. Scope

In scope:

1. Extract MainWindow UI regions into focused controls under StoryboardDesigner.App.
2. Keep existing data flow through MainWindowViewModel and existing commands.
3. Move view-only interaction code to local control code-behind where appropriate.
4. Add UI smoke automation as a mandatory pre-implementation gate.

Out of scope:

1. Runtime contract/schema changes in Storyboard.Shared.
2. Simulator host project restructuring.
3. Feature expansion beyond parity refactor unless explicitly approved.

## 4. Phase Prerequisite Gate (Required Before Refactor Implementation)

## Phase 0: UI Smoke Automation Baseline (Prerequisite)

Goal:

1. Establish minimal but reliable desktop UI smoke automation to detect integration regressions during control extraction.

Required deliverables:

1. Select and wire a WPF desktop UI automation framework (recommended: FlaUI UIA3).
2. Add stable AutomationId identifiers to critical MainWindow controls used by smoke tests.
3. Implement and stabilize baseline smoke suite covering critical navigation paths.

Required baseline tests:

1. App shell loads and key panes are present.
2. Hierarchy interaction can open Room Designer tabs.
3. Area map designer supports room placement path verification.
4. Game simulator initialize plus command plus console append flow works.
5. Output pane clear/save actions function.

Phase 0 acceptance gate:

1. Smoke tests run green locally and in CI on at least two consecutive runs.
2. Tests are non-flaky under normal workstation load.
3. Control identifiers are documented for refactor-safe selector updates.
4. No MainWindow extraction work begins before this gate is marked complete.

## 4.1 Confidence-First Micro Rollout (Small Bites)

This rollout intentionally starts with tiny wins and low assertion depth so UI automation can earn trust before it gates larger work.

### Bite 0: Skeleton Only

1. Create a single UI automation test that launches the app and confirms main window is discoverable.
2. Do not automate workflows yet.
3. Focus only on process launch stability and teardown reliability.

Acceptance:

1. Test passes locally three times in a row.
2. No orphaned app processes after test run.

### Bite 1: Read-Only Presence Check

1. Add AutomationId values to three shell anchors (hierarchy pane, workspace tabs host, output pane).
2. Add one read-only test asserting those anchors exist.
3. No clicks, typing, drag/drop, or dialogs.

Acceptance:

1. Presence test passes locally three times in a row.
2. Selectors rely on AutomationId only.

### Bite 2: Single Safe Interaction

1. Add one interaction test that switches to the Game Simulator tab and verifies command input controls are present.
2. Keep assertions coarse (control exists, enabled state expected).

Acceptance:

1. Interaction test is stable across three consecutive runs.
2. No timing sleeps; use bounded polling/waits only.

### Bite 3: One End-to-End Happy Path

1. Add one full-path smoke: initialize in-memory session, submit one command, verify console line count increases.
2. Keep strict text matching out of scope for this bite.

Acceptance:

1. Happy path passes locally and in CI on two consecutive runs.
2. Failure output is readable and points to the failed step.

### Bite 4: Expand to Minimum Gate Set

1. Add remaining baseline smoke tests defined in Phase 0.
2. Keep each test focused to one user journey.
3. Keep fixture setup shared and small.

Acceptance:

1. Full smoke set runs green locally and CI twice in a row.
2. Suite runtime stays within agreed budget.

Execution rule:

1. Do not start UI composition refactor phases until Bite 4 is complete.
2. If any bite is flaky, pause expansion and stabilize before proceeding.

Post-Bite-4 follow-on (agreed):

1. Create a small automation-only sample project fixture dedicated to smoke tests.
2. Keep the fixture intentionally stable and minimal so UI smoke tests remain deterministic.
3. Use this fixture as the default smoke data source instead of relying on broader demo samples.

Post-initial-smoke enhancement (new):

1. Implement a targeted fix for the known Create New Project workflow bug in authoring UX.
2. Adapt Bite 4 hierarchy smoke coverage to use the corrected create-project behavior.
3. Keep this enhancement scoped and regression-tested before any broader MainWindow refactor phases begin.

Follow-on acceptance:

1. Fixture can be loaded automatically by smoke tests.
2. Bite 0-4 smoke tests pass against the automation-only fixture.
3. Fixture content and ownership are documented in this plan.

## 4.2 WPF Smoke Prerequisite Design Lock-Off Checklist

Answer each item before implementation starts.

1. Framework lock:
Question: Do we lock on FlaUI UIA3 for desktop automation?
Decision: Locked - FlaUI UIA3 (2026-07-10)

2. Test project shape:
Question: Do we use a separate UI smoke test project?
Decision: Locked - Yes, separate new UI smoke test project (2026-07-10)

3. Scope lock:
Question: Do we cap initial work at Bite 0 through Bite 4 only?
Decision: Locked - Yes, cap initial scope to Bite 0 through Bite 4 (2026-07-10)

4. Refactor gate:
Question: Is "no MainWindow refactor until Bite 4 is green" a hard stop?
Decision: Locked - Yes, hard stop until Bite 4 is green (2026-07-10)

5. Selector policy:
Question: Do we require AutomationId-first selectors and avoid text-based selectors by default?
Decision: Locked - Yes, AutomationId-first selectors by default (2026-07-10)

6. AutomationId ownership:
Question: Is the feature author responsible for adding/updating AutomationIds for touched UI?
Decision: Deferred (2026-07-10)

7. Execution environment:
Question: Must smoke tests pass locally and in CI from the start?
Decision: Locked - CI-ready by design; local execution required now; CI execution enforced once pipeline integration exists (2026-07-10)

8. Stability threshold:
Question: Do we require 3/3 local passes and 2/2 CI passes before expanding scope?
Decision: Locked - 3/3 local passes required now; 2/2 CI passes required after pipeline integration (2026-07-10)

9. Runtime budget:
Question: What is the max allowed runtime for the Phase 0 smoke suite?
Decision: Locked - 5 minutes max for Phase 0 smoke suite (2026-07-10)

10. Flake policy:
Question: If a smoke test flakes, do we freeze new smoke additions until stabilized?
Decision: Locked - Yes, freeze new smoke additions until stabilized (2026-07-10)

11. Failure artifacts:
Question: Do we require screenshot and failure-step logging for each failed smoke test?
Decision: Locked - Yes, require screenshot and failure-step logging for each failure (2026-07-10)

12. Data baseline:
Question: Do we lock one canonical sample project/fixture for all Phase 0 smoke tests?
Decision: Locked - Yes, use one canonical fixture for all Phase 0 smoke tests (2026-07-10)

## 5. Planned Refactor Phases

## 5.1 Locked Implementation Sequence (Designer-Only)

Execution order:

1. Slice A: composition root extraction in StoryboardDesigner.App only.
- Introduce dedicated composition module/factory.
- Keep runtime behavior parity.
- No Shared/Simulator edits.

2. Slice B: shell orchestrator contract and shell state model.
- Add strongly-typed workflow contracts and result envelope.
- Add immutable shell/global state snapshot model.
- Keep feature/domain editing state mutable.

3. Slice C: workflow migration for create/open/save/close + unsaved-changes policy split.
- Route multi-step workflows through orchestrator.
- Add save-guard policy service.
- Preserve existing dialog behaviors via gateway abstraction.

4. Slice D: validation orchestration.
- Add orchestrator baseline validation on create/open.
- Preserve focused feature-invoked validation paths.
- Keep validation services data-only.

5. Slice E: diagnostics and status console routing.
- Add diagnostics sink abstraction.
- Route selected workflow diagnostics into existing status console echo.

6. Slice F: command availability and busy/cancel unification.
- Centralize global command rules from shell state.
- Apply phased no-blocking-UI policy to file/validation/export workflows first.

7. Slice G: architecture guardrail tests.
- Add/refine tests for orchestrator ownership, dependency boundaries, and cross-VM call prevention.
- Keep guardrails in StoryboardDesigner.App.Tests.

8. Slice H: parity verification and cleanup.
- Remove obsolete composition duplication.
- Run full validation gates.

Per-slice required checks:

1. Boundary check note: "Designer-only changes; no Shared/Simulator edits; no JSON format changes."
2. Build and required tests pass.
3. If boundary pressure is discovered, mark "Approval Required" and pause that path.

## Phase 1: Low-Risk Visual Extractions

Estimate calibration note (requested):

1. Track actual elapsed implementation time for Phase 1 bullet 1 (OutputConsolePane and SettingsAccessPane extraction only).
2. When Phase 1 bullet 1 is complete, report estimate vs actual with a short variance summary.

Phase 1 bullet 1 completion note:

1. Estimate: 30 minutes.
2. Actual elapsed: 23 minutes.
3. Variance summary: completed 7 minutes faster than estimate due to straightforward XAML extraction and no smoke selector churn.

1. Extract OutputConsolePane control.
2. Extract SettingsAccessPane control.
3. Keep behavior parity and command bindings unchanged.

Acceptance:

1. No behavior changes.
2. Existing tests pass.
3. Phase 0 smoke suite remains green.

## Phase 2: Room Designer Surface Decomposition

1. Extract RoomDesignerWorkspace host control.
2. Extract reusable RoomImageSlotCard control for directional slots.
3. Extract RoomDefaultViewPreview control.
4. Extract vertical look slot control (Up/Down).

Acceptance:

1. No change in room image editing outcomes.
2. Overlay controls and previews remain parity.
3. Smoke suite remains green with selector updates only where necessary.

## Phase 3: Area Map Designer Surface Decomposition

1. Extract AreaMapDesignerWorkspace host control.
2. Extract AreaMapToolbar control.
3. Extract AreaMapCanvas control and AreaRoomPlacementCard control.
4. Keep drag/drop and traversal interactions behaviorally equivalent.

Acceptance:

1. Drop, move, traversal context menu, and destination-change flows behave as before.
2. No regressions in traversal editing workflows.
3. Smoke suite remains green.

## Phase 4: Hierarchy and Template Resource Isolation

1. Extract ProjectHierarchyPane control.
2. Move tree templates/styles into dedicated resource dictionary or control-level resources.
3. Keep selection synchronization behavior unchanged.

Acceptance:

1. Tree rendering parity and context actions unchanged.
2. Double-click routing and selection sync remain stable.
3. Smoke suite remains green.

## Phase 5: Game Simulator Tab Isolation

1. Extract GameSimulatorWorkspace control.
2. Extract simulator command and console control.
3. Isolate debug popup content into dedicated control.

Acceptance:

1. Initialize, command submission, diagnostics, and playback controls remain functional.
2. Console interactions keep parity.
3. Smoke suite remains green.

## Phase 6: Code-Behind Rationalization and Cleanup

1. Reduce MainWindow code-behind to composition and shell-level concerns.
2. Move control-specific view mechanics to extracted controls.
3. Remove obsolete duplication and dead handlers.

Acceptance:

1. MainWindow file size and complexity materially reduced.
2. No behavior regressions from current UX baseline.
3. Build and targeted regression tests pass.
4. Smoke suite remains green.

## 6. Candidate Control Inventory

Priority 1:

1. OutputConsolePane
2. SettingsAccessPane
3. RoomImageSlotCard
4. RoomDefaultViewPreview

Priority 2:

1. RoomDesignerWorkspace
2. AreaMapToolbar
3. AreaRoomPlacementCard
4. GameSimulatorCommandConsole

Priority 3:

1. AreaMapCanvas
2. AreaMapDesignerWorkspace
3. ProjectHierarchyPane
4. GameSimulatorWorkspace

## 7. Validation and Regression Gates

Per phase minimum:

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
4. UI smoke suite execution

Runtime-boundary touchpoints (if any arise during refactor):

1. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"

## 8. Architecture Guardrails

1. Keep StoryboardDesigner.App-specific UI concerns in Designer project.
2. Keep Storyboard.Simulator independent of StoryboardDesigner.App.
3. Keep reusable runtime/domain logic in Storyboard.Shared.
4. Keep view code-behind limited to view mechanics, not persistence/export/runtime decision logic.

## 9. Open Decisions To Resolve During Phase 0

1. Final UI automation framework and package versions.
2. CI job shape for desktop UI smoke execution.
3. Initial AutomationId naming convention and ownership.
4. Flaky-test quarantine rule and stabilization threshold.
5. Final location/name for automation-only sample fixture (target after Bite 4).

## 10. Exit Criteria

1. MainWindow is reduced to shell composition and cross-surface coordination only.
2. Core authoring workflows remain behaviorally unchanged.
3. UI smoke suite guards critical integration paths and runs reliably in CI.
4. Refactor can continue incrementally with low regression risk.
