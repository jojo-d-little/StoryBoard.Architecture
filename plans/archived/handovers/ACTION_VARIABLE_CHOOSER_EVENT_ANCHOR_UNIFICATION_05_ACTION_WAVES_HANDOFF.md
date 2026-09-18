# Action Variable Chooser Event Anchor Unification - Stage 05 Action Output Catalog Migration Waves Handoff

Status: Completed (Waves 1-7 completed)
Stage: 5 of 7
Date: 2026-09-13
Owner Session: GitHub Copilot (GPT-5.3-Codex)

## Opening Prompt (Use To Start This Stage)

Start Stage 05 of [plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md](plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md).
Complete only action-output migration waves with one action at a time and parity evidence per wave.
Do not begin Stage 07 final retirement until Stage 05 evidence is complete.
Honor stage boundaries from the main plan and record validation evidence.

## Wave Execution Policy (Required)

1. Use one action family per wave.
2. Perform wave-scoped cleanup during the same wave when the cleanup impacts only that migrated action family.
3. Defer shared/global retirement to Stage 07 when cleanup affects non-migrated families, shared parser semantics, or cross-family fallback behavior.
4. Keep compatibility fallback active for all non-migrated action families.

## Wave Opening Prompts (Copy/Paste)

Use one prompt per wave and replace placeholders before execution.

### Generic Wave Prompt Template

Start Stage 05 Wave <WAVE_NUMBER> for action family <ACTION_FAMILY> using [plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md](plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md) and [plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md](plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md).
Migrate only <ACTION_FAMILY> to canonical actionOutputVariableCatalog discovery and runtime emission parity.
Do wave-local cleanup only for <ACTION_FAMILY>; defer all shared/global legacy retirement to Stage 07.
Produce before/after token snapshot, parity evidence, validation results, and an explicit rollback note for this wave.

### Wave 1 Prompt (Move Family)

Start Stage 05 Wave 1 for action family MoveRoomObjectOnGrid and MoveRoomObjectByPoints parity companion using [plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md](plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md) and [plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md](plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md).
Migrate only move-family action output discovery and runtime emission alignment to canonical actionOutputVariableCatalog.
Do wave-local cleanup only for move-family legacy paths that are no longer needed after parity is proven; keep shared/global fallback untouched.
Produce before/after move token snapshot, parity evidence, validation results, and rollback note.

### Wave 2 Prompt (Stack Family)

Start Stage 05 Wave 2 for action family StackRoomObjectOnAnother using [plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md](plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md) and [plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md](plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md).
Migrate only stack-family action output discovery and runtime emission alignment to canonical actionOutputVariableCatalog.
Do wave-local cleanup only for stack-family legacy paths; defer shared/global retirement to Stage 07.
Produce before/after stack token snapshot, parity evidence, validation results, and rollback note.

### Wave 3 Prompt (Active Object Family)

Start Stage 05 Wave 3 for action family SetActiveRoomObject and ClearActiveRoomObjects using [plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md](plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md) and [plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md](plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md).
Migrate only active-object-family output discovery and runtime emission alignment.
Do wave-local cleanup only for this family; defer shared/global retirement to Stage 07.
Produce before/after token snapshot, parity evidence, validation results, and rollback note.

### Wave 4 Prompt (Container Family)

Start Stage 05 Wave 4 for action family PutObjectInContainer and RemoveObjectFromContainer using [plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md](plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md) and [plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md](plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md).
Migrate only container-family output discovery and runtime emission alignment.
Do wave-local cleanup only for this family; defer shared/global retirement to Stage 07.
Produce before/after token snapshot, parity evidence, validation results, and rollback note.

### Wave 5 Prompt (Open/Close/Lock Family)

Start Stage 05 Wave 5 for action family OpenObject, CloseObject, LockObject, and UnlockObject using [plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md](plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md) and [plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md](plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md).
Migrate only object-state-family output discovery and runtime emission alignment.
Do wave-local cleanup only for this family; defer shared/global retirement to Stage 07.
Produce before/after token snapshot, parity evidence, validation results, and rollback note.

### Wave 6 Prompt (Composite Family)

Start Stage 05 Wave 6 for action family BuildCompositeByTarget, BuildCompositeByParts, and BreakCompositeItem using [plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md](plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md) and [plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md](plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md).
Migrate only composite-family output discovery and runtime emission alignment.
Do wave-local cleanup only for this family; defer shared/global retirement to Stage 07.
Produce before/after token snapshot, parity evidence, validation results, and rollback note.

