# Action Variable Refactor Inventory Baseline

Status: R0 baseline complete
Date: 2026-07-08
Related plan: plans/active/ACTION_VARIABLE_REFACTOR_PLAN.md

## 1. Scope Of This Baseline

This inventory compares three things:

1. Designer-declared action variable tokens (token providers and payload hints).
2. Runtime-populated action variable values (resolvers + executable merge points).
3. Mismatch/drift notes to drive refactor slices.

## 2. Primary Evidence Locations

Designer token declarations:

1. StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviderRegistry.cs
2. StoryboardDesigner.App/Services/DefaultEmptyActionEchoReferenceTokenProviders.cs
3. StoryboardDesigner.App/Services/BuildCompositeByPartsActionEchoReferenceTokenProvider.cs
4. StoryboardDesigner.App/Services/BreakCompositeItemActionEchoReferenceTokenProvider.cs
5. StoryboardDesigner.App/Models/ActionPayloads/CompositeByPartsPayload.cs

Runtime token keys and value population:

1. Storyboard.Shared/GameServices/References/RuntimeTokenCatalog.cs
2. Storyboard.Shared/GameServices/References/BuildCompositeByPartsActionVariableResolver.cs
3. Storyboard.Shared/GameServices/References/BreakCompositeItemActionVariableResolver.cs
4. Storyboard.Shared/GameServices/References/PutObjectInContainerActionVariableResolver.cs
5. Storyboard.Shared/GameServices/References/RemoveObjectFromContainerActionVariableResolver.cs
6. Storyboard.Shared/GameServices/References/MoveToRoomActionVariableResolver.cs

Runtime executable merge points:

1. Storyboard.Shared/GameServices/Actions/GameActions/RuntimeActionExecutable/BuildCompositeByPartsExecutableAction.cs
2. Storyboard.Shared/GameServices/Actions/GameActions/RuntimeActionExecutable/BreakCompositeItemExecutableAction.cs
3. Storyboard.Shared/GameServices/Actions/GameActions/RuntimeActionExecutable/PutObjectInContainerExecutableAction.cs
4. Storyboard.Shared/GameServices/Actions/GameActions/RuntimeActionExecutable/RemoveObjectFromContainerExecutableAction.cs
5. Storyboard.Shared/GameServices/Actions/GameActions/RuntimeActionExecutable/NavigateDirectionExecutableAction.cs

## 3. Action-Type Inventory (Current)

### 3.1 BuildCompositeByParts

Designer tokens:

1. action.missingParts
2. action.missingPartsCount
3. action.missingParts[0]
4. action.missingParts[1]
5. action.candidateTargets
6. action.candidateTargetsCount
7. action.candidateTargets[0]
8. action.candidateTargets[1]
9. action.providedParts
10. action.providedPartsCount
11. action.providedParts[0]
12. action.providedParts[1]
13. action.requestedTarget
14. action.requestedTargetCount

Runtime population:

1. MissingParts list + count + indexed values.
2. CandidateTargets list + count + indexed values.
3. ProvidedParts list + count + indexed values.
4. RequestedTarget scalar + count.

Path behavior:

1. Failure path populated.
2. Success path populated (recent fix applied).

Status:

1. Aligned for declared tokens and path availability.

### 3.2 BreakCompositeItem

Designer tokens:

1. action.returnedParts
2. action.returnedPartsCount
3. action.returnedParts[0]
4. action.returnedParts[1]
5. action.restoredParts
6. action.restoredPartsCount
7. action.restoredParts[0]
8. action.restoredParts[1]
9. action.recipeParts
10. action.recipePartsCount
11. action.recipeParts[0]
12. action.recipeParts[1]
13. action.breakTarget
14. action.breakTargetCount

Runtime population:

1. ReturnedParts list + count + indexed values.
2. RestoredParts list + count + indexed values.
3. RecipeParts list + count + indexed values.
4. BreakTarget scalar + count.

Status:

1. Aligned.

### 3.3 PutObjectInContainer

Designer tokens:

1. action.targetContainer
2. action.targetContainerName
3. action.targetContainerCount
4. action.primaryitem
5. action.primaryitemCount
6. action.failureReason
7. action.failureReasonCount
8. action.totalCapacity
9. action.totalCapacityCount
10. action.totalUsedPoints
11. action.totalUsedPointsCount
12. action.totalRemainingCapacity
13. action.totalRemainingCapacityCount
14. action.totalInventoryPointsNeeded
15. action.totalInventoryPointsNeededCount

Runtime population:

1. TargetContainer scalar + count.
2. TargetContainerName scalar + count.
3. PrimaryItem scalar + count.
4. FailureReason scalar + count.
5. TotalCapacity scalar + count.
6. TotalUsedPoints scalar + count.
7. TotalRemainingCapacity scalar + count.
8. TotalInventoryPointsNeeded scalar + count.

Status:

1. Aligned.

### 3.4 RemoveObjectFromContainer

Designer tokens:

1. action.sourceContainer
2. action.sourceContainerCount
3. action.sourceContainerName
4. action.sourceContainerNameCount
5. action.failureReason
6. action.failureReasonCount
7. action.totalCapacity
8. action.totalCapacityCount
9. action.totalUsedPoints
10. action.totalUsedPointsCount
11. action.totalRemainingCapacity
12. action.totalRemainingCapacityCount
13. action.totalInventoryPointsNeeded
14. action.totalInventoryPointsNeededCount
15. action.isInternalPlayerTransfer
16. action.isInternalPlayerTransferCount

Runtime population:

1. SourceContainer scalar + count.
2. SourceContainerName scalar + count.
3. FailureReason scalar + count.
4. TotalCapacity scalar + count.
5. TotalUsedPoints scalar + count.
6. TotalRemainingCapacity scalar + count.
7. TotalInventoryPointsNeeded scalar + count.
8. IsInternalPlayerTransfer scalar + count.

Status:

1. Aligned.

### 3.5 NavigateDirection

Designer provider tokens (currently non-action-prefixed room references):

1. currentRoom.Name
2. currentRoom.name
3. priorRoom.Name
4. priorRoom.name

Runtime population:

1. action.Success and action.SuccessCount.
2. action.ResultCode and action.ResultCodeCount.
3. currentRoom.Name/currentRoom.name.
4. priorRoom.Name/priorRoom.name.

Drift note:

1. Designer provider does not declare action.Success/action.ResultCode although runtime populates them.
2. NavigateDirection currently mixes action.* and non-action room tokens in same provider concept.

Status:

1. Partial alignment; refactor target.

### 3.6 Other Action Types Registered With Empty Providers

Types currently returning no tokens from designer providers:

1. LinkedActions
2. Synonym
3. EchoMessage
4. SetFlag
5. CheckGameProperty
6. SetGameProperty
7. AddItem
8. RemoveItem
9. SetObjectState
10. BuildCompositeByTarget
11. UnlockExit
12. RandomChance
13. StartDialogue

Runtime notes:

1. BuildCompositeByTarget currently uses no-op runtime action variable resolver.
2. Multiple action types still rely on no-op resolvers and do not expose action.* token sets.

Status:

1. No immediate drift for empty/none behavior, but lacks explicit runtime-owned declarations.

## 4. Key Drift and Refactor Priorities

Priority 1:

1. Replace designer-owned token provider lists with Shared runtime-owned metadata for migrated action types.

Priority 2:

1. Resolve NavigateDirection mixed token model by defining canonical metadata that can represent:
   1. action.* tokens,
   2. non-action scoped aliases (if intentionally supported), or
   3. separation into dedicated categories.

Priority 3:

1. Define explicit metadata entries for currently empty action types so absence is declared, not implicit.

Priority 4:

1. Add parity tests that compare designer-consumed tokens to Shared metadata per action type.

## 5. Baseline Conclusion

1. Composite and container actions are mostly aligned today.
2. NavigateDirection demonstrates why single-source runtime-owned metadata is needed.
3. Current architecture still duplicates token declarations in designer and runtime logic.
4. Refactor should proceed with BuildCompositeByParts + BreakCompositeItem + container actions as first migration batch, then NavigateDirection normalization.
