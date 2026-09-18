# Designer DTO One-Class Cadence Plan

Status: Active
Owner: Designer DTO contract/authored migration workflow
Last updated: 2026-08-06

## Purpose

Enforce a strict, repeatable, one-class-at-a-time workflow for remaining designer DTO migrations while preventing scope creep.

## Non-Negotiable Guardrails

1. One class only per cycle.
2. Do not bring any other class into implementation scope during that cycle.
3. If another class is discovered, log it for later and continue with the current class only.
4. Stop at the defined review checkpoints.
5. When a class is split into authored/contract partials, place those files under `StoryboardDesigner.App/Services/DesignerJsonContracts/` as the canonical location.

## Cadence (Per Class)

1. Select one class only.
2. Identify and confirm the next class to attack.
3. Split the existing class into contract/authored partials.
	- Ensure the resulting partial files are located in `StoryboardDesigner.App/Services/DesignerJsonContracts/`.
4. Review class members against current schema and classify what is common/contract vs distinct/authored.
5. Split schema into three parts: Core, Designer, Runtime.
6. Stop for manual review.
7. After review and manual adjustments, build and run tests.
8. Allow manual codegen and lock.
9. Rebuild and address corrections.
10. Build and run tests again.

## Completion Checklist (Per Class)

1. Partial split complete and compile-safe.
2. 3-way schema split complete (Core/Designer/Runtime).
3. Manual review done.
4. Manual codegen/lock done.
5. Post-lock corrections applied.
6. Final build green.
7. Final tests green.

## Scope Creep Handling

1. Capture out-of-scope findings in a follow-up queue.
2. Do not fix out-of-scope items in the current class cycle.
3. Start a new cycle for the next class only after the current class checklist is complete.

## Current Cycle

Class: ProjectGlobalNodeDto
Checkpoint: Step 5 reached (schema split proposal drafted; waiting for manual review)

Completed this cycle so far:

1. ProjectGlobalNodeDto selected as the only in-scope class for the next cadence pass.
2. ProjectGlobalNodeDto confirmed as the next class to attack after ProjectAuthoringRootDto cycle closeout.
3. Step 3 verification complete: ProjectGlobalNodeDto contract/authored partial split is present and compile-safe in `StoryboardDesigner.App/Services/DesignerJsonContracts/`.
4. Step 4 classification drafted against RuntimeProjectDto and scope-base peers.
5. Step 5 schema split implementation drafted in-schema (Core/Designer/Runtime) for manual review before generation/lock.

Pending after manual review:

1. Step 6 manual review checkpoint and shape approval.
2. Step 7-10 execution after review (build/tests, manual codegen+lock, post-lock corrections, final validation).

ProjectGlobalNodeDto Step 4 Classification (2026-08-06):

1. Core candidate (shared semantics with runtime scope/project contracts):
2. Scope-base global primitives from inherited base: ScopeKind, BaseObjectIds, GameObjectIds, GameProperties, AdditionalVerbs, AdditionalDirectionals, AdditionalDirectionalTraversalMappings, AvailableGameActions.
3. Identity set alignment: Id/Name/NameInGame (designer) aligns near-directly with ScopeNodeId/Name/NameInGame (runtime).
4. PlanetIds aligns structurally and semantically.

1. Requires explicit semantic decision before inclusion in Core:
2. ProcedureIds currently exists in both lanes but semantics are not yet fully locked as identical (designer ownership list vs runtime root export index); keep explicitly reviewed in Step 5 schema proposal.

1. Designer-only members (keep in designer wrapper unless separately approved):
2. SharedVariables.
3. ObjectTemplateIds.
4. ObjectTemplates.
5. RoomTemplateIds.
6. RoomTemplates.
7. Authored partial graph payloads: GameObjects and BaseObjects.

1. Runtime-only members/behavior (keep out of designer wrapper):
2. SchemaVersion.
3. AutoSaveSeconds.
4. RoomImageCanvasWidth.
5. RoomImageCanvasHeight.
6. RoomDesignerGridCellSize.
7. StartingPlanetName.
8. RuntimeProcedures (authored runtime-only partial state).
9. Runtime graph attachment behavior (`SetAttachedGlobalObjects`, `SetAttachedPlanets`, and corresponding getters).

ProjectGlobalNodeDto Step 5 Schema Split Implementation (Draft):

1. Core schema file created:
2. `Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Core/GlobalScopeNode.core.schema.json`
3. Content scope: only shared global-scope primitives currently aligned in both lanes.
4. Included properties: PlanetIds and ProcedureIds.

1. Designer wrapper schema file created:
2. `Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Designer/ProjectGlobalNodeDto_contract.schema.json`
3. Composition: `DesignerScopeNodeDtoBase_contract.schema.json` + `GlobalScopeNode.core.schema.json` + designer-only extension block.
4. Designer-only extension block includes: SharedVariables, ObjectTemplateIds, ObjectTemplates, RoomTemplateIds, RoomTemplates.
5. Existing authored-only graph payload properties retained in wrapper schema with `x-csharp-emit-property: false`: GameObjects, BaseObjects.

