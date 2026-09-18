# Global Objects Scope Model Consolidation Plan

## 0. Purpose
Consolidate global-object authoring state to a single owner in `ProjectModel` and retire `GlobalObjectsScope` as a separate data layer.

Primary intent:
1. Eliminate duplicate-model appearance where `ProjectModel` and `GlobalObjectsScope` appear to own the same data.
2. Keep behavior stable while migrating in small, reversible slices.
3. Preserve load/save compatibility and existing editor workflows.
4. Keep "Import Globals" viable throughout migration, including merge/update behavior for global objects, actions, and variables.

## 1. Current Problem
Today, `ProjectModel` exposes global-object members through compatibility passthrough properties backed by `GlobalObjectsScope`.

This creates drift/confusion:
1. It appears there are two owners of global-object state.
2. Scope semantics and storage ownership are mixed.
3. Refactors become riskier because many surfaces read via one type and write via another.

## 2. Target End State
1. `ProjectModel` is the only data owner for global-object state.
2. Global-object state lives directly on `ProjectModel` fields/properties:
- top-level `GameObjects`
- global-object variables
- global-object available actions
- global-object ignored validation rule ids
- global-object display metadata (name/notes)
3. `GlobalObjectsScope` no longer owns persistent/editor model data.
4. Validation/hierarchy/path aliases remain behavior-compatible through migration.

## 3. Decision Locks
1. **Storage ownership now**: move to `ProjectModel` immediately (safe first objective).
2. **Scope-node root replacement later**: do not make `ProjectModel : ScopeNodeBase` in the first slices.
3. **Compatibility aliases retained during migration**: keep legacy scope path aliases (including `Global / Player`) until final hard cleanup.
4. **No contract/schema changes in this effort**: this is model-layer only.

## 4. Non-Goals
1. No runtime contract enum retirement in this plan.
2. No command grammar changes.
3. No broad UI redesign.
4. No unrelated serializer format revisions beyond compatibility-preserving mapping updates.
5. No redesign of "Import Globals" UX/semantics beyond corrections required to preserve current behavior.

## 5. Phased Execution

### Phase A: Baseline + Guardrails
1. Capture all references to `GlobalObjectsScope` and `project.GlobalObjectsScope` in app and tests.
2. Capture behavior baseline with focused tests.
3. Add/confirm guardrail tests that lock current load/save + validation suppression behavior for global scopes.
4. Add/confirm guardrail coverage for importing globals from another project (including object/action/variable transfer and ignored-rule mapping).

Exit criteria:
1. Reference inventory complete.
2. Baseline tests green.
3. Import Globals baseline behavior documented and test-locked.

### Phase B: ProjectModel Single Storage Ownership
1. Add direct backing storage in `ProjectModel` for all global-object state currently backed by `_globalObjectsScope`.
2. Repoint public properties (`GameObjects`, `GlobalObjectVariables`, `GlobalObjectAvailableActions`, `GlobalObjectIgnoredValidationRuleIds`, scope name/notes) to direct fields.
3. Keep `GlobalObjectsScope` property temporarily as compatibility façade, but no independent storage.

Exit criteria:
1. One real in-memory copy of each global-object datum.
2. No behavior drift in load/save/edit flows.

### Phase C: Scope Adapter Realignment
1. Convert `GlobalObjectsScope` to a thin adapter over `ProjectModel` storage, or replace usages with a dedicated non-owning adapter type.
2. Ensure hierarchy/validation traversal still sees the same node identities/paths during transition.
3. Keep legacy aliases (`Global / Player`) while migration is active.

Exit criteria:
1. Zero data ownership in `GlobalObjectsScope`.
2. Scope traversal and suppression behavior unchanged.

### Phase D: Service/ViewModel Migration
1. Migrate services/viewmodels from direct `GlobalObjectsScope` data access to `ProjectModel` owned members.
2. Keep compatibility shims only where needed to avoid large blast-radius changes.
3. Prioritize high-traffic surfaces first:
- project explorer/hierarchy
- validation lookup/issue processing
- selection option discovery
- import/export orchestration
4. Include explicit migration checks for Import Globals command flow to ensure imported global objects/actions/variables/ignored-rules continue to map to the consolidated `ProjectModel` storage.

Exit criteria:
1. Core workflows no longer depend on `GlobalObjectsScope` as data source.
2. Manual smoke checks pass.
3. Import Globals workflow passes automated and manual checks.

