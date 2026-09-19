# Navigation Action Consolidation Plan

Last updated: 2026-09-13
Status: Deferred / Planning
Owner: Runtime follow-up

## Problem Statement

`NavigateDirection` and `NavigateToAdjacent` currently overlap heavily in traversal execution, output emission, and token contracts, but they differ in selection policy and result-code vocabulary. This creates long-term drift risk and inconsistent behavior/docs across equivalent navigation outcomes.

## Non-Goals

1. Immediate retirement of either action in this planning step.
2. One-shot migration of all existing projects without compatibility window.
3. Unrelated traversal feature changes.

## Current Baseline

1. `NavigateDirection` is direction-first and uses `RuntimeNavigateDirectionResultCode` outcomes.
2. `NavigateToAdjacent` is destination-first with directional fallback and uses `RuntimeNavigateToAdjacentResultCode` outcomes.
3. Both emit the same core output keys (`success`, `resultCode`, `navigatedDirection`, `priorRoomName`, `destinationRoomName`, `all`) but behavior and documentation have drifted.

## Proposed Shape

1. Introduce one canonical navigation action behavior profile with explicit resolution mode.
2. Keep both legacy action types temporarily as compatibility aliases into one shared navigation engine.
3. Retire one legacy action type only after migration tooling, fixtures, and guardrails demonstrate parity.

## Initial Scope Breakdown

### A) Runtime Engine Unification

1. Extract shared traversal selection/evaluation/emission flow.
2. Encode policy differences as mode flags instead of separate executors.
3. Normalize room-name output semantics and diagnostics behavior.

Estimated effort: Medium.

### B) Authoring and Migration Safety

1. Keep authoring/editing stable while introducing canonical path.
2. Add conversion/migration support for existing projects.
3. Preserve compatibility until retirement stage is complete.

Estimated effort: Medium.

## Structured Delivery Order And Session Handoffs (Locked)

1. Stage 01: Contracts And Shared Runtime Mapping
2. Stage 02: Designer Authoring UX
3. Stage 03: GameEngine Runtime Integration
4. Stage 04: Host Interface And Web Portal Runtime Consumption
5. Stage 05: Simulator Host Parity
6. Stage 06: Contract Retirement (Non-Backwards-Compatible)
7. Stage 07: Regression Hardening And Closeout

## Stage Inclusion Matrix

| Stage ID | Stage Name | Inclusion | Reason (if not Required) | Boundary Override |
| --- | --- | --- | --- | --- |
| 01 | Contracts And Shared Runtime Mapping (Backwards-Compatible) | Optional | Needed only if introducing canonical contract metadata before runtime cutover. | None |
| 02 | Designer Authoring UX | Required | Designer must expose canonical authoring path and compatibility messaging. | None |
| 03 | GameEngine Runtime Integration | Required | Core consolidation and compatibility routing live here. | None |
| 04 | Host Interface And Web Portal Runtime Consumption | Skipped | No host/web-specific API change is expected for navigation action identity. | None |
| 05 | Simulator Host Parity | Optional | Required only if simulator-specific UX/diagnostics differ after consolidation. | None |
| 06 | Contract Retirement (Non-Backwards-Compatible) | Required | Retirement of one action type is the explicit end goal. | None |
| 07 | Regression Hardening And Closeout | Required | Required for parity proof and safe closeout evidence. | None |

## Stage Details

### Stage 01: Contracts And Shared Runtime Mapping

Goal:
1. Decide whether canonical navigation metadata or mode descriptors need additive contract introduction.

Codebase context:
1. Primary projects in scope: `Storyboard.Shared.Contracts`, `Storyboard.SchemaCodegen`.
2. Key files expected first: runtime action contract enums/descriptors and schema docs.
3. Boundary constraints: additive-only; no removals or narrowing.
4. Required validation/tests before handoff: contract guardrails and transport codegen tests.
5. Allowed read scope: `plans/**`, `Storyboard.TransportCodegen.Tests/**`.
6. Allowed edit scope: `Storyboard.Shared.Contracts/**`, `Storyboard.SchemaCodegen/**`, `plans/active/NAVIGATION_ACTION_CONSOLIDATION_PLAN.md`, `plans/active/handovers/NAVIGATION_ACTION_CONSOLIDATION_01_CONTRACT_RUNTIME_HANDOFF.md`.