1. Runtime wrapper updated in schema:
2. `Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Runtime/RuntimeProjectDto_contract.schema.json` now composes in `GlobalScopeNode.core.schema.json`.
3. Keep runtime-only properties local to RuntimeProjectDto wrapper: SchemaVersion, AutoSaveSeconds, RoomImageCanvasWidth, RoomImageCanvasHeight, RoomDesignerGridCellSize, StartingPlanetName.

1. ProcedureIds placement decision:
2. ProcedureIds moved to `ScopeNodeBase.core.schema.json` so ProjectGlobalNodeDto and peers inherit it from the base schema layer.

Previous completed cycle notes (ProjectAuthoringRootDto):

1. ProjectAuthoringRootDto completed through Steps 1-10.
2. Post-lock correction applied for removed runtime contract property assignment (`RuntimeProjectDto.Procedures`) in `JsonExportService`.
3. Validation after correction is green:
4. `dotnet build .\\StoryboardDesigner.slnx` passed.
5. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj` passed (741/741).
6. Playback regression gate passed.

Previous cycle note (RoomDto):

1. RoomDto reached Step 6 and is paused for manual review with known command/image follow-up notes.

Previous cycle note (RoomCommandDto):

1. RoomCommandDto reached Step 6 and is paused for manual review with known designer/runtime shape divergence follow-up.

Previous completed cycle notes (SharedVariableDefinitionDto):

1. SharedVariableDefinitionDto completed through Steps 1-10.
2. Post-lock rebuild and tests passed (729/729, playback gate passed).

Step 7 notes (SharedVariableDefinitionDto):

1. `dotnet build .\StoryboardDesigner.slnx` passed.
2. Playback regression gate passed.
3. Full `StoryboardDesigner.App.Tests` passed (729/729).

Step 9/10 notes (SharedVariableDefinitionDto):

1. Manual codegen and lock completed.
2. SharedVariableDefinition and SharedVariableParticipant designer contract partials remained in sync with locked output.
3. Post-lock `dotnet build .\StoryboardDesigner.slnx` passed.
4. Playback regression gate passed.
5. Full `StoryboardDesigner.App.Tests` passed (729/729).

Previous completed cycle notes (RoomPlacementDto):

1. RoomPlacementDto completed through Steps 1-10.
2. Lock reported exact match after generation; no post-lock correction was needed.
3. Validation status remained green (solution build and app tests).

Previous completed cycle notes (ProjectCommandActionReferenceDto):

1. ProjectCommandActionReferenceDto completed through Steps 1-10.
2. Post-lock rebuild and tests passed (729/729, playback gate passed).

Step 7 notes (ProjectCommandActionReferenceDto):

1. `dotnet build .\StoryboardDesigner.slnx` passed.
2. Playback regression gate passed.
3. Full `StoryboardDesigner.App.Tests` passed (729/729).

Step 9/10 notes (ProjectCommandActionReferenceDto):

1. Manual codegen and lock completed.
2. Regenerated contract shape for `ProjectCommandActionReferenceDto` remained compile-safe with authored/contract partial split.
3. Post-lock `dotnet build .\StoryboardDesigner.slnx` passed.
4. Playback regression gate passed.
5. Full `StoryboardDesigner.App.Tests` passed (729/729).

Previous completed cycle notes (CommandActionDto):

1. CommandActionDto completed through Steps 1-10.
2. Post-lock rebuild and tests passed (729/729, playback gate passed).

Step 7 notes (CommandActionDto):

1. `dotnet build .\StoryboardDesigner.slnx` passed.
2. Playback regression gate passed.
3. Full `StoryboardDesigner.App.Tests` has 1 expected pre-lock guardrail failure:
	`SchemaEmittedContractDriftGuardrailsTests.LockedRuntimeContractDtos_MatchSchemaEmitterOutput_AndContainGeneratedHeader`
	(lock expects legacy schema path `RuntimeCommandActionDto_contract.schema.json`, which was moved during split).

Step 9/10 notes (CommandActionDto):

1. Manual codegen and lock completed.
2. Post-lock compile fix applied in `JsonExportService` to default nullable `SimilarChildDispatchMode` to `SingleMatchingChild` during model mapping.
3. `dotnet build .\StoryboardDesigner.slnx` passed.
4. Playback regression gate passed.
5. Full `StoryboardDesigner.App.Tests` passed (729/729).

Previous completed cycle notes (ProjectRoomLinkDto):

1. ProjectRoomLinkDto completed through Steps 1-10.
2. Post-lock rebuild and tests passed (729/729, playback gate passed).

Previous completed cycle notes (ProjectObjectImageVariantDto):

1. ProjectObjectImageVariantDto completed through Steps 1-10.
2. Post-lock rebuild and tests passed (729/729, playback gate passed).

Previous completed cycle notes (ProjectLockParticipantVariableRequirementDto):

1. ProjectLockParticipantVariableRequirementDto completed through Steps 1-10.
2. Post-lock rebuild and tests passed (729/729, playback gate passed).

Previous completed cycle notes (ProjectLockOperationRequirementsDto):

1. ProjectLockOperationRequirementsDto completed through Steps 1-10.
2. Post-lock rebuild and tests passed (729/729, playback gate passed).

Previous completed cycle notes (ProjectLockKeyRequirementDto):

1. ProjectLockKeyRequirementDto completed through Steps 1-10.
2. Post-lock rebuild and tests passed (729/729, playback gate passed).

Previous completed cycle notes (CompositePartRequirementDto):

1. CompositePartRequirementDto completed through Steps 1-10.
2. Post-lock rebuild and tests passed (729/729, playback gate passed).

Previous completed cycle notes (ProjectGameObjectAppearanceDto):

1. ProjectGameObjectAppearanceDto completed through Steps 1-10.
2. Post-lock rebuild and tests passed (729/729, playback gate passed).

Previous cycle notes (ProjectCompositeRecipeDto):

1. ProjectCompositeRecipeDto reached Step 8 and is waiting on manual codegen/lock follow-through.

Previous completed cycle notes (ProcedureParticipantRequirementDto):

1. ProcedureParticipantRequirementDto completed through Steps 1-10.
2. Post-lock rebuild and tests passed (729/729, playback gate passed).

Previous completed cycle notes (ProcedureParticipantMutationDto):

1. ProcedureParticipantMutationDto completed through Steps 1-10.
2. Post-lock rebuild and tests passed (729/729, playback gate passed).

Step 7 notes (ProjectGameObjectDto):

1. dotnet build .\StoryboardDesigner.slnx passed.
2. Playback regression test gate passed.
3. Full StoryboardDesigner.App.Tests run has 1 expected guardrail failure before lock-path alignment:
	SchemaEmittedContractDriftGuardrailsTests.LockedRuntimeContractDtos_MatchSchemaEmitterOutput_AndContainGeneratedHeader
	(missing legacy path RuntimeGameObjectDto_contract.schema.json).

Previous cycle note (ProjectGameObjectDto):

1. ProjectGameObjectDto reached Step 8 and is still waiting on manual codegen/lock follow-through.

Previous completed cycle notes (ProcedureDefinitionDto):

1. ProcedureDefinitionDto completed through Steps 1-10.
2. Post-lock rebuild and tests passed (729/729, playback gate passed).

Previous completed cycle notes (AreaDto):

1. AreaDto completed through Steps 1-10.
2. Post-lock rebuild and tests passed (729/729, playback gate passed).

Previous completed cycle notes (CountryDto):

1. CountryDto completed through Steps 1-10.
2. Post-lock rebuild and tests passed (729/729, playback gate passed).

Previous completed cycle notes (PlanetDto):

1. PlanetDto completed through Steps 1-10.
2. Post-lock rebuild and tests passed (729/729, playback gate passed).

## Deferred Follow-Up (Post-Lock Campaign)

1. Revisit CommandActionDto split after the remaining class-lock cadence is complete.
2. Audit Core vs Designer vs Runtime ownership for move/rotate/stack payload fields (for example moveDistanceInCells, moveDirectionToken, moveAllowPartialMove, moveVisualTransitionHint, rotateMode, rotateTurnDegrees, rotateFacingDirectionToken, rotateVisualTransitionHint, stackVisualTransitionHint).
3. Validate that runtime export and runtime bootstrap mapping preserve intended execution semantics for those fields and do not silently default behavior when authored values are present.
4. If changes are needed, handle as a dedicated follow-up slice (schema + generation + mapping + focused regression coverage) rather than mid-cadence churn.
5. Revisit ProjectCommandActionReferenceDto relationship semantics after lock cadence: `LinkedActions` is currently represented as a flat list but likely encodes call-stack/tree behavior.
6. Evaluate whether runtime/designer contracts should model explicit parent/child execution structure (or equivalent stack metadata) instead of relying on implicit list ordering.
7. If structural modeling changes are approved, execute as a dedicated compatibility-reviewed slice (schema + codegen + mapper + playback regression and command-processor regression coverage).
8. Revisit RoomCommandDto deeper after lock cadence: designer persists `phrase` while runtime contract uses structured command fields (`commandType`, `verb`, `qualifier`, `actionLinks`).
9. Decide canonical command authoring contract direction (phrase-first with runtime parsing, or structured command-first with designer editing support) and document migration implications.
10. If alignment changes are approved, execute as a dedicated compatibility slice (schema + codegen + JsonExportService mapping updates + runtime command loading/processing regression coverage).
11. Revisit RoomDto specifically for odd command behavior and image issues after lock cadence.
12. Command review scope: identify any RoomDto command oddities (phrase-to-runtime translation gaps, directional/qualifier edge cases, or ordering/dispatch anomalies) and confirm intended runtime semantics.
13. Image review scope: verify RoomDto image ownership/placement expectations across Designer vs Runtime contracts and validate export mapping does not drop or mis-shape room image references.
