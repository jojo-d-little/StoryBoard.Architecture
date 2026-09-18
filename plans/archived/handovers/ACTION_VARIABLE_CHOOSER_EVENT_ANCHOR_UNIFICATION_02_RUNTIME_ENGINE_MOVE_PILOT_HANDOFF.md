# Action Variable Chooser Event Anchor Unification - Stage 02 Runtime Engine Move Pilot Handoff

Status: Completed (executed)
Stage: 2 of 7
Date: 2026-09-12
Owner Session: GitHub Copilot (GPT-5.3-Codex)

## Opening Prompt (Use To Start This Stage)

Start Stage 02 of [plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md](plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md).
Complete only Stage 02 runtime engine pilot for one action end-to-end (primary: MoveRoomObjectOnGrid; companion parity: MoveRoomObjectByPoints where shared plumbing applies).
Before coding, read ENHANCEMENT_GUIDELINES.md and AGENTS.md.
Do not begin broad designer UX rollout or global legacy-token retirement in this stage.
Honor the stage boundary allowlists below.
Update this handoff with exact runtime files changed, test evidence, and clear pass/fail parity notes.

## Stage Boundary Allowlist Snapshot (From Main Plan)

Default deny rule:
1. Any path not explicitly listed in allowed read/edit scope is out of scope for this stage.

Edit-implies-read rule:
1. Any path in allowed edit scope is automatically readable.
2. Allowed read scope lists only extra read-only dependencies.