Primary output handoff document:
1. `plans/active/handovers/NAVIGATION_ACTION_CONSOLIDATION_01_CONTRACT_RUNTIME_HANDOFF.md`

Downstream usage:
1. Stage 02 and Stage 03 consume approved compatibility/canonical contract shape.

### Stage 02: Designer Authoring UX

Goal:
1. Expose one preferred authoring action path and preserve clarity for legacy existing actions.

Codebase context:
1. Primary projects in scope: `StoryboardDesigner.App`, `StoryboardDesigner.App.Tests`, `StoryboardDesigner.App.SmokeTests`.
2. Key files expected first: action editor viewmodels, action creation defaults, validation helpers.
3. Boundary constraints: preserve backward opening/editing of existing projects.
4. Required validation/tests before handoff: designer action editing tests and playback regression subset.
5. Allowed read scope: `plans/**`, `Storyboard.Shared.Contracts/**`.
6. Allowed edit scope: `StoryboardDesigner.App/**`, `StoryboardDesigner.App.Tests/**`, `StoryboardDesigner.App.SmokeTests/**`, `plans/active/handovers/NAVIGATION_ACTION_CONSOLIDATION_02_DESIGNER_HANDOFF.md`.

Primary output handoff document:
1. `plans/active/handovers/NAVIGATION_ACTION_CONSOLIDATION_02_DESIGNER_HANDOFF.md`

Downstream usage:
1. Stage 03 uses authoring output assumptions and conversion behavior.

### Stage 03: GameEngine Runtime Integration

Goal:
1. Implement shared navigation execution core and compatibility routing for both action types.

Codebase context:
1. Primary projects in scope: `Storyboard.GameEngine`, `Storyboard.GameEngine.Tests`, `Storyboard.Shared`.
2. Key files expected first: `RuntimeCommandActionExecutor.NavigateDirectionExecutableAction.cs`, `RuntimeCommandActionExecutor.NavigateToAdjacentExecutableAction.cs`, shared resolver utilities, result-code mapping adapters.
3. Boundary constraints: no contract retirement in this stage.
4. Required validation/tests before handoff: navigation runtime tests, action variable emission guardrails, targeted playback tests.
5. Allowed read scope: `plans/**`, `Storyboard.Shared.Contracts/**`, `StoryboardDesigner.App.Tests/**`, `Storyboard.GameClient.Tests/**`.
6. Allowed edit scope: `Storyboard.GameEngine/**`, `Storyboard.GameEngine.Tests/**`, `Storyboard.Shared/**`, `Storyboard.GameHost/**`, `Storyboard.GameClient.Tests/**`, `plans/active/handovers/NAVIGATION_ACTION_CONSOLIDATION_03_GAMEENGINE_HANDOFF.md`.

Primary output handoff document:
1. `plans/active/handovers/NAVIGATION_ACTION_CONSOLIDATION_03_GAMEENGINE_HANDOFF.md`

Downstream usage:
1. Stage 06 retirement depends on proven compatibility behavior from this stage.

### Stage 04: Host Interface And Web Portal Runtime Consumption

Goal:
1. Not planned unless later analysis reveals host/web dependence on action identity.

Codebase context:
1. Stage marked `Skipped` in this workstream.
2. Re-open only by explicit boundary decision.
3. Allowed read scope: `plans/**`.
4. Allowed edit scope: `plans/active/handovers/NAVIGATION_ACTION_CONSOLIDATION_04_HOST_WEBPORTAL_HANDOFF.md`.

Primary output handoff document:
1. `plans/active/handovers/NAVIGATION_ACTION_CONSOLIDATION_04_HOST_WEBPORTAL_HANDOFF.md`

Downstream usage:
1. None unless stage is re-activated.

### Stage 05: Simulator Host Parity

Goal:
1. Validate simulator behavior and diagnostics parity if runtime consolidation changes user-visible outcomes.

