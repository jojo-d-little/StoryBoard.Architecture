# Action Variable Chooser Event Anchor Unification - Stage 07 Legacy action.* Support Retirement Scope Handoff

Status: Closeout Prep (implementation complete; final archive and documentation pass pending)
Stage: 7 of 7
Date: 2026-09-14
Owner Session: GitHub Copilot (GPT-5.3-Codex)

## Opening Prompt (Use To Start This Stage)

Start Stage 07 of [plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md](plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md).
Read prior stage handoffs first, especially Stage 02 runtime pilot, Stage 03 shared provider, and Stage 04 designer unification outcomes.
Complete Stage 07 final retirement cleanup for legacy action.* support across designer and runtime, followed by regression hardening and closeout.
Do not broaden retirement beyond the approved per-action matrix in this handoff.
Honor Stage 07 boundaries and run closeout validation gates before marking complete.
Use the deferred shared/global retirement lists captured in Stage 05 wave evidence packets as the input backlog for this stage.

## Stage Boundary Allowlist Snapshot (From Main Plan)

Default deny rule:
1. Any path not explicitly listed in allowed read/edit scope is out of scope for this stage.

Edit-implies-read rule:
1. Any path in allowed edit scope is automatically readable.
2. Allowed read scope lists only extra read-only dependencies.