### Phase E: Remove Compatibility Seam
1. Remove M1 seam comments and temporary passthrough compatibility members from `ProjectModel`.
2. Remove `GlobalObjectsScope` type when no active references remain.
3. Remove stale path/alias handling only after confirming no legacy dependency in tests/fixtures.

Exit criteria:
1. `GlobalObjectsScope` removed from active product code.
2. `ProjectModel` is sole owner.

### Phase F (Optional, Deferred): ProjectModel As ScopeNodeBase
1. Evaluate making `ProjectModel` itself a true scope node (`ScopeKind.Global`) only after consolidation is stable.
2. If adopted, migrate from `ProjectGlobalScopeNode` adapter safely in separate slices.

Exit criteria:
1. Decision documented.
2. If implemented, hierarchy/validation parity confirmed.

## 6. Slice Strategy
For each phase, execute in small slices:
1. One small edit cluster.
2. Run focused tests.
3. Evaluate diffs/behavior.
4. Continue only if green.

Phase boundary hard gate:
1. At the end of each phase, run the full test suite for the solution.
2. Do not begin the next phase unless full tests pass.
3. If full tests fail, stop, fix regressions, and re-run full tests before advancing.

Rollback policy:
1. Revert only current slice if regressions appear.
2. Do not stack unrelated changes in the same slice.

