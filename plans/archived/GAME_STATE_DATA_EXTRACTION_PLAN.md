# Game State Data Extraction Plan

## Goal
Decouple runtime game state memory/scope-tree implementation from designer model types, so runtime state can live in Shared and operate on runtime-native contracts only.

## Execution Status
- [x] GSD-1 contract scaffolding created in Shared.
- [x] GSD-2 runtime property state port.
- [x] GSD-3 runtime scope node port.
- [x] GSD-4 runtime session port.
- [x] GSD-5 app mapper/adapter integration.
- [x] GSD-6 cleanup and guardrails.

Latest validation after full six-phase pass:
- dotnet build .\\StoryboardDesigner.slnx: passed
- focused regression gate (33 tests): passed

## Scope
In scope:
- Runtime state tree nodes and variable state/value runtime types.
- Runtime session behavior (navigation, variable get/set, object/container movement, capacity math).
- Runtime object identity and deterministic object id generation.
- Adapter seams from app model/runtime command bridge into runtime-native session.

Out of scope (for this plan):
- Designer editing model changes.
- WPF view model/UI refactors beyond compile adaptation.
- JSON schema redesign (consume existing bootstrap export).

## Current Coupling Inventory (Must Remove)
Primary model-coupled file:
- StoryboardDesigner.App/GameStateData/GameStateSession.cs

Direct designer model dependencies currently used by runtime state:
- ProjectModel
- GameObject
- IScopedAwareNode
- ScopeNodeKind
- GamePropertyDefinition and property enums from app model namespace

Secondary coupling points that will need adapters after extraction:
- StoryboardDesigner.App/GameManager/GameManager.cs
- StoryboardDesigner.App/GameServices/AppRuntimeCommandModelBridge.cs
- StoryboardDesigner.App/GameServices/RuntimeScopeMutationGatewayAdapter.cs
- StoryboardDesigner.App/ViewModels/MainWindowViewModel.GameSimulator.cs

## Target Architecture
Shared owns runtime state:
- Storyboard.Shared/GameStateData/RuntimeGameStateSession.cs
- Storyboard.Shared/GameStateData/RuntimeGameStateScopeNode.cs
- Storyboard.Shared/GameStateData/RuntimePropertyState.cs
- Storyboard.Shared/GameStateData/RuntimePropertyValue.cs
- Storyboard.Shared/GameStateData/RuntimeObjectIdentity.cs (already in Shared)

Shared contracts for runtime state construction and command-time lookup:
- RuntimeGameWorldSnapshot (runtime-native world graph input)
- RuntimeScopeNodeDescriptor (kind/name/tokens/verbs/directionals/ids)
- RuntimePropertyDefinition (name/default/lifetime/restriction)
- RuntimeScopeReference (object id, optional parent context, scope kind)

Primary node contract direction:
- Runtime traversal/execution uses IRuntimeScopeNode.
- Runtime mutation/session APIs migrate from IScopedAwareNode + ScopeNodeKind to IRuntimeScopeNode and/or RuntimeScopeReference + RuntimeScopeKind.
- IScopedAwareNode remains app-side only as a mapping concern.

App owns mapping only:
- ProjectModel -> RuntimeGameWorldSnapshot mapper
- Scoped-aware node -> RuntimeScopeReference adapter
- UI projection adapters for tree display

## Extraction Phases

### Phase GSD-1: Introduce Runtime-Native Contracts in Shared
Deliverables:
- Add RuntimePropertyLifetime and RuntimePropertyValueRestriction enums in Shared.
- Add RuntimePropertyDefinition record in Shared.
- Add RuntimeScopeNodeDescriptor record in Shared.
- Add RuntimeGameWorldSnapshot record (global vars, player objects, planet tree, start location, player character name).
- Add RuntimeScopeReference record for mutation/lookups.

Acceptance:
- No app model types referenced by new Shared contracts.
- Shared builds.

### Phase GSD-2: Move State Primitives to Shared
Deliverables:
- Move/port GamePropertyRuntimeValue -> RuntimePropertyValue in Shared.
- Move/port GamePropertyState -> RuntimePropertyState in Shared.
- Update validation logic (numeric/true-false/unrestricted) to use Shared enums.

Acceptance:
- Property state tests pass (new tests in StoryboardDesigner.App.Tests can still target Shared types).

### Phase GSD-3: Move Scope Node to Shared
Deliverables:
- Move/port GameStateScopeNode -> RuntimeGameStateScopeNode in Shared.
- Node implements IRuntimeScopeNode from Shared runtime contracts.
- Remove all references to app model property types.

Acceptance:
- Scope enter/leave transient behavior preserved.
- Deterministic object ids still generated via RuntimeObjectIdentity.

### Phase GSD-4: Move Session Core to Shared
Deliverables:
- Move/port GameStateSession -> RuntimeGameStateSession in Shared.
- Constructor/factory takes RuntimeGameWorldSnapshot, not ProjectModel.
- Container transfer APIs accept RuntimeScopeReference (or runtime object id) instead of IScopedAwareNode.
- Keep capacity metrics and behavior parity.

Acceptance:
- Existing simulator regression scenarios still pass.
- No app model namespaces imported by RuntimeGameStateSession.

### Phase GSD-5: App Mapper + Bridge Integration
Deliverables:
- Add app mapper: ProjectModelToRuntimeGameWorldSnapshotMapper.
- Add app scope resolver: ScopedAwareNodeToRuntimeScopeReferenceAdapter.
- Update GameManager and command bridges to call RuntimeGameStateSession via runtime-native contracts.

Acceptance:
- App compiles with GameStateData runtime types sourced from Shared.
- Command processing + linked actions tests pass.

### Phase GSD-6: Cleanup and Guardrails
Deliverables:
- Delete obsolete app-local runtime state implementations once migrated.
- Add architecture guardrail test: app GameStateData namespace cannot reference designer model contracts for runtime execution state.
- Update extraction tracker docs.

Acceptance:
- Focused regression gate green.
- Architecture guardrail passes.

## Detailed Task Backlog (Execution Order)
1. Create Shared runtime state contract files (GSD-1).
2. Add contract mapping helpers from existing bootstrap DTOs (GSD-1/GSD-5 bridge).
3. Port property runtime types (GSD-2).
4. Port scope node type with unchanged behavior (GSD-3).
5. Port session logic using runtime contracts (GSD-4).
6. Wire app mappers/adapters (GSD-5).
7. Switch GameManager/session call sites to new Shared runtime session (GSD-5).
8. Remove obsolete app-local runtime state files and update tests/guardrails (GSD-6).

## Validation Gate (Run Every Phase)
- dotnet build .\StoryboardDesigner.slnx
- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"

## Risk and Mitigation
Risk: Runtime behavioral drift in container capacity/load math.
Mitigation: Keep method-level parity during port; add focused unit tests around GetEffectiveInventoryLoadToParent and transfer checks.

Risk: Scope resolution mismatches after replacing IScopedAwareNode input.
Mitigation: Introduce RuntimeScopeReference adapter in app first, then swap session signatures.

Risk: UI tree depends on app enum types.
Mitigation: Add small enum mapping layer in view-model projection, keep UI contract stable until final cleanup.

## Definition of Done
- Runtime game state memory tree executes fully from Shared runtime contracts.
- No ProjectModel, GameObject, IScopedAwareNode, or ScopeNodeKind dependency in runtime state engine.
- App layer is only responsible for mapping and presentation.
- Focused regression gate remains green.