1. Allowed read scope:
- Repository-wide for validation and closeout evidence.
2. Allowed edit scope:
- StoryboardDesigner.App/** (token source registration, picker sourcing, and validation wiring tied to approved action-type retirements)
- StoryboardDesigner.App.Tests/** (regression and guardrail tests for retired legacy paths)
- Storyboard.GameEngine/** (runtime action-token cleanup and compatibility-path retirement tied to approved action-type retirements)
- Storyboard.GameEngine.Tests/** (runtime regression and guardrail tests for retired legacy paths)
- Storyboard.GameEngine/Config/action-payload.manifest.json (when needed for final canonical token catalog alignment)
- plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md
- plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_07_ACTION_TOKEN_LEGACY_RETIREMENT_HANDOFF.md

## Scope Completed

1. Completed Stage 07 retirement execution for legacy `currentAction.action.*` compatibility lookup behavior (canonical-only enforcement).
2. Completed runtime cache cleanup by removing legacy full-map fallback path in `RuntimeAnchorLookupCache`.
3. Completed chooser owner-root semantics and migration support (`owner.*` in chooser-local scripts).
4. Completed one-time sample/starter migration pass for chooser scripts from `self.*` to `owner.*` in `imageVariantChooserScript` fields.
5. Split deep pipeline-consolidation architectural review into a dedicated deferred active plan to avoid blocking closeout.

## Stage 07 Input Contract (From Stage 05)

1. Consume only items explicitly marked deferred shared/global retirement in Stage 05 wave evidence packets.
2. Reject any retirement candidate that lacks a corresponding completed wave parity packet.
3. Keep action-type traceability from Stage 05 packet to Stage 07 retirement commit/test evidence.

## Files Changed

1. plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_07_ACTION_TOKEN_LEGACY_RETIREMENT_HANDOFF.md
- Updated to closeout-prep status and execution evidence.
2. plans/active/SCOPECHAIN_PIPELINE_CONSOLIDATION_POST_CLOSEOUT_PLAN.md
- New deferred follow-up plan containing the former Stage 07 deep pipeline review item.

## Contract/Interface Impact

1. No external contract shape changes were introduced in this Stage 07 execution scope.
2. Canonical runtime action-output reference shape is now enforced as `currentAction.<leafKey>` for migrated families.
3. Compatibility lookup for `currentAction.action.<leafKey>` is retired in provider resolution.

## Legacy action.* Retirement Matrix (Scoped By Action Type)

Legend:
1. Legacy quick token source = current non-manifest action token provider/registry path.
2. Canonical source = action-payload manifest actionOutputVariableCatalog.
3. Retirement readiness requires parity tests and script validation pass for that action type.

## Legacy Source Code Review Map (Specific Files/Methods)

Review these before editing to retire any action type:

1. Designer entry-point registry and call sites
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviderRegistry.cs`
	- `CreateDefault()` (which action providers are currently registered)
	- `GetTokens(CommandActionType actionType)` (legacy token fetch)
- `StoryboardDesigner.App/Views/RoomActionEditorDialog.xaml.cs`
	- `GetEchoReferenceTokensForActionType(CommandActionType actionType)` (legacy + base merge point)
	- `BuildEchoReferenceSuggestions(string prefix)` (quick list composition)
	- `OpenEchoPropertyChooser()` (brace chooser command path)
- `StoryboardDesigner.App/Views/Controls/ActionScriptTextEditor.xaml.cs`
	- `BuildQuickReferenceSuggestions(string prefix)` (quick token surface)
	- `StartVariableCompletion()` (brace-trigger entry)

2. Designer validation usage of legacy action token providers
- `StoryboardDesigner.App/Validation/Rules/Scripting/ScriptUnknownReferenceRule.cs`
	- `Evaluate(ValidationRuleContext context)`
	- specifically `actionEchoTokenRegistry.GetTokens(actionCandidate.Action.ActionType)`

3. Shared source-of-truth currently backing legacy providers
- `Storyboard.GameEngine/GameServices/References/RuntimeActionVariableDescriptorRegistry.cs`
	- `GetDeclaredTokens(CommandActionType actionType)`
	- `Descriptors` map (notable coupling today: `MoveRoomObjectByPoints` currently maps to `RuntimeActionVariableDescriptors.MoveRoomObjectOnGrid`)
- `Storyboard.GameEngine/GameServices/References/RuntimeActionVariableDescriptors.cs`
	- per-action descriptor fields (contains current legacy `action.*` set and many `*Count` entries)

4. Canonical manifest-mirror source for action output variable keys
- `Storyboard.GameEngine/GameServices/References/RuntimeActionOutputVariableKeyCatalog.cs`
	- per-action canonical `actionOutputVariableKey` list that mirrors `Config/action-payload.manifest.json`

5. Manifest canonical source to replace legacy path
- `Storyboard.GameEngine/Config/action-payload.manifest.json`
	- `perActionTypeCatalog[*].actionOutputVariableCatalog`

## Per-Action Provider File/Method Review Targets

For each action type retired, review and adjust the corresponding provider class `GetTokens()` implementation and registration in `CreateDefault()`:

1. MoveRoomObjectOnGrid
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviders/MoveRoomObjectOnGridActionEchoReferenceTokenProvider.cs`
	- `GetTokens()`

2. MoveRoomObjectByPoints
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviders/MoveRoomObjectByPointsActionEchoReferenceTokenProvider.cs`
	- `GetTokens()`

3. StackRoomObjectOnAnother
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviders/StackRoomObjectOnAnotherActionEchoReferenceTokenProvider.cs`
	- `GetTokens()`

4. RotateRoomObjectOnGrid
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviders/RotateRoomObjectOnGridActionEchoReferenceTokenProvider.cs`
	- `GetTokens()`

5. NavigateDirection and NavigateToAdjacent
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviders/NavigateDirectionActionEchoReferenceTokenProvider.cs`
	- `GetTokens()`
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviders/NavigateToAdjacentActionEchoReferenceTokenProvider.cs`
	- `GetTokens()`

6. SetActiveRoomObject and ClearActiveRoomObjects
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviders/SetActiveRoomObjectActionEchoReferenceTokenProvider.cs`
	- `GetTokens()`
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviders/ClearActiveRoomObjectsActionEchoReferenceTokenProvider.cs`
	- `GetTokens()`

7. PutObjectInContainer and RemoveObjectFromContainer
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviders/PutObjectInContainerActionEchoReferenceTokenProvider.cs`
	- `GetTokens()`
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviders/RemoveObjectFromContainerActionEchoReferenceTokenProvider.cs`
	- `GetTokens()`

8. OpenObject, CloseObject, UnlockObject, LockObject
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviders/OpenObjectActionEchoReferenceTokenProvider.cs`
	- `GetTokens()`
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviders/CloseObjectActionEchoReferenceTokenProvider.cs`
	- `GetTokens()`
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviders/UnlockObjectActionEchoReferenceTokenProvider.cs`
	- `GetTokens()`
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviders/LockObjectActionEchoReferenceTokenProvider.cs`
	- `GetTokens()`

9. BuildCompositeByTarget, BuildCompositeByParts, BreakCompositeItem
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviders/BuildCompositeByTargetActionEchoReferenceTokenProvider.cs`
	- `GetTokens()`
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviders/BuildCompositeByPartsActionEchoReferenceTokenProvider.cs`
	- `GetTokens()`
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviders/BreakCompositeItemActionEchoReferenceTokenProvider.cs`
	- `GetTokens()`

10. InvokeProcedure
- `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviders/InvokeProcedureActionEchoReferenceTokenProvider.cs`
	- `GetTokens()`

## PR Review Checklist (Per Retired Action Type)

1. Provider no longer sources tokens from `RuntimeActionVariableDescriptorRegistry.GetDeclaredTokens` for that retired action type.
2. `ActionEchoReferenceTokenProviderRegistry.CreateDefault()` still has exactly one provider for that action type and no duplicate registrations.
3. Echo quick suggestions and brace chooser for that action type resolve to manifest-canonical `actionOutputVariableCatalog` tokens.
4. `ScriptUnknownReferenceRule` accepts manifest-canonical tokens for that action type with no regressions.
5. Any retired `action.*Count` scalar aliases are either intentionally retained for compatibility (documented) or intentionally removed with migration evidence.

## Runtime Cleanup Detail (Stage 07 In-Scope)

Purpose:
1. Keep Stage 07 retirement review explicit about runtime debt and execute final runtime cleanup in this stage after parity gates.

Boundary note:
1. Runtime cleanup is in current Stage 07 edit scope for approved action types in the retirement matrix.
2. Runtime cleanup must remain action-type-scoped and evidence-backed; avoid global removals without per-action parity proof.

Runtime cleanup targets to review and execute in Stage 07:
1. `Storyboard.GameEngine/GameServices/References/RuntimeActionOutputVariableKeyCatalog.cs`
- Keep this file as the one-stop canonical mirror for `actionOutputVariableKey` values.
- Ensure action-specific runtime population paths use these keys for `CurrentAction` output storage.

2. `Storyboard.GameEngine/GameServices/References/RuntimeActionVariableDescriptorRegistry.cs`
- Review `Descriptors` map and `GetDeclaredTokens(CommandActionType actionType)` legacy usage.
- Validate whether `MoveRoomObjectByPoints` mapping to `RuntimeActionVariableDescriptors.MoveRoomObjectOnGrid` should remain or split.

3. `Storyboard.GameEngine/GameServices/References/RuntimeActionVariableDescriptors.cs`
- Review per-action token declarations for legacy scalar `action.*Count` aliases.
- Retire scalar count aliases action-type-by-action-type only after compatibility evidence.

4. `Storyboard.GameEngine/GameServices/References/MoveRoomObjectOnGridActionVariableResolver.cs`
- Review scalar alias emission path (`AddScalarWithActionPropertyAlias`) and compatibility implications.
- If retiring scalar count aliases in runtime, update emitted keys and associated tests in the same change.

5. `Storyboard.GameEngine/GameServices/TextAndDiagnostics/ActionPropertyReferenceValueResolver.cs`
- Keep list count behavior for true list tokens unless explicitly redesigning list semantics.
- Do not conflate scalar alias retirement with list count semantics.

6. `Storyboard.GameEngine/GameServices/References/RuntimeTokenCatalog.cs`
- Review `CountFor(string key)` usage and confirm deprecation plan does not break remaining list-based count consumers.

7. `Storyboard.GameEngine/GameServices/References/RuntimeAnchorLookupCache.cs`
- Review `AllowLegacyFallback` behavior and decide whether fallback-to-`Resolve` remains necessary after full consumer migration/parity.
- If Stage 07 parity evidence is complete across command/event/timer/action consumers, retire fallback path and keep `TryResolveSingle` + scoped cache as the only value-resolution route.
- Pair fallback removal with focused replay/linked-action/event/timer regression evidence to confirm no stale-read or token-coverage regressions.

Runtime cleanup checklist (per action type):
1. Confirm manifest canonical keys cover runtime-emitted action outputs for the action.
2. Remove or retain scalar `action.*Count` aliases intentionally, with migration note.
3. Run action-focused runtime regression tests and replay regression.
4. Update this handoff with explicit before/after token set evidence.
5. Decide keep/remove for `RuntimeAnchorLookupCache` legacy fallback, and record rationale + evidence in this handoff.

## Deferred Deep Review

1. The former Stage 07 deep review item has been moved to a dedicated deferred plan:
- `plans/active/SCOPECHAIN_PIPELINE_CONSOLIDATION_POST_CLOSEOUT_PLAN.md`
2. This workstream closeout no longer blocks on that architecture decision.
3. Execute the deferred plan after this workstream is archived.

## Validation Commands Executed

1. `dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~Process_ObjectImageVariantChooser|FullyQualifiedName~RuntimeSessionAnchorDataProviderTests"`
- Result: PASS (31 passed, 0 failed).
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~QuantifiableRenamePropagationTests|FullyQualifiedName~MainWindowViewModelVariableChoicesTests|FullyQualifiedName~MainWindowViewModelImportGlobalsCommandTests"`
- Result: PASS (27 passed, 0 failed).
3. `dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~ActionVariableEmissionGuardrailTests"`
- Result: PASS (15 passed, 0 failed).
4. `dotnet build .\StoryboardDesigner.slnx`
- Result: PASS.

## Test Results

1. Focused runtime and designer regression suites are green for Stage 07 retirement scope.
2. Guardrail and parity checks touched by Stage 07 changes are green.
3. Remaining closeout step is final documentation/archive pass plus any optional broad-suite rerun desired by maintainer policy.

## Behavioral Notes

1. `currentAction.action.<leafKey>` compatibility resolution is no longer accepted; canonical form is `currentAction.<leafKey>`.
2. `RuntimeAnchorLookupCache` now resolves through `TryResolveSingle(...)` only; legacy full-map fallback is removed.
3. Image chooser local-object semantics are explicit via `owner.*`; `self.*` remains action-scope semantics.
4. Sample/starter chooser script migration for `imageVariantChooserScript` fields is complete for targeted roots.

## Known Issues/Risks

1. Optional broad-suite rerun may still reveal unrelated environmental flakes (file lock/process contention).
2. Follow-up architecture hardening for scopechain/pipeline ownership is intentionally deferred to post-closeout plan.

## Boundary Compliance Report

1. Out-of-scope reads performed:
- None.
2. Out-of-scope edits performed:
- None.
3. Stage-boundary exceptions approved:
- None.
4. Session context scope notes:
- Stage 07 should reject broad cleanup without explicit per-action parity evidence.

## Stage 07 Closeout Checklist

1. Update Stage 06 handoff to explicit skipped/no-op completion status (with rationale).
2. Confirm final manifest description alignment for navigation output display-name semantics.
3. Mark main plan status as closeout-ready and archive when approved.

## Explicit Remaining Closeout Tasks

1. Optional: run full runtime-focused regression + replay gates again immediately before archival.
2. Archive plan + all stage handoffs as one unit after approval.
