# CommandAction Payload Contract Hard Switch Plan

Status: Closed
Date: 2026-08-08
Owner: Copilot + user

## Closeout Summary

Completed:

1. CommandAction contract switched to payload-centered shape with root common fields only.
2. Action-specific flattened root fields removed from CommandAction schema and emitted DTO contracts.
3. LinkedActions hard cut completed: linked flow now payload-owned; envelope linkedActions removed.
4. Runtime/designer mapping updated to payload-first and hard-cut behavior.
5. Runtime + designer + enum lock workflows completed and drift guardrail brought green.
6. Sample runtime exports regenerated from authored sample sources to remove legacy flattened runtime fixtures.
7. Final cleanup completed, including removal of temporary RuntimeCommandActionDto legacy compatibility shim.

Validation status at close:

1. dotnet build .\StoryboardDesigner.slnx: PASS
2. SchemaEmittedContractDriftGuardrailsTests filter: PASS
3. Runtime-focused guardrail suite (includes playback/architecture filters): PASS

Residual notes:

1. Non-blocking nullable warnings remain in generated designer contract DTOs and are outside the hard-switch closure scope.

## Decision

We will replace the flattened CommandAction contract bag with a discriminated payload contract model.

- No backward-compatibility bridge in runtime DTO contracts.
- No dual-write period for old flat action fields.
- Existing sample projects will be migrated to the new shape.

## Problem Statement

Current action contracts in [Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Core/CommandAction.core.schema.json](Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Core/CommandAction.core.schema.json) contain many action-specific fields in a single object.

This diverges from existing runtime/designer code organization where each action kind has dedicated payload models, e.g. Synonym:

- Designer payload: [StoryboardDesigner.App/Models/Actions/ActionPayloads/SynonymPayload.cs](StoryboardDesigner.App/Models/Actions/ActionPayloads/SynonymPayload.cs)
- Runtime payload: [Storyboard.Shared/GameServices/Actions/GameActions/RuntimeActionPayload/RuntimeSynonymActionPayload.cs](Storyboard.Shared/GameServices/Actions/GameActions/RuntimeActionPayload/RuntimeSynonymActionPayload.cs)

## Target Contract Shape

CommandAction becomes:

- Common fields only (id, name, actionType, verbs/noVerbLinkage, direction, outcomeMessageMap, forwarding fields).
- A payload child property, action-specific by actionType.

High-level JSON shape:

- id
- name
- actionType
- shared/common action metadata
- payload: action-specific object

## Generator Constraint and Required Upgrade

Current schema codegen in [Storyboard.SchemaCodegen/Program.cs](Storyboard.SchemaCodegen/Program.cs) flattens allOf object properties and does not generate typed discriminated unions from oneOf + actionType.

Hard switch requires generator changes to support:

1. Payload oneOf references in schema.
2. Lane-specific payload type hints for runtime/designer DTOs.
3. Optional discriminator metadata that can be consumed by mapper code (actionType -> payload type).

## Implementation Plan

### Phase 1 - Schema and DTO Structure Cutover

1. Create payload core schemas per action kind under DtoContracts/Core/ActionPayloads.
2. Create runtime payload DTO wrapper schemas under DtoContracts/Runtime/ActionPayloads.
3. Create designer payload DTO wrapper schemas under DtoContracts/Designer/ActionPayloads.
4. Replace action-specific top-level fields in CommandAction core schema with a payload property.
5. Keep only common fields at CommandAction root.

### Phase 2 - Schema Codegen Enhancements

1. Update codegen to infer payload type from referenced schema title and lane.
2. Add support for oneOf payload schema branches where each branch can carry lane-specific type hint.
3. Ensure generated DTO property types are concrete runtime/designer payload DTO types, not object.
4. Preserve existing enum/type behavior and contract lock guardrails.

### Phase 3 - Runtime and Designer Mapping Updates

1. Update designer contract mapping in [StoryboardDesigner.App/Services/JsonExportService.cs](StoryboardDesigner.App/Services/JsonExportService.cs) to map between CommandAction payload models and contract payload DTOs.
2. Update runtime bootstrap mapping in [Storyboard.Shared/GameServices/Bootstrap/CleanRuntimeBootstrapSnapshotMapper.cs](Storyboard.Shared/GameServices/Bootstrap/CleanRuntimeBootstrapSnapshotMapper.cs) to map contract payload DTOs to Runtime*ActionPayload classes.
3. Update runtime action payload accessors and executor call sites only where field access currently assumes flattened action DTO members.

### Phase 4 - Sample Migration

1. Migrate sample authored/exported action JSON to new payload contract shape.
2. Validate sample load and playback flows.

### Phase 5 - Lock and Validation

1. Regenerate runtime/designer/enum staging.
2. Lock runtime/designer/enum artifacts.
3. Run:
   - dotnet build .\StoryboardDesigner.slnx
   - dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~SchemaEmittedContractDriftGuardrailsTests"
   - dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
   - dotnet test .\StoryboardDesigner.slnx

## Hard-Switch Scope Clarifications

In scope:

- CommandAction contract shape redesign.
- Codegen support required for typed action payload child DTOs.
- Runtime + designer mapping updates to new contract shape.
- Sample migration to keep smoke/regression fixtures valid.

Out of scope:

- Maintaining deserialization compatibility for old flat action payload fields.
- Transitional feature flags or dual contract mode.

## Risk Notes

1. Generator change can affect non-action schemas if not scoped carefully.
2. Contract lock guardrails will fail until regen/lock done after each schema/codegen slice.
3. Large enum breadth means payload schema coverage must be complete before final cutover.

## Execution Order Recommendation

1. First vertical slice: Synonym action payload end-to-end.
2. Then broad payload schema creation for all action types.
3. Then final CommandAction core contract switch.

This reduces debugging blast radius while keeping final state hard-switch.