1. Allowed read scope:
- plans/**
- Storyboard.GameEngine/**
- StoryboardDesigner.App/**
- StoryboardDesigner.App.Tests/**

2. Allowed edit scope:
- Storyboard.GameEngine/**
- Storyboard.GameEngine.Tests/**
- Storyboard.GameHost/** (only if required for runtime event payload parity)
- plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md
- plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_02_RUNTIME_ENGINE_MOVE_PILOT_HANDOFF.md

## Scope Completed

1. Completed runtime pilot for MoveRoomObjectOnGrid with MoveRoomObjectByPoints parity companion.
2. Preserved canonical move action output emission set (success, resultCode, movedObjectName, moveDirection, moveDistance, all).
3. Implemented additive runtime support-object context propagation for move-driven stack landings.
4. Corrected stack event target derivation to use landing support object telemetry instead of moved-object telemetry.
5. Added focused runtime tests for support-context telemetry and move-family parity.

## Files Changed

1. Storyboard.GameEngine/GameServices/RuntimeContext/RuntimeMoveRoomObjectOnGridAttemptResult.cs
- Added `LandingSupportObjectId` as additive runtime attempt-result context for stack-aware move outcomes.

2. Storyboard.GameEngine/GameServices/Commands/SessionRuntimeScopeMutationGateway.cs
- Tracked and emitted landing support object id through move attempt outcomes.

3. Storyboard.GameEngine/GameServices/Commands/GameCommandMoveLegTelemetry.cs
- Added `LandingSupportObjectId` to per-leg runtime telemetry.

4. Storyboard.GameEngine/GameServices/Actions/GameActions/RuntimeActionExecutable/RuntimeCommandActionExecutor.MoveRoomObjectOnGridExecutableAction.cs
- Propagated move attempt landing support context into emitted move-leg telemetry.

5. Storyboard.GameEngine/GameManager/GameManager.cs
- Updated stack target event context resolution to derive target from `LandingSupportObjectId` rather than moved-object identity.

6. Storyboard.GameEngine.Tests/RuntimeMoveRoomObjectOnGridActionTests.cs
- Added assertions proving non-stack moves emit null support context and stack landings emit support object id.
- Added MoveRoomObjectByPoints parity assertions for non-stack point-sequence moves.

7. Storyboard.GameEngine.Tests/ActionVariableEmissionGuardrailTests.cs
- Added parity companion test asserting MoveRoomObjectByPoints declared token set equals MoveRoomObjectOnGrid declared token set.

8. plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_02_RUNTIME_ENGINE_MOVE_PILOT_HANDOFF.md
- Updated from seeded to executed with evidence and parity notes.

## Contract/Interface Impact

1. No schema contract changes are required by default for this runtime pilot.
2. Runtime context/model additions were applied to surface support-object context for move-driven stacking.
3. Additive runtime-output compatibility preserved (no removals or behavior-breaking contract renames).

## Runtime Code Review Map (Exact Targets)

Primary runtime action flow:
1. Storyboard.GameEngine/GameServices/Actions/GameActions/RuntimeActionExecutable/RuntimeCommandActionExecutor.MoveRoomObjectOnGridExecutableAction.cs
- Review move outcome composition and action-variable injection path.
- Review move-leg telemetry construction used downstream by event emission.

2. Storyboard.GameEngine/GameServices/References/MoveRoomObjectOnGridActionVariableResolver.cs
- Review emitted action.* keys and compatibility behavior.
- Ensure manifest-declared keys are represented in runtime emission.

3. Storyboard.GameEngine/GameServices/RuntimeContext/RuntimeMoveRoomObjectOnGridAttemptResult.cs
- Review whether result shape needs additive fields for support/bottom object context.

4. Storyboard.GameEngine/GameServices/Commands/GameCommandMoveLegTelemetry.cs
- Review telemetry fields used to infer stack-target context.
- Additive expansion allowed if required for support-object correctness.

5. Storyboard.GameEngine/GameManager/GameManager.cs
- Review item_stacked payload emission path and stack-target derivation.
- Ensure support-object fields map correctly for move-driven stacking outcomes.

6. Storyboard.GameEngine/Config/action-payload.manifest.json
- Confirm pilot action output catalog remains canonical and aligned to runtime emission.

Known coupling checkpoint:
1. Storyboard.GameEngine/GameServices/References/RuntimeActionVariableDescriptorRegistry.cs
- Verified and intentionally preserved for this stage.
- Rationale: Wave 1 parity requires MoveRoomObjectByPoints output-token semantics to match MoveRoomObjectOnGrid exactly while broad action-catalog retirement remains out of scope.

## Execution Checklist (Stage 02)

1. Confirm baseline runtime outputs for MoveRoomObjectOnGrid and MoveRoomObjectByPoints against manifest keys.
2. Implement additive runtime changes needed for correctness gaps discovered in stack-target support-object reporting.
3. Keep legacy compatibility behavior while introducing canonical mapping.
4. Add or update focused regression tests around:
- Move action output token availability.
- Move-driven stack landing target correctness.
- Replay behavior stability.
5. Record before/after token and payload behavior in this handoff.

## Validation Commands Executed

1. dotnet build .\StoryboardDesigner.slnx
- Result: PASS.

2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "EventSubscription|VariableChoices"
- Result: PASS.
- Summary: Failed 0, Passed 25, Skipped 0.

3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests|SchemaCodegenHardcodedDtoGuardrailsTests|SchemaEmittedContractDriftGuardrailsTests|TransportArtifactGuardrailsTests"
- Result: PASS.
- Summary: Failed 0, Passed 67, Skipped 0.

4. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
- Result: PASS.
- Summary: Failed 0, Passed 9, Skipped 0.

Additional focused stage evidence:
1. dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~RuntimeMoveRoomObjectOnGridActionTests|FullyQualifiedName~ActionVariableEmissionGuardrailTests"
- Result: PASS.
- Summary: Failed 0, Passed 45.

2. dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~RuntimeEventPublicationFlowTests"
- Result: PASS.
- Summary: Failed 0, Passed 6.

## Test Results

1. All required Stage 02 validation gates passed.
2. Focused runtime pilot tests passed, including new support-context telemetry assertions and MoveRoomObjectByPoints parity companion coverage.

## Behavioral Notes

1. Move-driven stack event context now resolves stack target from landing support object, preventing moved-object misclassification as stack target.
2. MoveRoomObjectByPoints remains descriptor-coupled to MoveRoomObjectOnGrid by intent for Wave 1 parity.
3. Designer-wide picker unification remains out of scope for this stage.

## Known Issues/Risks

1. Risk: runtime changes diverge from manifest if emission and catalog are updated in different passes.
2. Mitigation: require per-PR parity review against action-payload manifest entries.
3. Residual risk: event payload fields `stackDirection` and `stackDistance` for move-driven stack publication remain sourced outside this stage's new support-context path.
4. Mitigation: evaluate explicit population policy during Stage 05 Wave 1 evidence packet and Stage 07 cleanup planning.

## Boundary Compliance Report

1. Out-of-scope reads performed:
- Storyboard.GameEngine/Config/action-payload.manifest.json inspected for runtime alignment checks.
- StoryboardDesigner.App/ and StoryboardDesigner.App.Tests/ inspected read-only per stage allowlist.
2. Out-of-scope edits performed:
- None.
3. Stage-boundary exceptions approved:
- None.
4. Session context scope notes:
- Stage remained runtime-focused; no Stage 03+ provider unification or Stage 05+ broad wave migration was started.

## Explicit Next-Stage Start Checklist

1. Confirm Stage 02 runtime pilot parity is green and documented here.
2. Use Stage 02 outputs to drive Stage 03 shared-provider and Stage 04 designer picker behavior with stable runtime truth.
3. Maintain action-type-scoped retirement planning in Stage 07; do not perform global removals prematurely.