### Wave 7 Prompt (Navigation/Procedure Family)

Start Stage 05 Wave 7 for action family NavigateDirection, NavigateToAdjacent, and InvokeProcedure using [plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md](plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md) and [plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md](plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md).
Migrate only navigation/procedure-family output discovery and runtime emission alignment.
Do wave-local cleanup only for this family; defer shared/global retirement to Stage 07.
Produce before/after token snapshot, parity evidence, validation results, and rollback note.

## Required Per-Wave Evidence Packet

1. Scope statement naming only the selected action family.
2. Before/after token set snapshot.
3. Runtime emission parity proof for selected family.
4. Unknown-reference script validation proof for selected family.
5. List of wave-local cleanups performed.
6. List of deferred retirements explicitly pushed to Stage 07.
7. Rollback note describing how to revert only this wave.

## Stage Boundary Allowlist Snapshot (From Main Plan)

Default deny rule:
1. Any path not explicitly listed in allowed read/edit scope is out of scope for this stage.

Edit-implies-read rule:
1. Any path in allowed edit scope is automatically readable.
2. Allowed read scope should list only extra read-only dependencies.

1. Allowed read scope:
- plans/**
- StoryboardDesigner.App/**
- StoryboardDesigner.App.Tests/**
- Storyboard.GameEngine/**
- Storyboard.GameEngine.Tests/**

2. Allowed edit scope:
- StoryboardDesigner.App/**
- StoryboardDesigner.App.Tests/**
- Storyboard.GameEngine/**
- Storyboard.GameEngine.Tests/**
- Storyboard.GameEngine/Config/action-payload.manifest.json
- plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md
- plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md

## Scope Completed

1. Completed Wave 1 for move-family actions:
- `MoveRoomObjectOnGrid`
- `MoveRoomObjectByPoints` (parity companion)
2. Completed Wave 2 for stack-family action:
- `StackRoomObjectOnAnother`
3. Finalized canonical current-action output key flow for move/stack-family parity:
- Canonical authored/lookup direction is `currentAction.<leafKey>`.
- Temporary compatibility alias retained for `currentAction.action.<leafKey>` during migration.
4. Added/confirmed one-stop canonical runtime key catalog mirroring manifest ownership for action output keys.
5. Finalized action-frame output lifecycle semantics:
- Begin action context starts with empty `ActionOutputValues`.
- Current action output values are populated only when known and updated in-place on the existing action context.
6. Captured Stage 07 follow-up review item for resolver-pipeline duplication/performance and exact heavy-path usage inventory.

## Wave 1 Evidence Packet (Move Family)

1. Scope statement:
- Only move family was treated as completed in this wave (`MoveRoomObjectOnGrid` with `MoveRoomObjectByPoints` parity companion).

2. Before/after token snapshot:
- Canonical move output token shape is now `currentAction.<leafKey>` (for example `currentAction.movedObjectName`).
- Compatibility alias accepted temporarily: `currentAction.action.<leafKey>`.
- Move-family declared output leaf keys in canonical catalog/manifest mirror:
	- `success`
	- `resultCode`
	- `movedObjectName`
	- `moveDirection`
	- `moveDistance`
	- `all`

3. Runtime emission parity proof:
- Canonical manifest parity guardrail exists and passes for declared output keys.
- Move-family emission guardrail coverage exists and passes for declared token legality and parity companion expectations.

4. Unknown-reference/script-path proof relevant to this wave:
- Current-action output values resolve through current action context storage and anchor lookup paths with compatibility alias retained.

5. Wave-local cleanup performed:
- Removed begin-frame pre-seeding of current action outputs from invocation values.
- Begin-frame now starts with empty outputs; mutation occurs when real values are available.

6. Deferred shared/global retirement (explicitly pushed to Stage 07):
- Retire temporary `currentAction.action.*` compatibility alias after project-wide migration/search correction and validation.
- Resolve duplication between `ScopeChainReferenceValueResolver` full-population behavior and `RuntimeSessionAnchorDataProvider` single-path logic.
- Decide keep/remove of `RuntimeAnchorLookupCache` legacy full-map fallback (`AllowLegacyFallback`).

## Stage 07 Cleanup Backlog Linked To Wave 2 (Stack Family)

1. Stack-family alias retirement target:
- Remove temporary acceptance of `currentAction.action.<leafKey>` for stack outputs after all Stage 05 waves complete and canonical `currentAction.<leafKey>` references are validated across scripts/tests.

2. Stack-family canonical discovery enforcement target:
- Remove stack-family chooser/unknown-reference dependency on descriptor-registry fallback for output-key ownership once all families are catalog-migrated and parity gates are green.

3. Stack-family emission compatibility retirement target:
- Retire stack-family legacy `actionProperty.*` compatibility expectations where still only serving migration fallback, keeping only canonical leaf-key ownership from `actionOutputVariableCatalog` and `RuntimeActionOutputVariableKeyCatalog`.

4. Stack-family Stage 07 exit checks:
- Run full Stage 05/07 validation cycle with stack-focused assertions proving canonical-only key path acceptance remains green.
- Confirm no remaining authored or test references require stack-specific compatibility alias paths.

7. Rollback note:
- Roll back Wave 1 by reverting only Wave 1 runtime/action-output catalog + current-action lifecycle commits; do not revert Stage 03/04 shared-provider and designer-unification groundwork.

## Wave 2 Evidence Packet (Stack Family)

1. Scope statement:
- Only stack family was treated as completed in this wave (`StackRoomObjectOnAnother`).

2. Before/after token snapshot:
- Before Wave 2:
	- `action-payload.manifest.json` had an empty `actionOutputVariableCatalog` for `StackRoomObjectOnAnother`.
	- `RuntimeActionOutputVariableKeyCatalog` had no canonical stack output-key list.
- After Wave 2:
	- Canonical stack output token shape is `currentAction.<leafKey>` (for example `currentAction.stackTargetObjectName`).
	- Compatibility alias remains accepted temporarily: `currentAction.action.<leafKey>`.
	- Stack-family declared output leaf keys in canonical catalog/manifest mirror:
		- `success`
		- `resultCode`
		- `stackedObjectName`
		- `stackTargetObjectName`
		- `stackDirection`
		- `stackDistance`
		- `all`

3. Runtime emission parity proof:
- Canonical manifest parity guardrail passes with Wave 2 stack keys included.
- Stack-family emission guardrail coverage passes for emitted-token legality against declared metadata.
- Stack runtime action suite remains green after canonical catalog migration.

4. Unknown-reference/script-path proof relevant to this wave:
- Added focused provider coverage proving stack outputs resolve via current action context for both:
	- canonical `currentAction.stackTargetObjectName`
	- temporary alias `currentAction.action.stackTargetObjectName`

5. Wave-local cleanup performed:
- Stack-family action variable resolver now emits canonical leaf keys directly (`success`, `resultCode`, `stackedObjectName`, `stackTargetObjectName`, `stackDirection`, `stackDistance`) in addition to legacy `actionProperty.*` aliases.
- Stack-family descriptor metadata now declares those canonical leaf keys so emission guardrails and token discovery include the canonical shape during the migration window.
- This reduces stack-family dependence on legacy-prefixed extraction while preserving compatibility paths for non-retired families.

6. Deferred shared/global retirement (explicitly pushed to Stage 07):
- Retire temporary `currentAction.action.*` compatibility alias after project-wide migration/search correction and validation.
- Resolve duplication between `ScopeChainReferenceValueResolver` full-population behavior and `RuntimeSessionAnchorDataProvider` single-path logic.
- Decide keep/remove of `RuntimeAnchorLookupCache` legacy full-map fallback (`AllowLegacyFallback`).

7. Rollback note:
- Roll back Wave 2 by reverting only Wave 2 stack-family changes in:
	- `Storyboard.GameEngine/Config/action-payload.manifest.json`
	- `Storyboard.GameEngine/GameServices/References/RuntimeActionOutputVariableKeyCatalog.cs`
	- `Storyboard.GameEngine/GameServices/References/StackRoomObjectOnAnotherActionVariableResolver.cs`
	- `Storyboard.GameEngine/GameServices/References/RuntimeActionVariableDescriptors.cs`
	- `Storyboard.GameEngine.Tests/ActionVariableEmissionGuardrailTests.cs`
	- `Storyboard.GameEngine.Tests/RuntimeSessionAnchorDataProviderTests.cs`
	- this handoff markdown update
- Do not revert prior Wave 1 or Stage 03/04 groundwork.

## Wave 2 Validation Addendum (Stack Cleanup Follow-Through)

1. `dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~ActionVariableEmissionGuardrailTests"`
- Result: PASS (15 passed, 0 failed).

2. `dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~RuntimeActionOutputVariableManifestParityTests|FullyQualifiedName~RuntimeSessionAnchorDataProviderTests|FullyQualifiedName~RuntimeStackRoomObjectOnAnotherActionTests"`
- Result: PASS (17 passed, 0 failed).

## Wave 2 Validation Commands Executed

1. `dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~ActionVariableEmissionGuardrailTests|FullyQualifiedName~RuntimeActionOutputVariableManifestParityTests|FullyQualifiedName~RuntimeSessionAnchorDataProviderTests"`
- Result: PASS (30 passed, 0 failed).

2. `dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~RuntimeStackRoomObjectOnAnotherActionTests"`
- Result: PASS (2 passed, 0 failed).

3. `dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~RuntimeScopeMutationGatewayPhase3ATests"`
- Result: PASS (39 passed, 0 failed).

4. `dotnet build .\StoryboardDesigner.slnx`
- Result: PASS.

## Wave 2 Boundary Compliance Report

1. Out-of-scope reads performed:
- None.
2. Out-of-scope edits performed:
- None.
3. Stage-boundary exceptions approved:
- None.
4. Session context scope notes:
- Wave 2 remained scoped to stack-family action-output catalog migration and parity verification.

## Files Changed

1. plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md
- Upgraded from placeholder to Wave 1 completed evidence + Wave 2+ pending execution scaffold.

2. Storyboard.GameEngine/Config/action-payload.manifest.json
- Move-family action output variable keys aligned to leaf-key canonical form.

3. Storyboard.GameEngine/GameServices/References/RuntimeActionOutputVariableKeyCatalog.cs
- Canonical per-action output-key catalog introduced/expanded for manifest parity ownership.

4. Storyboard.GameEngine/GameStateData/GameStateActionContext.cs
- Current action output values modeled as mutable dictionary contents.

5. Storyboard.GameEngine/GameStateData/GameStateSession.cs
- Added in-place current action output update path.

6. Storyboard.GameEngine/GameServices/Commands/GameCommandProcessorService.cs
- Begin-action context now initializes empty action outputs at frame start.

7. Storyboard.GameEngine/GameServices/Actions/RuntimeCommandActionExecutor.cs
- Runtime script resolution path updates current-action output values when known.

8. Storyboard.GameEngine/GameServices/References/RuntimeSessionAnchorDataProvider.cs
- Added current-action output resolution path with temporary `action.` prefix compatibility strip.

9. Storyboard.GameEngine/GameServices/References/ScopeChainReferenceValueResolver.cs
- Emits current-action output aliases from `ActionOutputValues`.

10. Storyboard.GameEngine.Tests/RuntimeActionOutputVariableManifestParityTests.cs
- Added/maintained manifest-to-catalog parity guardrail.

11. Storyboard.GameEngine.Tests/RuntimeSessionAnchorDataProviderTests.cs
- Added/maintained canonical and compatibility alias coverage for current-action output lookup.

## Contract/Interface Impact

1. No external contract-breaking changes required for Wave 1 completion.
2. Compatibility alias retained temporarily to avoid abrupt breakage during migration.

## Validation Commands Executed

1. `dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~ActionVariableEmissionGuardrailTests|FullyQualifiedName~RuntimeActionOutputVariableManifestParityTests|FullyQualifiedName~RuntimeSessionAnchorDataProviderTests"`
- Result: PASS.

2. `dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~RuntimeMoveRoomObjectOnGridActionTests"`
- Result: PASS.

3. Prior focused regression confirmation in this stage stream:
- `dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~RuntimeActionOutputVariableManifestParityTests|FullyQualifiedName~RuntimeSessionAnchorDataProviderTests"`
- Result: PASS (14 passed, 0 failed).

## Test Results

1. Wave 1 focused runtime guardrails are green.
2. Move-family runtime action test suite is green.
3. No Wave 1 blocker failures remain for proceeding to Wave 2.

## Behavioral Notes

1. Current action outputs now follow action-frame lifecycle semantics (empty at frame start, populated when known).
2. Canonical token direction is `currentAction.<leafKey>` for move-family outputs.
3. Temporary compatibility with `currentAction.action.<leafKey>` remains in place pending Stage 07 retirement.

## Known Issues/Risks

1. Residual duplication risk remains between full-population resolver pipeline and single-path provider logic.
2. Legacy fallback path may still trigger full-map population in some flows until Stage 07 decisions are executed.
3. Removing compatibility aliases too early would risk regressions; retirement remains deferred.

## Boundary Compliance Report

1. Out-of-scope reads performed:
- None for this Wave 1 finalization update.
2. Out-of-scope edits performed:
- None.
3. Stage-boundary exceptions approved:
- None.
4. Session context scope notes:
- This handoff now records Wave 1 as complete and leaves Wave 2+ execution staged.

## Explicit Next-Stage Start Checklist

1. Start Wave 3 (`SetActiveRoomObject` + `ClearActiveRoomObjects`) in a fresh session using this handoff prompt.
2. Keep Wave 3 action-family scope strict; do not fold Stage 07 retirements into Wave 3.
3. Continue producing per-wave evidence packets with before/after token snapshots and parity results.
4. Preserve temporary compatibility aliases/fallbacks for non-retired families until Stage 07 gate.

## Wave 3-7 Completion Addendum

1. Scope statement:
- Completed Stage 05 Waves 3-7 action-family migration to manifest-backed action output declaration ownership and runtime parity:
	- Wave 3: `SetActiveRoomObject`, `ClearActiveRoomObjects`
	- Wave 4: `PutObjectInContainer`, `RemoveObjectFromContainer`
	- Wave 5: `OpenObject`, `CloseObject`, `LockObject`, `UnlockObject`
	- Wave 6: `BuildCompositeByTarget`, `BuildCompositeByParts`, `BreakCompositeItem`
	- Wave 7: `NavigateDirection`, `NavigateToAdjacent`, `InvokeProcedure`

2. Before/after token snapshot:
- Before: descriptor-registry-centric action output declarations with mixed casing and partial family coverage.
- After: `actionOutputVariableCatalog` in manifest and `RuntimeActionOutputVariableKeyCatalog` are authoritative for canonical leaf keys.
- Canonical authored/lookup direction remains `currentAction.<leafKey>` with temporary compatibility support for `currentAction.action.<leafKey>`.

3. Runtime emission parity proof:
- `RuntimeActionOutputVariableManifestParityTests` now compares manifest output-key declarations to key-catalog declarations.
- `ActionVariableEmissionGuardrailTests` validates that emitted action output tokens for migrated families are declared by catalog ownership.
- `RuntimeSessionAnchorDataProviderTests` covers canonical and compatibility alias resolution for migrated families.

4. Unknown-reference/script validation proof:
- Designer script unknown-reference and token-provider suites validate action token availability/ordering after migration.
- Compatibility registry bridge preserves designer token surfaces while runtime declaration authority stays in manifest/key-catalog.

5. Wave-local cleanup performed:
- Removed runtime dependency on legacy descriptor-type declaration system for guardrail declaration ownership.
- Runtime declaration flow is now keyed by canonical output-key ownership in `RuntimeActionOutputVariableKeyCatalog`.
- Added compatibility bridge `RuntimeActionVariableDescriptorRegistry` to preserve designer/test token contract while broader retirement sequencing is stabilized.

6. Deferred/shared retirement notes:
- Full removal of compatibility bridge and temporary alias paths remains a Stage 07 retirement concern once all authored references are validated canonical-only.
- Keep/remove decisions for legacy full-map fallback behavior remain tracked for Stage 07 closeout.

7. Rollback note:
- Revert Wave 3-7 by reverting only Wave 3-7 family catalog/emission changes and the related test updates; keep Wave 1-2 and prior stage groundwork intact.

## Wave 3-7 Validation Commands Executed

1. `dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~ActionVariableEmissionGuardrailTests|FullyQualifiedName~RuntimeActionOutputVariableManifestParityTests|FullyQualifiedName~RuntimeSessionAnchorDataProviderTests|FullyQualifiedName~RuntimeSetActiveRoomObjectActionTests|FullyQualifiedName~RuntimeClearActiveRoomObjectsActionTests|FullyQualifiedName~RuntimeOpenObjectActionTests|FullyQualifiedName~RuntimeLockObjectActionTests|FullyQualifiedName~RuntimeUnlockObjectActionTests|FullyQualifiedName~RuntimeInvokeProcedureActionTests|FullyQualifiedName~BuildComposite|FullyQualifiedName~BreakComposite|FullyQualifiedName~NavigateDirection|FullyQualifiedName~NavigateToAdjacent|FullyQualifiedName~SetActiveRoomObjectActionVariableResolverTests|FullyQualifiedName~OpenObjectActionVariableResolverTests|FullyQualifiedName~LockObjectActionVariableResolverTests|FullyQualifiedName~UnlockObjectActionVariableResolverTests"`
- Result: PASS (101 passed, 0 failed, 0 skipped).

2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~ActionEchoReferenceTokenProviderRegistryTests|FullyQualifiedName~ScriptUnknownReferenceRuleTests|FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`
- Result: PASS (61 passed, 0 failed, 0 skipped).

3. `dotnet test .\Storyboard.TransportCodegen.Tests\Storyboard.TransportCodegen.Tests.csproj`
- Result: PASS (9 passed, 0 failed, 0 skipped).

4. `dotnet build .\StoryboardDesigner.slnx`
- Result: PASS (0 errors).
