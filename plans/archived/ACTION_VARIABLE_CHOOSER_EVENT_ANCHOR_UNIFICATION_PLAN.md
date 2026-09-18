# Action Variable Chooser Event Anchor Unification Plan

Last updated: 2026-09-14
Status: Closeout Prep (Stage 06 skipped by design, Stage 07 implementation complete, archive pending)

Purpose: define a safe, staged path to unify action/output variable discovery, anchor-based references, chooser UX, and script validation under a single supportable metadata model, while preserving runtime behavior and avoiding all-at-once migration risk.

## Intent And Timing

1. This plan is now execution-prepared for a two-action pilot and staged rollout.
2. Current high-priority delivery tracks remain unblocked.
3. This plan now includes explicit provider ownership, parity gates, and retirement targets so stage handoffs can start without re-discovery.

## Runtime Provider Trace (Confirmed)

Primary question answered: the current runtime provider mapping `sourcePath` to values is `RuntimeEventPayloadBuilder`.

Concrete chain:
1. `RuntimeEventPayloadBuilder.Build` loads per-event payload mappings from `Config/event-payload.manifest.json`.
2. `TryResolvePayloadProperty` parses each mapping `sourcePath` into a resolution plan.
3. Object resolution uses `RuntimeObjectPathResolverPipeline`:
- `RuntimeObjectPathStartContextStage` for anchor roots (`anchor::`).
- `RuntimeObjectPathProjectionStage` for projected object tokens.
- `RuntimeObjectPathStructuralTraversalStage` for hierarchy traversal.
4. Anchor roots resolve through `RuntimeAnchorObjectResolver` (`currentPlanet`, `currentCountry`, `currentArea`, `currentRoom`, `activePlayer`, `currentCommand`, `currentAction`).
5. Leaf variable resolution uses `RuntimeObjectNodeLeafVariableResolver`; special leafs `id` and `nameInGame` are handled directly.

Important nuance:
1. `RuntimeEventPayloadBuilder` currently hardcodes known anchors and includes compatibility normalization (`currentRoom.activePlayer.* -> activePlayer.*`).
2. This means current anchor metadata is event-centric and not yet promoted to a session-wide foundation.
3. Target state is a shared anchor provider (`RuntimeSessionAnchorDataProvider`) backed by a dedicated session anchor manifest, then reused by both event payload and action script/runtime reference pipelines.

## Divergence Baseline (Expanded)

### Event Anchor Authoring/Validation
1. Designer reads anchor keys and `supportedSubProperties.propertyKey` from event manifest data in multiple places.
2. Validation currently checks anchor + subproperty key declaration, but does not validate runtime-resolvable `sourcePath` semantics end-to-end.
3. Multiple duplicated JSON readers exist across designer surfaces for event keys, anchors, and mappings.

### Action Output Variables
1. Canonical action output declarations now live in `RuntimeActionOutputVariableKeyCatalog` as a per-action manifest mirror of `actionOutputVariableCatalog.actionOutputVariableKey`.
2. Designer action token providers and payload helpers mostly proxy this registry.
3. `action-payload.manifest.json` has broad `actionPayloadArgumentCatalog` usage in event-input mapping UX; `actionOutputVariableCatalog` is now partially populated for move actions and remains incomplete for most other actions.

### Script Token Composition
1. `ScriptUnknownReferenceRule` merges:
- Project-known variable tokens.
- Action payload reference tokens.
- Action echo provider tokens.
2. This is useful but still multi-source and drift-prone.

### Legacy/Dead-Code Signals
1. `RuntimeProjectedAnchorObjectResolver` is flagged in-code as partial/legacy and appears test-only in active usage.
2. Hardcoded runtime keys in `RuntimeActionInputArgumentPolicy` mirror manifest catalog intent but are not generated/validated against manifest.

## Problem Statement

Variable discoverability and usage are split between:
1. Manifest-driven anchor subproperty suggestions.
2. Code-driven action output registries.
3. Project graph variable discovery.

Result:
1. Users must browse too often for values that should be obvious from anchors.
2. Action-variable complexity remains high and difficult to evolve safely.
3. Validation, chooser UX, and runtime truth can drift independently.

## Non-Goals

1. No hidden grammar expansion, implicit fallback verbs, or synonym magic.
2. No runtime/designer host-boundary violations.
3. No big-bang replacement of all actions in one pass.
4. No broad scripting language redesign in this workstream.

## Desired Outcome