## 7. Validation Gates
Run after each meaningful slice:
1. `dotnet build .\\StoryboardDesigner.slnx`
2. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "JsonExportServiceProjectStateTests|JsonExportServiceRuntimeExportTests|JsonExportServiceRuntimeExportSnapshotTests|JsonExportServiceHideEmptyConfigurationPersistenceTests"`
3. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "ValidationEngineRegistrationTests|ValidationIssueRunProcessorTests|SinglePlayerMarkerRuleTests|RuntimeExportIdUniquenessRuleTests|DuplicateNameInScopeRuleTests"`
4. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "MainWindowViewModelImportGlobalsCommandTests|RoomTreeTraversalProjectionTests|GameObjectSelectionOptionDiscoveryServiceTests"`
5. If new import-focused regressions are identified, add targeted Import Globals tests to this gate before continuing to the next slice.

Mandatory phase-end hard gate (all tests):
1. `dotnet test .\\StoryboardDesigner.slnx`
2. Required between every phase transition (A->B, B->C, C->D, D->E, E->F).
3. No phase transition is allowed on a red full-suite run.

## 8. Manual Regression Checklist
1. Load existing project with legacy global-object data and verify successful hydration.
2. Create/rename/delete top-level global object; save and reload.
3. Edit global-object actions/properties and confirm persistence.
4. Validate ignore-rule behavior at Global root and Global Objects scope.
5. Run Import Globals from a second project and verify imported objects/actions/variables/ignored-rules are present and correctly scoped.
6. Export runtime and verify object/action/property parity with baseline sample.

## 9. Completion Criteria
1. `ProjectModel` solely owns global-object data.
2. No active references to `GlobalObjectsScope` as data owner.
3. All validation gates pass.
4. No regression in load/save/export/hierarchy/validation workflows.
5. No regression in Import Globals behavior (automated and manual coverage).
6. Migration notes updated with compatibility decisions and final cleanup status.

## 10. Immediate Next Slice Proposal
1. Status:
- Phase B complete.
- Phase C complete (scope adapter realigned to `ProjectModel`-owned storage; full-suite hard gate green).
2. Implement **Phase D** in small slices:
- migrate service/viewmodel call sites from `GlobalObjectsScope` data access to `ProjectModel` members
- keep compatibility aliases (`Global / Player`) active
- prioritize project explorer/hierarchy and validation lookup paths first
3. Phase D progress:
- Slice 1 complete: project/bootstrap/load wiring now initializes global-object state via `ProjectModel` members (`GameObjects`, `GlobalObjectAvailableActions`, `GlobalObjectVariables`, `GlobalObjectScopeName`, ignored-rule ids) instead of constructing a data-owning `GlobalObjectsScope` payload.
- `GlobalObjectsNodeViewModel` now supports a `ProjectModel`-first path for global object collection access while retaining compatibility constructor/scope property for existing tests and incremental migration safety.
- Slice 2 complete: metadata fallback load path in `JsonExportService` now initializes global-object state directly on `ProjectModel`; scope discovery service global-branch traversal now consumes `GlobalObjectsScope.ChildScopes` (scope-node contract) instead of direct `GlobalObjectsScope.GameObjects` data-member coupling.
4. Phase D hard gate complete:
- `dotnet test .\StoryboardDesigner.slnx` passed (1099/1099).
5. Next phase target:
- Begin Phase E by removing compatibility seam members where no active product dependency remains, then re-run focused gates per slice.
6. Phase E progress:
- Slice 1 complete: removed legacy `GlobalObjectsNodeViewModel(GlobalObjectsScope, ...)` constructor path so global-object hierarchy nodes are created via `ProjectModel`-backed constructor only in product code and updated affected tests accordingly.
- Focused gates green after the slice (build + import/hierarchy + playback + serialization/export + validation filters).
- Slice 2 complete: removed redundant `GlobalObjectsScope` field state from `GlobalObjectsNodeViewModel`; node now resolves both data (`GameObjects`) and scope identity (`ScopeNode`) directly from `ProjectModel`.
- Focused gates green after the slice (build + import/hierarchy + playback + serialization/export + validation filters).
- Slice 3 in progress: migrated additional test fixtures from `GlobalObjectsScope = new GlobalObjectsScope { ... }` setup to direct `ProjectModel` global-object members in the following files:
	- `QuantifiableRenamePropagationTests`
	- `MainWindowViewModelBaseObjectMovementRestrictionsTests`
	- `GameObjectSelectionOptionDiscoveryServiceTests`
	- `RoomTreeTraversalProjectionTests`
	- `MainWindowViewModelValidationSaveWorkflowTests`
	- `TraversalWizardApplyPipelineTests`
	- `ValidationEngineRegistrationTests`
- Focused gates green for migrated-slice coverage (`72/72`, `17/17`, playback `7/7`).
- Slice 3 complete: migrated remaining `GlobalObjectsScope = new GlobalObjectsScope { ... }` fixture initializers in test code, including JsonExport-focused suites (`JsonExportServiceHideEmptyConfigurationPersistenceTests`, `JsonExportServiceRuntimeExportTests`, `JsonExportServiceProjectStateTests`) and additional validation/workflow suites.
- Remaining explicit `GlobalObjectsScope = new GlobalObjectsScope` fixture initializers in `StoryboardDesigner.App.Tests` reduced to zero.
- Focused gates and full hard gate green after slice (`dotnet test .\StoryboardDesigner.slnx` passed 1099/1099).
- Slice 4 complete: removed `ProjectModel.GlobalObjectsScope` compatibility seam member and migrated remaining app/test call sites to semantic player-scope handling with local compatibility-node construction where needed.
- Product code no longer depends on `project.GlobalObjectsScope`/`_project.GlobalObjectsScope`; remaining `GlobalObjectsScope` references are constructor-based compatibility nodes and the adapter type itself.
- Focused gates green after slice (export/persistence `63/63`, validation `28/28`, import/hierarchy/discovery `22/22`, playback `7/7`).
- Full hard gate green after slice (`dotnet test .\StoryboardDesigner.slnx` passed 1099/1099).
- Slice 5 complete: replaced the remaining constructor-based compatibility-node uses with `ProjectPlayerScopeNode` and removed `GlobalObjectsScope` from product source.
- Active C# references to `GlobalObjectsScope` are now zero; global/player scope identity is represented by `ProjectPlayerScopeNode` while data ownership remains on `ProjectModel`.
- Focused gates green after slice (export/persistence `63/63`, validation `28/28`, import/hierarchy/discovery `22/22`, playback `7/7`).
- Full hard gate green after slice (`dotnet test .\StoryboardDesigner.slnx` passed 1099/1099).

## 11. Phase F Execution Checklist (Archive-Blocking)

Goal for this checklist:
1. Reach reasonable structural parity across three layers:
- Shared runtime contract DTOs (`Storyboard.Shared.Contracts/RuntimeContracts/Dtos/*`)
- Project JSON file DTOs (authoring persistence DTO layer)
- Designer models (`ProjectModel`-rooted model graph)
2. Make `ProjectModel` the effective root scope node (`ScopeNodeKind.Global`) and retire `ProjectGlobalScopeNode` from active product flows.

### F0. Lock Decisions Before Code Changes
1. Lock canonical root behavior contract (scope name/tokens, child ordering, add/remove semantics) to current `ProjectGlobalScopeNode` behavior.
2. Lock persistence contract rule: scope-graph mechanics introduced by `ScopeNodeBase` inheritance must not leak into authored project JSON shape.
3. Lock parity acceptance floor: no unexplained structural drift in mapper outputs across designer model -> project JSON DTO -> runtime contract DTO.

Exit criteria:
1. Locks recorded in this section before Phase F1 starts.

### F1. Baseline and Parity Harness
1. Inventory all active `new ProjectGlobalScopeNode(...)` call sites and type checks.
2. Add/confirm parity tests for:
- global child ordering and membership
- global add/remove child behavior
- validation path and hierarchy-target parity
- runtime export parity on representative fixtures
3. Capture baseline authored JSON and runtime export snapshots for comparison.

Exit criteria:
1. Baseline inventory complete and parity harness green.

### F2. Introduce Project JSON DTO Mapping Boundary
1. Ensure authored persistence flows through explicit project JSON DTO mapping (no implicit direct model-shape coupling).
2. Move legacy field-name compatibility handling into mapping layer only.
3. Keep persisted file shape stable unless an explicit migration note is approved.

Exit criteria:
1. Save/load and snapshot tests confirm stable authored JSON behavior.

### F3. Add Root-Scope Capability to ProjectModel (Dual Path)
1. Add `ScopeNodeBase` behavior to `ProjectModel` (or a minimal bridge) while keeping `ProjectGlobalScopeNode` temporarily available.
2. Implement global-node semantics on `ProjectModel` with parity to current adapter behavior:
- scope identity metadata
- ignored-rule ownership
- child scope projection order
- add/remove child behavior
3. Keep dual-path execution (adapter and model-root) until parity is proven.

Exit criteria:
1. Focused parity tests pass in dual-path mode.

### F4. Cut Over Call Sites to ProjectModel Root
1. Replace product-flow construction/consumption of `ProjectGlobalScopeNode` with `ProjectModel` root-scope usage.
2. Remove concrete-type assumptions in validation/hierarchy/lookup flows that require `ProjectGlobalScopeNode` identity.
3. Keep behavior lock checks green after each slice.

Exit criteria:
1. No active product-path dependency on `ProjectGlobalScopeNode`.

### F5. Remove Adapter and Finalize Parity
1. Remove `ProjectGlobalScopeNode` after zero active product references are confirmed.
2. Remove temporary dual-path/shim code introduced for migration.
3. Run full parity validation and update migration notes.

Exit criteria:
1. `ProjectModel` is the sole active global root scope node.
2. Parity matrix is green across all three layers.
3. All required gates in this plan are green.

### F-Validation Gates (Required Per Slice)
1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "JsonExportServiceProjectStateTests|JsonExportServiceRuntimeExportTests|JsonExportServiceRuntimeExportSnapshotTests|JsonExportServiceHideEmptyConfigurationPersistenceTests"`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "ValidationEngineRegistrationTests|ValidationIssueRunProcessorTests|SinglePlayerMarkerRuleTests|RuntimeExportIdUniquenessRuleTests|DuplicateNameInScopeRuleTests"`
4. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "MainWindowViewModelImportGlobalsCommandTests|RoomTreeTraversalProjectionTests|GameObjectSelectionOptionDiscoveryServiceTests"`
5. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`
6. Phase boundary hard gate: `dotnet test .\StoryboardDesigner.slnx`

### Archive Readiness Rule
This plan is not archive-ready until either:
1. Phase F checklist completes through F5 and parity gates are green, or
2. A documented explicit decision defers/cancels Phase F with rationale, risk acceptance, and a follow-on plan link.

## 12. Closeout Decision (2026-08-02)
1. Decision: Defer Phase F from this consolidation plan and continue it in a standalone follow-on plan: `plans/active/PROJECTMODEL_ROOT_SCOPE_PARITY_PLAN.md`.
2. Rationale: Phases A-E completed the original consolidation scope (single storage owner, compatibility seam retirement) and reached green focused/full validation gates; Phase F carries distinct architectural parity and root-scope inheritance risk that warrants independent tracking.
3. Risk acceptance: retaining `ProjectGlobalScopeNode` as the active global adapter is accepted temporarily while Phase F executes in the follow-on plan.
4. Validation evidence at closeout:
- `dotnet build .\StoryboardDesigner.slnx` passed.
- `dotnet test .\StoryboardDesigner.slnx` passed (1104/1104).
