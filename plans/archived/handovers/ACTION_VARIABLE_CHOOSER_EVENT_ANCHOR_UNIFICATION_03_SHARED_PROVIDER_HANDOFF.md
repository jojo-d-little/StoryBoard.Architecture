# Action Variable Chooser Event Anchor Unification - Stage 03 Shared Provider And Reader Unification Handoff

Status: Completed (reopened extension complete)
Stage: 3 of 7
Date: 2026-09-12
Owner Session: GitHub Copilot (GPT-5.3-Codex)

## Opening Prompt (Use To Start This Stage)

Start Stage 03 of [plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md](plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md).
Complete only shared provider/reader unification with no behavior drift.
Do not begin Stage 04 designer picker rollout or Stage 07 retirement in this stage.
Honor stage boundaries from the main plan and record validation evidence.

## Stage Boundary Allowlist Snapshot (From Main Plan)

Default deny rule:
1. Any path not explicitly listed in allowed read/edit scope is out of scope for this stage.

Edit-implies-read rule:
1. Any path in allowed edit scope is automatically readable.
2. Allowed read scope should list only extra read-only dependencies.

1. Allowed read scope:
- plans/**
- Storyboard.GameEngine/**
- StoryboardDesigner.App/**
- StoryboardDesigner.App.Tests/**

2. Allowed edit scope:
- Storyboard.GameEngine/**
- Storyboard.GameEngine.Tests/**
- StoryboardDesigner.App/**
- StoryboardDesigner.App.Tests/**
- plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md
- plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_03_SHARED_PROVIDER_HANDOFF.md

## Scope Completed

1. Added a shared session-anchor manifest reader in runtime shared services.
2. Added a shared session-anchor data provider to centralize anchor metadata and resolved reference-value access.
3. Routed runtime session-anchor consumers to the shared provider path without changing command grammar behavior.
4. Routed designer-side anchor metadata readers to use the shared runtime manifest reader.
5. Added focused tests for manifest reader/provider behavior and compatibility normalization.
6. Preserved behavior by adding compatibility anchor-root recognition required by event payload source-path resolution.

## Reopen Note (2026-09-12)

1. Stage 03 is reopened to promote `self` from alias-only behavior to a first-class runtime anchor.
2. Objective: keep `self.*` support while removing special-case resolution ownership from script alias plumbing.
3. Direction: resolve `self` through shared session-anchor provider (`TryResolveSingle`) using current action scope context.
4. Stage 04 remains blocked from closeout until this Stage 03 extension is validated and recorded.

## Reopen Extension Outcome (2026-09-12)

1. Added `self` as an explicit anchor key in `session-anchordata.manifest.json`.
2. Added runtime provider support so `self` resolves from current action scope node.
3. Updated action-script resolution flow to prefer shared provider/cache lookup for `self` rather than self-prefix special casing.
4. Added provider tests verifying `self` known-key behavior and action-scope resolution semantics.
5. Validation pass completed; Stage 03 extension is now closed and Stage 04 can proceed.

## Files Changed

1. Storyboard.GameEngine/GameServices/References/RuntimeSessionAnchorManifestReader.cs
- Added shared manifest loading and parsing logic.

2. Storyboard.GameEngine/GameServices/References/RuntimeSessionAnchorDataProvider.cs
- Added centralized anchor metadata and session reference-value access.
- Added compatibility anchor-root recognition for payload source paths.

3. Storyboard.GameEngine/GameServices/References/RuntimeSessionAnchorManifestData.cs
- Split parsed manifest data type into dedicated file for architecture guardrail compliance.

4. Storyboard.GameEngine/GameServices/References/RuntimeSessionAnchorManifestAnchor.cs
- Split anchor record into dedicated file for architecture guardrail compliance.

5. Storyboard.GameEngine/GameServices/References/RuntimeSessionAnchorManifestSubProperty.cs
- Split subproperty record into dedicated file for architecture guardrail compliance.

6. Storyboard.GameEngine/GameServices/References/RuntimeSessionAnchorValueProvider.cs
- Routed through shared session-anchor data provider.

7. Storyboard.GameEngine/GameServices/Events/RuntimeEventPayloadBuilder.cs
- Replaced local anchor-root handling with shared provider metadata and normalization.

8. Storyboard.GameEngine/GameServices/Commands/GameCommandProcessorService.cs
- Routed invocation and variant-selection reference values through shared session-anchor provider.

9. Storyboard.GameEngine/GameServices/Events/RuntimeEventSubscriptionDispatcher.cs
- Routed resolved reference values through shared session-anchor provider.

10. Storyboard.GameEngine/GameServices/Timers/RuntimeTimerDispatchLoop.cs
- Routed resolved reference values through shared session-anchor provider.

11. StoryboardDesigner.App/Validation/Rules/Project/EventSubscriptionRuleSupport.cs
- Replaced local session-anchor manifest parsing usage with shared reader-backed flow.

12. StoryboardDesigner.App/ViewModels/MainWindowViewModel.ProjectExplorer.cs
- Replaced local session-anchor manifest parsing usage with shared reader-backed flow.

13. StoryboardDesigner.App/Views/EventSubscriptionEditorDialog.xaml.cs
- Replaced local session-anchor manifest parsing usage with shared reader-backed flow.

14. Storyboard.GameEngine.Tests/RuntimeSessionAnchorDataProviderTests.cs
- Added focused tests for shared reader/provider parsing, key ordering, and source-path normalization.

15. plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_03_SHARED_PROVIDER_HANDOFF.md
- Replaced placeholder with executed Stage 03 handoff.

## Contract/Interface Impact

1. No public contract grammar changes.
2. No producer command vocabulary changes.
3. Runtime/data-provider internal API added for shared anchor metadata/value access.

## Validation Commands Executed

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~RuntimeSessionAnchorDataProviderTests"
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "EventSubscription|VariableChoices"
4. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests|SchemaCodegenHardcodedDtoGuardrailsTests|SchemaEmittedContractDriftGuardrailsTests|TransportArtifactGuardrailsTests"
5. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"

## Test Results

1. Build: PASS.
2. RuntimeSessionAnchorDataProviderTests: PASS (4/0).
3. EventSubscription|VariableChoices: PASS (25/0).
4. Grouped regression/guardrail suite: PASS (67/0).
5. Replay_RecordedSession_OutputLinesMatchAtEachStep: PASS (9/0).

Interim failure resolved during stage execution:
1. ArchitectureSeparationGuardrailsTests initially failed due multiple top-level types in RuntimeSessionAnchorManifestReader.cs.
2. Resolved by splitting manifest data/records into dedicated files; guardrail suite then passed.

## Behavioral Notes

1. Stage 03 objective met: runtime and designer now share one manifest-reader model for session anchors.
2. Shared provider normalization keeps compatibility for legacy source-path prefix form currentRoom.activePlayer.*.
3. Compatibility anchor roots were retained to avoid behavior drift for payload mapping roots not represented in the session-anchor manifest.

## Known Issues/Risks

1. None identified after full Stage 03 validation pass.

## Boundary Compliance Report

1. Out-of-scope reads performed:
- None.
2. Out-of-scope edits performed:
- None.
3. Stage-boundary exceptions approved:
- None.
4. Session context scope notes:
- Stage 04 not started.

## Explicit Next-Stage Start Checklist

1. Manual review and sign-off for Stage 03 completed handoff.
2. Start Stage 04 only after explicit go-ahead.