1. One normalized metadata catalog feeding chooser + validation (and eventually runtime declaration checks).
2. Session anchors become a first-class foundational contract, independent from event payload mapping definitions.
3. Action-specific output variables declared in manifest catalog over time, one action at a time.
4. Explicit compatibility window with parity telemetry before retiring legacy paths.

## Anchor Foundation Decision (Locked For This Plan)

1. Extract anchors from `Storyboard.GameEngine/Config/event-payload.manifest.json` into a new dedicated manifest: `Storyboard.GameEngine/Config/session-anchordata.manifest.json`.
2. Keep `event-payload.manifest.json` focused on event keys and payload mappings only.
3. Introduce a shared runtime provider (proposed: `RuntimeSessionAnchorDataProvider.cs`) that loads session anchor metadata and resolves anchor-backed values.
4. Make this provider a reusable dependency for:
- event payload building.
- action script/reference value resolution.
- designer validation/chooser metadata consumption through shared contract readers.
5. Preserve existing authored syntax and behavior during migration; no grammar expansion.

## Canonical Model Proposal (Draft)

Use one normalized entry shape for chooser/validation/runtime declaration:
1. `tokenKey`: final script token (`action.moveDistance`, `currentAction::scope.nameInGame`).
2. `sourceExpression`: canonical origin expression (`sourcePath` or descriptor expression).
3. `sourceKind`: `EventAnchor`, `ActionOutput`, `ScopedVariable`, `LegacyAlias`.
4. `actionType`: optional action discriminator for action outputs.
5. `anchorKey`: optional for anchor-derived tokens.
6. `subPropertyKey`: optional for anchor subproperty derivation.
7. `valueType`: optional (`string`, `int`, `bool`, `guid`, `list`).
8. `availability`: optional context (`Always`, `ActionSuccess`, `ActionFailure`, `EventOnly`, etc.).
9. `aliases`: compatibility list.
10. `deprecation`: optional metadata (`isDeprecated`, `replacementToken`).

Default canonical expression policy:
1. Canonical runtime/object resolution source remains `sourcePath` semantics.
2. Chooser can display friendlier labels while preserving exact `tokenKey` insertion.

## Migration Strategy (One Action At A Time)

## Locked Stage Order

1. Stage 00: Inventory/Drift Mapping.
2. Stage 01: Session Anchor Manifest Extraction.
3. Stage 02: Runtime Engine Move Pilot (`MoveRoomObjectOnGrid` plus `MoveRoomObjectByPoints` parity).
4. Stage 03: Shared Anchor Provider And Reader Unification.
5. Stage 04: Designer Anchor Path And Picker Unification.
6. Stage 05: Action Output Catalog Migration Waves.
7. Stage 06: Contract Retirement Gate (Optional/Skipped unless contract shape changes are introduced).
8. Stage 07: Legacy Retirement Cleanup And Closeout.

## Stage Inclusion Matrix

1. Stage 00 Inventory/Drift Mapping: Required.
2. Stage 01 Session Anchor Manifest Extraction: Required.
3. Stage 02 Runtime Engine Move Pilot: Required.
4. Stage 03 Shared Anchor Provider And Reader Unification: Required.
5. Stage 04 Designer Anchor Path And Picker Unification: Required.
6. Stage 05 Action Output Catalog Migration Waves: Required.
7. Stage 06 Contract Retirement Gate: Skipped by default for this workstream unless contract schema/interface retirement is explicitly introduced.
8. Stage 07 Legacy Retirement Cleanup And Closeout: Required.

### Stage 0: Inventory/Drift Mapping (No Behavior Change)
Tasks:
1. Produce token source inventory across:
- Session anchor manifest candidates currently inside event manifest.
- Event payload mappings.
- Runtime action descriptor registry.
- Designer provider registry/payload token hints.
- Script known-token builder.
2. Generate drift matrix:
- Missing by source.
- Case/alias mismatches.
- Runtime-declared but non-discoverable tokens.
3. Tag each token:
- `Canonical`.
- `CompatibilityAlias`.
- `CandidateRemoval`.

Deliverables:
1. Drift report markdown in plan handoff.
2. Machine-readable baseline snapshot for regression comparison.
3. Handoff output path: `plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_00_INVENTORY_HANDOFF.md`.

### Stage 1: Session Anchor Manifest Extraction (No Behavior Change)

