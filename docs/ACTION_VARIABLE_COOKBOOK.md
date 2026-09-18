# Action Variable Cookbook

Status: Consolidated implementation guide (2026-07-08). Pilot approved, expanded action migration is in place, and plan closeout is accepted for current scope.

Purpose: provide a repeatable process for adding and extending action-scoped variables (action.*) across designer authoring, validation, runtime population, and tests.

Audience: developers modifying action behavior in StoryboardDesigner.App and Storyboard.Shared.

## TLDR Checklist (Add A New action.* Variable)

Use this when you only need the fastest safe path.

1. Add/update the canonical token key in Shared constants if needed:
	- Storyboard.Shared/GameServices/References/RuntimeTokenCatalog.cs
2. Add the token to the action's canonical descriptor in Shared:
	- Storyboard.Shared/GameServices/References/RuntimeActionVariableDescriptors.cs
3. If this is a brand-new descriptor/action mapping, register it:
	- Storyboard.Shared/GameServices/References/RuntimeActionVariableDescriptorRegistry.cs
4. Update the action variable resolver contract/implementation to produce the new token value:
	- Storyboard.Shared/GameServices/References/I*ActionVariableResolver.cs
	- Storyboard.Shared/GameServices/References/*ActionVariableResolver.cs
	- Prefer neutral resolver APIs (for example `BuildValues`) so the same resolver supports both success and failure contexts.
5. Ensure runtime executable action passes resolver values on every relevant path (success/failure as applicable) before outcome interpolation and guardrail checks:
	- Storyboard.Shared/GameServices/Actions/GameActions/RuntimeActionExecutable/*ExecutableAction.cs
6. Ensure designer token discovery uses Shared canonical declarations (no local literal token lists):
	- StoryboardDesigner.App/Services/*ActionEchoReferenceTokenProvider.cs
	- StoryboardDesigner.App/Models/ActionPayloads/*.cs
7. Add/update tests (minimum):
	- Provider parity against Shared declarations
	- Script unknown-reference acceptance for valid token use
	- Runtime emitted-token legality/guardrail coverage
	- Action execution behavior coverage for representative output
8. Run the full validation cycle:
	- dotnet build .\StoryboardDesigner.slnx
	- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"
	- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
	- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
9. Record the change in the active plan execution log and keep cookbook snapshot text in sync.

## 0. Current State Snapshot

Current objective:

1. Maintain Shared as the canonical source for action-variable declarations by action type.
2. Keep designer provider and payload token sourcing aligned to Shared metadata for migrated actions.
3. Enforce runtime emitted-token legality at execution merge points.

Implemented baseline (completed):

1. Shared canonical declarations exist in RuntimeActionVariableDescriptors and are served by RuntimeActionVariableDescriptorRegistry.
2. Migrated action providers consume Shared descriptor tokens (no local provider token literals for migrated actions).
3. Migrated payload GetReferenceTokens implementations consume Shared descriptor tokens where applicable.
4. Script unknown-reference validation remains compatible through registry-driven token assembly.
5. Runtime emitted-token declaration guardrail runs at executable merge points and appends diagnostics for undeclared emissions.
6. Validation evidence is captured through full validation cycle gates.
7. Final R4 redundancy sweep confirmed no remaining duplicate token declarations in migrated designer/runtime token paths.
8. First R5 opt-in declaration slice is in place: `action.all` is declared as an optional diagnostic token for NavigateDirection only; non-opted-in actions remain unchanged.
9. Runtime generation slice is in place for opted-in actions: `action.all` is emitted as declaration-order line-delimited `key=value` entries (excluding recursive `action.all` self-inclusion), currently exercised by NavigateDirection.
10. Composite opt-in expansion is complete: BuildCompositeByTarget, BuildCompositeByParts, and BreakCompositeItem now declare and emit `action.all` via the same optional Shared metadata pattern.
11. Container transfer opt-in expansion is complete: PutObjectInContainer and RemoveObjectFromContainer now declare and emit `action.all` using the same declaration-driven runtime append pattern and deterministic declaration-order output contract.
12. Current-scope non-composite opt-in is complete: NavigateDirection, PutObjectInContainer, and RemoveObjectFromContainer are all opted in to `action.all` within the existing Shared canonical descriptor set.

Migrated action coverage:

1. BuildCompositeByParts
2. BreakCompositeItem
3. PutObjectInContainer
4. RemoveObjectFromContainer
5. NavigateDirection

Scope note: additional action types are evaluated for `action.all` only after they are migrated into Shared canonical action-variable descriptors.

## 1. Quick Mental Model

Action variables are script references like:

- action.missingParts
- action.missingPartsCount
- action.missingParts[0]

These are used in outcome scripts (for example Failure, Success, or result-code-specific messages).

Lifecycle (current model):

1. Shared declares canonical token names by action type.
2. Designer consumes Shared token declarations through thin facade adapters.
3. Script validation allows tokens from the Shared-backed source.
4. Runtime produces values during action execution and validates emitted keys against canonical declarations at executable merge points.
5. If a declared token has no value in current context, resolver returns empty/no-value (not unknown-reference).
6. Tests confirm declaration parity, runtime legality, and unknown-reference stability.

## 2. Pilot Lockoffs Applied In This Draft

1. Canonical token declarations live in Shared and are registered by action type.
2. Designer stays Shared-first with legacy fallback during migration.
3. Token ordering is declaration order from Shared metadata.
4. Availability metadata shape is prepared now, but current implementation actively supports only `Both`.
5. Path-specific applicability behavior is deferred; declared token names remain legal even when unresolved values are empty for current context.
6. Diagnostics-style tokens (for example `action.all`) are treated as normal optional action variables declared per action type; there is no hard requirement that every action type support them.

## 3. Source Of Truth By Layer

Shared canonical token declaration and runtime behavior:

1. Storyboard.Shared (new canonical action-variable metadata registry and action-local descriptors in pilot implementation)
2. Storyboard.Shared/GameServices/References/RuntimeTokenCatalog.cs
3. Storyboard.Shared/GameServices/References/*ActionVariableResolver.cs
4. Storyboard.Shared/GameServices/References/RuntimeActionVariableDeclarationGuardrail.cs
5. Storyboard.Shared/GameServices/Actions/GameActions/RuntimeActionExecutable/*ExecutableAction.cs

Designer bridge and consumption:

1. StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviderRegistry.cs
2. StoryboardDesigner.App/Services/*ActionEchoReferenceTokenProvider.cs (thin adapters for migrated actions + legacy fallback for non-migrated actions)
3. StoryboardDesigner.App/Models/ActionPayloads/*.cs (migration compatibility; remove duplicates as Shared coverage expands)

Designer editor usage:

1. StoryboardDesigner.App/Views/RoomActionEditorDialog.xaml.cs

Designer script validation:

1. StoryboardDesigner.App/Validation/Rules/Scripting/ScriptUnknownReferenceRule.cs

Runtime token keys and value building:

1. Storyboard.Shared/GameServices/References/RuntimeTokenCatalog.cs
2. Storyboard.Shared/GameServices/References/*ActionVariableResolver.cs
3. Storyboard.Shared/GameServices/References/RuntimeActionVariableDeclarationGuardrail.cs
4. Storyboard.Shared/GameServices/TextAndDiagnostics/ActionPropertyReferenceValueResolver.cs
5. Storyboard.Shared/GameServices/Actions/GameActions/RuntimeActionExecutable/*ExecutableAction.cs

Runtime outcome/result-code metadata:

1. Storyboard.Shared/GameServices/Actions/GameActions/RuntimeActionResultCodeEnums/*.cs
2. Storyboard.Shared/GameServices/Actions/RuntimeActionResultCodeRegistry.cs

## 4. Cookbook A: Add New action.* Variables To An Existing Action Type (Pilot Model)

Example target: BuildCompositeByParts.

Step 1: Declare canonical token names in Shared metadata for the action type.

1. Add or update action-local descriptor entries in Shared.
2. Keep declaration order intentional (this is canonical ordering contract).
3. Optionally include availability shape metadata, but use `Both` as active value in current slices.

Step 2: Add or update canonical key names in RuntimeTokenCatalog when needed.

1. Edit Storyboard.Shared/GameServices/References/RuntimeTokenCatalog.cs.
2. Add constants under RuntimeTokenCatalog.ActionProperty.
3. Use short key names without the action. prefix. The prefix is applied by helper methods.

Step 3: Populate runtime values in the action variable resolver.

1. Edit or create the resolver in Storyboard.Shared/GameServices/References.
2. For list-like values, use ActionPropertyReferenceValueResolver.AddListValues.
3. For scalar values, set both value and count (for consistency with editor token expectations).

Notes:

1. AddListValues emits action.<key>, action.<key>Count, and action.<key>[n].
2. ensureZeroIndexAlias=true guarantees action.<key>[0] exists for empty lists.

Step 4: Ensure executable action injects resolver values on relevant paths.

1. Edit the runtime executable in Storyboard.Shared/GameServices/Actions/GameActions/RuntimeActionExecutable.
2. Merge resolver values into invocation context before TryAppendOutcomeMessage.
3. Run RuntimeActionVariableDeclarationGuardrail against emitted values before invocation-context merge.
4. Current-phase rule: declared token names are legal on any path; if value is unavailable in current context/path, resolve as empty/no-value.

Step 5: Advertise tokens in designer provider via Shared-backed facade adapter.

1. Keep IActionEchoReferenceTokenProvider interface unchanged in designer.
2. Implement provider as thin facade that reads Shared canonical metadata.
3. Register Shared-backed provider path in ActionEchoReferenceTokenProviderRegistry.CreateDefault.
4. Keep legacy fallback only for non-migrated action types.

Step 6: Keep payload model token hints aligned with Shared canonical descriptors.

1. For migrated actions, prefer Shared descriptor lookup from payload `GetReferenceTokens` instead of local hardcoded token arrays.
2. For non-migrated actions, keep payload token hints aligned with provider behavior until Shared migration is complete.

Step 7: Verify editor wiring.

1. Confirm RoomActionEditorDialog calls GetEchoReferenceTokensForActionType for the action.
2. Confirm fields that should use these variables invoke the action script editor with those tokens.

Step 8: Extend or confirm script validation behavior.

1. ScriptUnknownReferenceRule should already pull action echo tokens from the registry.
2. Add tests proving new action.* tokens are accepted.
3. Add tests proving legacy unprefixed aliases are rejected if that is expected.

Step 9: Add runtime tests for real value emission.

1. Add tests in StoryboardDesigner.App.Tests/CompositeBuildActionTests.cs (or action-specific test file).
2. Assert resolved outputs for success and failure templates using the new tokens.
3. Include indexed token coverage where relevant (for example action.foo[0]).

## 5. Cookbook B: Add action.* Variables For A Brand New Action Type

Do all steps from Cookbook A, plus:

1. Add a new IActionEchoReferenceTokenProvider implementation for the action type.
2. Register it in ActionEchoReferenceTokenProviderRegistry.CreateDefault.
3. Ensure RuntimeCommandActionExecutor maps the action type to its executable handler.
4. Add result-code enum + token map if this action has custom result codes.
5. Add/extend payload accessors in RuntimeActionPayloadAccessors if action payload shape is new.
6. Add summary/editor support in RoomActionEditorDialog and any presenter used by the action.

## 6. BuildCompositeByParts Reference (Current Supported action.* Tokens)

Current token set exposed in designer and used by runtime:

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

Runtime may emit additional list indices action.<key>[n] beyond [1] depending on list size.

## 7. Result Code Integration Checklist

For actions with typed result codes:

1. Add/maintain enum and token mapping in RuntimeActionResultCodeEnums.
2. Ensure RuntimeActionResultCodeRegistry includes the action type.
3. Ensure action execution returns the appropriate result-code token.
4. Ensure outcome message maps in designer include expected code keys.
5. Add tests for case-insensitive result-code lookup.

## 8. Common Failure Patterns

1. Token appears in editor but not at runtime.
- Resolver is missing value population or executable path does not merge values into invocation context.

2. Runtime produces value but script rule flags unknown reference.
- Token provider is missing registration or token list mismatch.

3. action.foo[0] throws unknown reference for empty list.
- Resolver did not call AddListValues with ensureZeroIndexAlias=true.

4. Outcome script never uses custom token on success.
- Variables only populated on failure path in executable action.

5. Token works in one action type but not another.
- Providers are action-type-specific; registry entry may be missing for that type.

## 9. Minimum Test Matrix For Any Token Change

1. Shared declaration test: action type exposes expected canonical tokens in declaration order.
2. Provider registry bridge test: designer-visible tokens match Shared canonical tokens for migrated actions.
3. Validation test: action.<token> accepted, legacy alias behavior validated.
4. Runtime legality test: emitted token names are declared (resolver-level and execution merge-point guardrail coverage).
5. Runtime execution test: output template resolves token value or empty/no-value for declared-but-unavailable contexts.
6. Indexed token test: [0] works when list empty and non-empty.
7. Optional replay regression update if snapshots intentionally changed.

## 10. Validation Commands

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
4. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj

## 11. Suggested PR Checklist (Copy/Paste)

1. Added/updated Shared canonical declaration for action token names.
2. Preserved or intentionally updated declaration ordering.
3. Added/updated RuntimeTokenCatalog keys (if needed).
4. Added/updated runtime variable resolver population.
5. Added/updated executable action invocation-context merge points.
6. Added/updated designer facade provider to Shared-backed token source (or documented fallback when not migrated).
7. Added/updated payload token hints (GetReferenceTokens) where still used.
8. Added/updated script validation tests.
9. Added/updated runtime legality and execution tests (including central guardrail coverage when execution-path behavior changed).
10. Ran build, focused runtime gate, replay gate, and full suite.
11. Documented intentional empty/no-value behavior for declared tokens in unavailable contexts.

## 12. Consolidation Review Checklist

1. Confirm migrated actions are declared in Shared descriptor registry and preserve intended token ordering.
2. Confirm migrated provider and payload token paths resolve from Shared descriptor registry.
3. Confirm unknown-reference validation accepts canonical tokens and rejects unknown aliases.
4. Confirm runtime emitted-token guardrail coverage exists for migrated actions.
5. Confirm full validation cycle evidence is attached and green.
6. Record any additional approvals or lock updates in plans/active/ACTION_VARIABLE_REFACTOR_PLAN.md.
