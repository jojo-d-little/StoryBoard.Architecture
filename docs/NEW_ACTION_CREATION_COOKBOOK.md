# New Action Creation Cookbook

Status: Solution-level implementation guide (2026-09-03).

Purpose: provide a repeatable, low-risk process for adding a new CommandActionType across runtime contracts, runtime execution, designer authoring UX, persistence/export, and regression coverage.

Audience: developers working in Storyboard.Shared.Contracts, Storyboard.GameEngine, StoryboardDesigner.App, Storyboard.Simulator, and test projects.

## Scope And Principles

1. Keep Storyboard.Simulator independent from StoryboardDesigner.App.
2. Put reusable runtime behavior in Storyboard.GameEngine and shared contracts in Storyboard.Shared.Contracts.
3. Keep designer-only editing UX and payload authoring behavior in StoryboardDesigner.App.
4. Treat clean export as external contract data and preserve compatibility.
5. Prefer additive changes over rewrites.

## TLDR Checklist

Use this when you need the shortest safe path.

1. Add enum value in shared action type contract.
2. Add or reuse runtime payload shape on RuntimeCommandActionDescriptor.
3. Wire RuntimeCommandActionExecutor to concrete executable path.
4. Register result codes/tokens in RuntimeActionResultCodeRegistry.
5. Register action variable descriptor mapping in RuntimeActionVariableDescriptorRegistry.
6. Register designer echo token provider in ActionEchoReferenceTokenProviderRegistry.
7. Add designer payload schema/model projection fields when editable.
8. Add save/load and clean export payload mapping for new fields.
9. Add runtime snapshot mapping in CleanRuntimeBootstrapSnapshotMapper.
10. Update tests for runtime behavior, persistence roundtrip, and guardrails.
11. Run build and test gates.

## Workflow Overview

1. Decide action visibility.
2. Apply contract/schema gate first when contract shape changes are required.
3. Implement Shared runtime contract and execution.
4. Implement designer authoring pipeline if visible/editable.
5. Implement persistence and mapping.
6. Validate with focused and full tests.

## Contract Schema Gate (Run Before Runtime Changes)

Use this gate whenever a new action requires contract or enum shape changes.

1. Update schema source (not generated contract outputs).
2. Regenerate contract outputs with codegen.
3. Stop and request lock/approval before dependent implementation changes.
4. Only continue runtime/designer behavior wiring after lock approval.

Notes:

1. Runtime contract files under Storyboard.Shared.Contracts/RuntimeContracts/Dtos ending in _contract.cs are generated artifacts.
2. Do not hand-edit generated _contract.cs files as a shortcut.

## 1. Decide Action Visibility

Choose one path before coding:

1. Runtime-only action.
2. Runtime + designer-visible action type.
3. Runtime + designer-editable action with custom fields.

Use this decision to avoid partial wiring.

## 2. Runtime Slice (Always Required)

### 2.1 Action Type Contract

1. Add new value to action type schema/enum source and regenerate contracts.
2. Confirm any DTO that serializes action type supports the new value.

Common file locations:

1. [Storyboard.Shared.Contracts/RuntimeContracts/Enums/CommandActionType.cs](Storyboard.Shared.Contracts/RuntimeContracts/Enums/CommandActionType.cs)
2. [Storyboard.Shared.Contracts/RuntimeContracts/Dtos/RuntimeCommandActionDto_contract.cs](Storyboard.Shared.Contracts/RuntimeContracts/Dtos/RuntimeCommandActionDto_contract.cs)

### 2.2 Runtime Descriptor Payload

1. Add typed runtime payload record/class.
2. Add payload slot on RuntimeCommandActionDescriptor.
3. Add payload accessor helper if needed.

Common file locations:

1. [Storyboard.GameEngine/GameServices/Actions](Storyboard.GameEngine/GameServices/Actions)
2. [Storyboard.GameEngine/GameServices/Actions/RuntimeCommandActionDescriptor.cs](Storyboard.GameEngine/GameServices/Actions/RuntimeCommandActionDescriptor.cs)
3. [Storyboard.GameEngine/GameServices/Actions/RuntimeActionPayloadAccessors.cs](Storyboard.GameEngine/GameServices/Actions/RuntimeActionPayloadAccessors.cs)

### 2.3 Executor Wiring

1. Add executable action implementation.
2. Route new type in RuntimeCommandActionExecutor switch.
3. Add result code enum and registry descriptor entries.
4. If the action reuses an existing executable flow, still map it in registry and executor-supported lists.

Common file locations:

1. [Storyboard.GameEngine/GameServices/Actions/RuntimeCommandActionExecutor.cs](Storyboard.GameEngine/GameServices/Actions/RuntimeCommandActionExecutor.cs)
2. [Storyboard.GameEngine/GameServices/Actions/RuntimeActionResultCodeRegistry.cs](Storyboard.GameEngine/GameServices/Actions/RuntimeActionResultCodeRegistry.cs)
3. [Storyboard.GameEngine/GameServices/Actions/GameActions/RuntimeActionExecutable](Storyboard.GameEngine/GameServices/Actions/GameActions/RuntimeActionExecutable)

### 2.4 Action Variable Descriptor Wiring

If action supports echo tokens:

1. Register descriptor mapping in RuntimeActionVariableDescriptorRegistry.
2. Reuse an existing descriptor when action semantics are aliases of an existing action type.

Common file location:

1. [Storyboard.GameEngine/GameServices/References/RuntimeActionVariableDescriptorRegistry.cs](Storyboard.GameEngine/GameServices/References/RuntimeActionVariableDescriptorRegistry.cs)

### 2.5 Runtime Mutation Or Session Extensions

If action mutates world/session data:

1. Extend runtime mutation gateway interface.
2. Implement gateway forwarding in command processor adapter.
3. Implement session mutation logic.

Common file locations:

1. [Storyboard.GameEngine/GameServices/RuntimeContext/IRuntimeScopeMutationGateway.cs](Storyboard.GameEngine/GameServices/RuntimeContext/IRuntimeScopeMutationGateway.cs)
2. [Storyboard.GameEngine/GameServices/Commands/GameCommandProcessorService.cs](Storyboard.GameEngine/GameServices/Commands/GameCommandProcessorService.cs)
3. [Storyboard.GameEngine/GameStateData/GameStateSession.cs](Storyboard.GameEngine/GameStateData/GameStateSession.cs)

## 3. Designer Slice (When Visible Or Editable)

### 3.1 Action Type Availability In UI

If action should appear in action-type pickers:

1. Add it to designer action-type value lists.
2. Confirm any linked-flow allowlist includes it when appropriate.
3. Ensure ActionEchoReferenceTokenProviderRegistry has a provider registration (or intentional runtime-only exemption in tests).

Common file location:

1. [StoryboardDesigner.App/ViewModels/CommandActionTypeValues.cs](StoryboardDesigner.App/ViewModels/CommandActionTypeValues.cs)
2. [StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviderRegistry.cs](StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviderRegistry.cs)

### 3.2 Payload Model And Projection

For editable custom fields:

1. Create designer payload model.
2. Register payload in ActionPayloadSchemaMap.
3. Extend CommandAction payload coordinator clone/default/get-set methods.
4. Add projection property on CommandAction partial projection file.
5. Add helper methods in ActionPayloadAccessors.

Common file locations:

1. [StoryboardDesigner.App/Models/Actions/ActionPayloads](StoryboardDesigner.App/Models/Actions/ActionPayloads)
2. [StoryboardDesigner.App/Models/Actions/ActionPayloads/ActionPayloadSchemaMap.cs](StoryboardDesigner.App/Models/Actions/ActionPayloads/ActionPayloadSchemaMap.cs)
3. [StoryboardDesigner.App/Models/Actions/CommandActionPayloadCoordinator.cs](StoryboardDesigner.App/Models/Actions/CommandActionPayloadCoordinator.cs)
4. [StoryboardDesigner.App/Models/Actions/CommandAction.PayloadProjections.cs](StoryboardDesigner.App/Models/Actions/CommandAction.PayloadProjections.cs)

### 3.3 Authoring Dialog UX

For custom input/browse interactions:

1. Add panel to RoomActionEditorDialog XAML.
2. Add visibility switching in code-behind.
3. Add choose/browse handlers and save-time validation.
4. Add dedicated chooser dialog if selection requires filtering.

Common file locations:

1. [StoryboardDesigner.App/Views/RoomActionEditorDialog.xaml](StoryboardDesigner.App/Views/RoomActionEditorDialog.xaml)
2. [StoryboardDesigner.App/Views/RoomActionEditorDialog.xaml.cs](StoryboardDesigner.App/Views/RoomActionEditorDialog.xaml.cs)
3. [StoryboardDesigner.App/Views](StoryboardDesigner.App/Views)

### 3.4 Dialog Workflow Threading

If editor needs scoped candidate lists:

1. Add candidate choice model.
2. Add scope-based candidate generation in MainWindowViewModel.
3. Thread candidate list through workflow service interface and implementations.
4. Pass through ScopedActionsDialog and LinkedActionTreeEditorDialog constructors.

Common file locations:

1. [StoryboardDesigner.App/ViewModels/MainWindowViewModel.cs](StoryboardDesigner.App/ViewModels/MainWindowViewModel.cs)
2. [StoryboardDesigner.App/ViewModels/MainWindowViewModel.DialogWorkflows.cs](StoryboardDesigner.App/ViewModels/MainWindowViewModel.DialogWorkflows.cs)
3. [StoryboardDesigner.App/Services/IMainWindowDialogWorkflowService.cs](StoryboardDesigner.App/Services/IMainWindowDialogWorkflowService.cs)
4. [StoryboardDesigner.App/Services/MainWindowDialogWorkflowService.cs](StoryboardDesigner.App/Services/MainWindowDialogWorkflowService.cs)
5. [StoryboardDesigner.App/Views/ScopedActionsDialog.xaml.cs](StoryboardDesigner.App/Views/ScopedActionsDialog.xaml.cs)
6. [StoryboardDesigner.App/Views/LinkedActionTreeEditorDialog.xaml.cs](StoryboardDesigner.App/Views/LinkedActionTreeEditorDialog.xaml.cs)

## 4. Persistence And Mapping Slice

### 4.1 Native Project Save/Load

1. Add new action field to JSON DTOs.
2. Map model to DTO for save.
3. Map DTO to model for load.

Common file locations:

1. [StoryboardDesigner.App/Services/JsonExportService.cs](StoryboardDesigner.App/Services/JsonExportService.cs)

### 4.2 Clean Export Contract

1. Map field to clean room action DTO when applicable.
2. Confirm defaults/null behavior preserves compatibility.

Common file locations:

1. [StoryboardDesigner.App/Services/JsonExportService.cs](StoryboardDesigner.App/Services/JsonExportService.cs)
2. [Storyboard.Shared.Contracts/RuntimeContracts/Dtos/RuntimeCommandActionDto_contract.cs](Storyboard.Shared.Contracts/RuntimeContracts/Dtos/RuntimeCommandActionDto_contract.cs)

### 4.3 Runtime Snapshot Mapping

1. Map designer model action to runtime descriptor typed payload.
2. Preserve field through any normalization rewrite paths.

Common file location:

1. [Storyboard.GameEngine/GameServices/Bootstrap/CleanRuntimeBootstrapSnapshotMapper.cs](Storyboard.GameEngine/GameServices/Bootstrap/CleanRuntimeBootstrapSnapshotMapper.cs)

## 5. Test Coverage Expectations

Minimum expected coverage:

1. Runtime behavior unit test for action execution semantics.
2. Runtime command processor integration-style test for end-to-end invocation.
3. Persistence roundtrip test for new payload fields.
4. Mapper test or coverage in existing mapper-focused suite.
5. Guardrail/registry test updates when enum growth affects strict assertions.

Common test locations:

1. [StoryboardDesigner.App.Tests](StoryboardDesigner.App.Tests)
2. [Storyboard.GameEngine.Tests](Storyboard.GameEngine.Tests)
3. [StoryboardDesigner.App.Tests/ArchitectureSeparationGuardrailsTests.cs](StoryboardDesigner.App.Tests/ArchitectureSeparationGuardrailsTests.cs)
4. [StoryboardDesigner.App.Tests/JsonExportServiceProjectStateTests.cs](StoryboardDesigner.App.Tests/JsonExportServiceProjectStateTests.cs)

## 6. Validation Gates

Run from solution root:

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
4. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"

## 7. Common Pitfalls

1. Action appears in runtime but not designer dropdown because CommandActionTypeValues was not updated.
2. Build fails due to strict provider/enum tests when ActionEchoReferenceTokenProviderRegistry or RuntimeActionVariableDescriptorRegistry was not updated.
3. Field edits appear in UI but do not persist because JsonExportService mappings were not updated both directions.
4. Field persists but has no runtime effect because CleanRuntimeBootstrapSnapshotMapper did not map typed payload.
5. Normalization code rewrites RuntimeCommandActionDescriptor and accidentally drops new payload property.
6. WPF build can fail with MSB3026/MSB3027 when StoryboardDesigner.App.exe is running and locking output.

## 8. Definition Of Done

An action addition is complete when all are true:

1. Shared runtime executes the action through typed payloads and result codes.
2. Designer visibility/editing matches intended product scope.
3. Action fields roundtrip save/load and clean export as expected.
4. Runtime snapshot mapper carries the action payload end-to-end.
5. Result code registry, variable descriptor registry, and designer echo token provider registry are all updated or intentionally exempted.
6. Required test slices are green.
7. Full build is green.

## 9. Suggested PR Checklist Template

1. Action type enum and DTO updates completed.
2. Runtime descriptor payload and executor wiring completed.
3. RuntimeActionResultCodeRegistry and RuntimeActionVariableDescriptorRegistry updates completed.
4. Designer ActionEchoReferenceTokenProviderRegistry updates completed.
5. Designer payload schema/projections/accessors completed.
6. Editor and workflow candidate threading completed (if editable).
7. Json export/load and clean export mapping completed.
8. Runtime snapshot mapping completed.
9. Tests added or updated and all validation commands passed.