Tasks:
1. Create `Storyboard.GameEngine/Config/session-anchordata.manifest.json` with anchor definitions currently in event manifest (`anchorKey`, `supportedSubProperties`, `sourcePath`, optional type metadata).
2. Remove `anchors` block from `event-payload.manifest.json` after compatibility readers are in place.
3. Add compatibility loader support so runtime/designer can read from new manifest first, then fallback to legacy location for one migration window.
4. Add parity tests asserting extracted anchor data is byte-equivalent in meaning to prior event-manifest anchors.

Deliverables:
1. Dedicated session anchor manifest with guardrail coverage.
2. Event payload manifest narrowed to event concerns.
3. Handoff output path: `plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_01_DESIGNER_DISCOVERY_HANDOFF.md`.

### Stage 02 Runtime Engine Pilot (Single-Action End-To-End)

Goal:
1. Prove one action end-to-end through runtime engine using manifest-canonical action output metadata before broad multi-action rollout.
2. Lock a repeatable runtime migration recipe that later action waves can follow.

Pilot action scope:
1. Primary: `MoveRoomObjectOnGrid`.
2. Parity companion: `MoveRoomObjectByPoints` only where it shares move-output plumbing.

Primary output handoff path:
1. `plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_02_RUNTIME_ENGINE_MOVE_PILOT_HANDOFF.md`

Runtime code targets for this stage:
1. `Storyboard.GameEngine/GameServices/Actions/GameActions/RuntimeActionExecutable/RuntimeCommandActionExecutor.MoveRoomObjectOnGridExecutableAction.cs`
2. `Storyboard.GameEngine/GameServices/References/MoveRoomObjectOnGridActionVariableResolver.cs`
3. `Storyboard.GameEngine/GameServices/RuntimeContext/RuntimeMoveRoomObjectOnGridAttemptResult.cs`
4. `Storyboard.GameEngine/GameServices/Commands/GameCommandMoveLegTelemetry.cs`
5. `Storyboard.GameEngine/GameManager/GameManager.cs`
6. `Storyboard.GameEngine/Config/action-payload.manifest.json`

Required implementation outcomes:
1. Ensure runtime-emitted `action.*` values for move actions are aligned to manifest-declared keys.
2. Add explicit runtime support fields for bottom/support object context for move-driven stack landings (fast follower listed in this plan) once move pipeline parity is proven.
3. Keep compatibility behavior for legacy token consumption during transition (no abrupt parser/grammar changes).
4. Document any `MoveRoomObjectByPoints` coupling to `MoveRoomObjectOnGrid` descriptors and either preserve intentionally or split with explicit rationale.