Codebase context:
1. Primary projects in scope: `Storyboard.Simulator`, `Storyboard.Simulator.Tests`, `Storyboard.Simulator.SmokeTests`.
2. Key files expected first: simulator rendering of navigation outcomes and logs.
3. Boundary constraints: keep simulator independent of designer.
4. Required validation/tests before handoff: simulator parity and smoke suites.
5. Allowed read scope: `plans/**`, `Storyboard.Shared/**`, `Storyboard.Shared.Contracts/**`.
6. Allowed edit scope: `Storyboard.Simulator/**`, `Storyboard.Simulator.Tests/**`, `Storyboard.Simulator.SmokeTests/**`, `plans/active/handovers/NAVIGATION_ACTION_CONSOLIDATION_05_SIMULATOR_HANDOFF.md`.

Primary output handoff document:
1. `plans/active/handovers/NAVIGATION_ACTION_CONSOLIDATION_05_SIMULATOR_HANDOFF.md`

Downstream usage:
1. Optional input to Stage 07 closeout evidence.

### Stage 06: Contract Retirement (Non-Backwards-Compatible)

Goal:
1. Retire one legacy navigation action contract surface after all consumers are migrated.

Codebase context:
1. Primary projects in scope: `Storyboard.Shared.Contracts`, `Storyboard.SchemaCodegen`.
2. Key files expected first: action type enum/schema descriptors and migration notes.
3. Boundary constraints: retirement list must be explicit and approved.
4. Required validation/tests before handoff: contract/interface guardrail gate and codegen transport tests.
5. Allowed read scope: `plans/**`, `Storyboard.TransportCodegen.Tests/**`.
6. Allowed edit scope: `Storyboard.Shared.Contracts/**`, `Storyboard.SchemaCodegen/**`, `plans/active/NAVIGATION_ACTION_CONSOLIDATION_PLAN.md`, `plans/active/handovers/NAVIGATION_ACTION_CONSOLIDATION_06_CONTRACT_RETIREMENT_HANDOFF.md`.

Primary output handoff document:
1. `plans/active/handovers/NAVIGATION_ACTION_CONSOLIDATION_06_CONTRACT_RETIREMENT_HANDOFF.md`

Downstream usage:
1. Stage 07 consumes retirement outcomes and validates full-suite stability.

### Stage 07: Regression Hardening And Closeout

Goal:
1. Prove parity and complete closeout evidence for consolidation + retirement.

Codebase context:
1. Primary projects in scope: repository-wide tests and closeout docs.
2. Key files expected first: regression tests, snapshots, plan/handover updates.
3. Boundary constraints: edits only for stabilization and documentation.
4. Required validation/tests before handoff: solution build, focused runtime guardrails, designer regression, transport guardrails.
5. Allowed read scope: repository-wide for validation evidence.
6. Allowed edit scope: stabilization tests/snapshots and `plans/active/handovers/NAVIGATION_ACTION_CONSOLIDATION_07_CLOSEOUT_HANDOFF.md`.

Primary output handoff document:
1. `plans/active/handovers/NAVIGATION_ACTION_CONSOLIDATION_07_CLOSEOUT_HANDOFF.md`

Downstream usage:
1. Marks workstream complete and ready for archive.

## Risk Register

1. Script branching regressions due to result-code vocabulary changes.
2. Legacy projects retaining mixed action types longer than expected.
3. Hidden host/simulator dependency on legacy diagnostics text.

## Validation Gates

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~Navigate|FullyQualifiedName~ActionVariableEmissionGuardrailTests|FullyQualifiedName~RuntimeActionOutputVariableManifestParityTests"`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`
4. `dotnet test .\Storyboard.TransportCodegen.Tests\Storyboard.TransportCodegen.Tests.csproj`

## MVP / Completion Acceptance Criteria

1. A canonical navigation execution path exists with explicit resolution mode.
2. Legacy action behavior is preserved through compatibility mapping until retirement stage.
3. One legacy action type is retired in Stage 06 with explicit migration evidence.
4. Stage 07 closeout records full guardrail pass and no unresolved contract drift.

## Final Closeout Checklist

1. All included stages have completed handoff documents.
2. Stage 06 retirement list and migration notes are finalized.
3. Required validation gates and targeted regressions are recorded in Stage 07.
4. Plan and all handoffs are archived together as one unit when complete.