Required validation for this stage:
1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests|SchemaCodegenHardcodedDtoGuardrailsTests|SchemaEmittedContractDriftGuardrailsTests|TransportArtifactGuardrailsTests"`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "EventSubscription|VariableChoices"`
4. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`

Stage boundary allowlist for this runtime pilot:
1. Allowed read scope:
- `plans/**`
- `Storyboard.GameEngine/**`
- `StoryboardDesigner.App/**` (read-only for token consumer context)
- `StoryboardDesigner.App.Tests/**` (read-only for test target selection)
2. Allowed edit scope:
- `Storyboard.GameEngine/**`
- `Storyboard.GameEngine.Tests/**`
- `plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md`
- `plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_02_RUNTIME_ENGINE_MOVE_PILOT_HANDOFF.md`

### Stage 03: Shared Anchor Provider And Reader Unification (No Behavioral Drift)
Tasks:
1. Introduce a shared manifest/catalog reader service used by both runtime and designer for metadata parsing.
2. Introduce `RuntimeSessionAnchorDataProvider.cs` for centralized anchor metadata and resolution.
2. Route `RuntimeEventPayloadBuilder` anchor resolution through `RuntimeSessionAnchorDataProvider`.
3. Route action reference/value pipelines to consume the same provider for anchor-backed values (including current command/action scope anchors).
4. Replace duplicated designer anchor JSON readers with shared reader/adaptor fed by the same manifest contract.
5. Add guardrail tests for deterministic ordering, schema validity, and provider parity against existing behavior.
Deliverables:
1. Shared metadata service contract.
2. Single reusable session anchor provider contract.
2. Green parity tests with no behavior deltas.
3. Handoff output path: `plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_03_SHARED_PROVIDER_HANDOFF.md`.

### Stage 04: Designer Anchor Path And Picker Unification
1. Remove scattered anchor manifest readers in designer and route through shared service.
2. Align anchor/subproperty validation to resolved canonical metadata (not ad hoc extraction).
3. Add runtime/design-time consistency tests for each anchor subproperty mapping.
4. Keep existing syntax acceptance (`anchor::subProperty.variable`) during migration.
5. Add explicit validation severity policy for anchor reference paths:
- Hard error when a reference path is definitively invalid at design time (for example unknown anchor root, malformed syntax, or unsupported declared subproperty).
- Soft finding when the path is partially dynamic and cannot be proven invalid at design time after a valid root/subproperty chain.
6. Route soft findings to validation report output written to disk and avoid save-time popup surfacing for those soft findings.

Deliverables:
1. Single source for anchor metadata in designer/runtime.
2. Compatibility tests proving no regression for existing authored references.
3. Validation severity policy tests proving hard vs soft classification behavior for anchor paths.
4. Soft anchor validation findings are persisted to disk report output and excluded from save-time popup triggers.
3. Handoff output path: `plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_04_DESIGNER_UNIFICATION_HANDOFF.md`.

### Stage 05: Action Output Catalog Migration Waves
Core rule: migrate one action type per wave, prove parity, then proceed.

Per-action wave checklist:
1. Populate `actionOutputVariableCatalog` entry for the selected action.
2. Add parity test against `RuntimeActionOutputVariableKeyCatalog` canonical keys.
3. Route chooser suggestions for that action through manifest-backed catalog first.
4. Keep runtime descriptor fallback active until parity confidence gate passes.
5. Validate script unknown-reference behavior for migrated action tokens.
6. Validate runtime emitted-token legality guardrail still passes.
7. Record migration handoff with before/after token set snapshot.

Suggested wave order (high value, lowest ambiguity first):
1. `MoveRoomObjectOnGrid` (already tied to rich event payload paths).
2. Fast follower after Wave 1 parity: add explicit bottom/support-object outputs for move-driven stacking (`action.stackTargetObjectName`, `action.stackTargetScopeNodeId`) only after the new move pipeline migration is complete and proven.
3. `StackRoomObjectOnAnother`.
4. `SetActiveRoomObject` and `ClearActiveRoomObjects`.
5. `PutObjectInContainer` and `RemoveObjectFromContainer`.
6. `OpenObject`, `CloseObject`, `LockObject`, `UnlockObject`.
7. Composite actions (`BuildCompositeByTarget`, `BuildCompositeByParts`, `BreakCompositeItem`).
8. Navigation/procedure actions.

Cutover rule:
1. Do not retire descriptor-registry source until all actions are migrated and drift checks are green for at least one full validation cycle.
2. Handoff output path: `plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_05_ACTION_WAVES_HANDOFF.md`.

### Stage 06: Contract Retirement Gate (Optional/Skipped)

1. Default state for this workstream: skipped unless explicit contract/schema/interface retirement is introduced.
2. If activated, constrain Stage 06 to approved contract retirement only and run contract/transport guardrails before Stage 07.
3. Handoff output path: `plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_06_CONTRACT_RETIREMENT_HANDOFF.md`.

### Stage 07: Legacy Retirement Cleanup And Closeout
Targets to retire or reduce:
1. Action-token duplication paths that become manifest-backed.
2. Legacy descriptor/registry declarations used as canonical key ownership once all action families are migrated to `RuntimeActionOutputVariableKeyCatalog`.
2. Redundant per-view manifest readers in designer.
3. Legacy/partial projected resolver path if confirmed not production-reachable (`RuntimeProjectedAnchorObjectResolver`).
4. Hardcoded action input argument keys that can be catalog-validated/generated.
5. Legacy fallback for reading anchors from `event-payload.manifest.json` once all consumers are migrated.
6. ScopeChain pipeline consolidation review is deferred to `plans/active/SCOPECHAIN_PIPELINE_CONSOLIDATION_POST_CLOSEOUT_PLAN.md` and is not a blocker for this workstream closeout.

Retirement gate:
1. Contract guardrails + runtime-focused test suite + replay regression must be green.
2. Handoff output path: `plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_07_ACTION_TOKEN_LEGACY_RETIREMENT_HANDOFF.md`.

## UX Direction (Chooser Behavior)

1. Quick suggestions remain intentionally concise.
2. Add a curated anchor shortlist for high-frequency context keys (for example `currentAction::scope.*`, `currentCommand::primaryCommandObject.*`).
3. Full chooser remains exhaustive and grouped by source kind.
4. Always display provenance in chooser rows (`Anchor`, `Action Output`, `Project Variable`, `Alias`).

## Candidate File Targets (Implementation)

1. `Storyboard.GameEngine/Config/session-anchordata.manifest.json` (new)
2. `Storyboard.GameEngine/Config/event-payload.manifest.json`
3. `Storyboard.GameEngine/Config/action-payload.manifest.json`
4. `Storyboard.GameEngine/GameServices/References/RuntimeSessionAnchorDataProvider.cs` (new)
5. `Storyboard.GameEngine/GameServices/Events/RuntimeEventPayloadBuilder.cs`
6. `Storyboard.GameEngine/GameServices/References/RuntimeReferenceValueResolverPipeline.cs`
7. `Storyboard.GameEngine/GameServices/References/ScopeChainReferenceValueResolver.cs`
8. `Storyboard.GameEngine/GameServices/References/RuntimeActionOutputVariableKeyCatalog.cs`
9. `Storyboard.GameEngine/GameServices/References/RuntimeActionVariableDescriptors.cs`
10. `Storyboard.GameEngine/GameServices/References/RuntimeActionVariableDescriptorRegistry.cs`
10. `StoryboardDesigner.App/Validation/Rules/Project/EventSubscriptionRuleSupport.cs`
11. `StoryboardDesigner.App/Validation/Rules/Scripting/ScriptRuleSupport.cs`
12. `StoryboardDesigner.App/Services/ActionEchoReferenceTokenProviderRegistry.cs`
13. `StoryboardDesigner.App/Views/EventSubscriptionEditorDialog.xaml.cs`
14. `StoryboardDesigner.App/Views/EventInputArgumentMappingsDialog.xaml.cs`
15. `StoryboardDesigner.App/ViewModels/MainWindowViewModel.ProjectExplorer.cs`
16. `StoryboardDesigner.App.Tests/*VariableChoices*`
17. `StoryboardDesigner.App.Tests/*EventSubscription*`
18. `Storyboard.GameEngine.Tests/*Reference*` and event payload mapping tests

## Validation Gates (Per Wave)

1. Build:
- `dotnet build .\StoryboardDesigner.slnx`
2. Runtime-focused regression gate:
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests|SchemaCodegenHardcodedDtoGuardrailsTests|SchemaEmittedContractDriftGuardrailsTests|TransportArtifactGuardrailsTests"`
3. Replay safety gate:
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`
4. Contract/transport gate when manifest contracts shift:
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "SchemaCodegenHardcodedDtoGuardrailsTests|SchemaEmittedContractDriftGuardrailsTests|ArchitectureSeparationGuardrailsTests|TransportArtifactGuardrailsTests"`
- `dotnet test .\Storyboard.TransportCodegen.Tests\Storyboard.TransportCodegen.Tests.csproj`

## Risks And Mitigations

1. Risk: hidden runtime dependencies on legacy aliases.
Mitigation: explicit alias inventory + deprecation metadata + telemetry before removal.
2. Risk: chooser noise from exhaustive catalogs.
Mitigation: strict quick-list policy and grouped full chooser.
3. Risk: manifest/catalog drift.
Mitigation: parity tests and build-breaking drift guardrails.
4. Risk: accidental boundary coupling.
Mitigation: keep shared metadata logic in shared/runtime-safe layer; keep UI concerns in designer only.

## Entry Criteria To Promote To Active Implementation

1. Stage 0 inventory + drift matrix completed.
2. Canonical token model approved.
3. First wave action type selected.
4. Parity and rollback gates agreed.

## Immediate Next Steps (When Triggered)

1. Start Stage 0 drift inventory and publish handoff report.
2. Execute Stage 1 extraction: author `session-anchordata.manifest.json` and introduce fallback reader.
3. Execute Stage 02 runtime engine pilot: complete one end-to-end move action runtime slice and publish Stage 02 handoff.
4. Execute Stage 03 provider wiring: land `RuntimeSessionAnchorDataProvider` and route event payload builder first.
5. Execute Stage 04 designer unification for three-lane discovery from one script entry flow.
6. Execute Stage 05 action output waves starting with `MoveRoomObjectOnGrid`.
7. Execute Stage 07 action-type-scoped retirement and closeout (Stage 06 only if contract retirement is activated).

## Closeout Prep Snapshot

1. Stage 05 is complete (waves 1-7 complete).
2. Stage 06 is complete as skipped/no-op by design.
3. Stage 07 implementation scope is complete; final archival unit move remains.
4. Deep scopechain/pipeline architecture review is deferred to follow-up plan:
- `plans/active/SCOPECHAIN_PIPELINE_CONSOLIDATION_POST_CLOSEOUT_PLAN.md`