# ProjectModel Root Scope Parity Plan

## 0. Purpose
Create structural parity across three layers while making ProjectModel the active global root scope node.

Parity target layers:
1. Shared runtime contract DTOs in Storyboard.Shared.Contracts/RuntimeContracts/Dtos
2. Project JSON file DTOs for authored persistence
3. Designer models rooted in ProjectModel

Primary objective:
1. Retire ProjectGlobalScopeNode from active product flows and make ProjectModel the only active ScopeNodeKind.Global root.

## 1. Why This Plan Exists
Current architecture still uses ProjectGlobalScopeNode as an adapter surface for scope graph behavior while ProjectModel remains data owner. This creates avoidable projection duplication and slows parity work between model, authored DTO shape, and runtime DTO shape.

This plan completes the remaining consolidation step in a controlled way with explicit parity gates.

## 2. Scope
In scope:
1. ProjectModel as root scope node behavior
2. Retirement of ProjectGlobalScopeNode from active product flows
3. Authoring JSON DTO boundary hardening and mapping parity
4. Cross-layer parity checks among model, project DTO, and runtime DTO

Out of scope:
1. Runtime command grammar changes
2. New gameplay features
3. Broad UI redesign
4. Runtime contract-breaking schema changes unless separately approved

## 3. Design Locks
1. Root identity lock: one active ScopeNodeKind.Global node in product flows, implemented by ProjectModel.
2. Behavior lock: preserve existing global child ordering and add/remove semantics.
3. Persistence lock: scope graph mechanics from ScopeNodeBase inheritance must not leak into authored JSON shape.
4. Mapping lock: legacy naming compatibility belongs in mapper boundaries, not domain model core.
5. Validation lock: validation path and hierarchy target behavior must remain parity-equivalent.

## 4. Success Criteria
1. ProjectModel is the only active global root scope node used by product code.
2. ProjectGlobalScopeNode has zero active product references and is removed.
3. Project JSON persistence flows through explicit DTO mapping boundary.
4. Structural parity matrix is green across model, project DTO, and runtime DTO.
5. Focused and full regression gates are green.

## 5. Phase Plan

### Phase F0: Baseline and Inventory
1. Inventory all construction and type-check call sites for ProjectGlobalScopeNode.
2. Inventory authored JSON load/save entry points and current DTO/model assumptions.
3. Capture baseline snapshots for authored JSON and runtime export on representative fixtures.

Exit criteria:
1. Inventory completed and documented.
2. Baseline tests and snapshots are green.

### Phase F1: Parity Harness
1. Add or confirm tests that lock current behavior:
2. Global child ordering and membership
3. Global add and remove child behavior
4. Validation path and hierarchy target parity
5. Import Globals parity
6. Runtime export parity for representative fixtures

Exit criteria:
1. Parity harness passes without behavioral drift.

### Phase F2: Project JSON DTO Boundary
1. Ensure authored persistence uses explicit project JSON DTOs and mappers.
2. Move compatibility and alias handling into mapper layer only.
3. Keep authored JSON shape stable unless a migration note is explicitly approved.

Exit criteria:
1. Save and load pass through DTO mappers.
2. Authored JSON snapshots remain stable or intentional migrations are documented.

### Phase F3: Dual-Path Root Scope Enablement
1. Add ScopeNodeBase behavior to ProjectModel in parity mode.
2. Port global root responsibilities from adapter behavior:
3. Scope identity metadata
4. Ignored rule ownership
5. Child scope projection and deterministic ordering
6. Add and remove child behavior
7. Keep ProjectGlobalScopeNode temporarily as delegating shim for low-risk transition.

Exit criteria:
1. Dual-path behavior (adapter vs ProjectModel-root) is parity-equivalent under focused tests.

### Phase F4: Product Call Site Cutover
1. Switch product call sites from new ProjectGlobalScopeNode construction to ProjectModel root use.
2. Remove concrete-type assumptions tied to ProjectGlobalScopeNode.
3. Keep path, hierarchy, and validation parity checks green after each slice.

Exit criteria:
1. Product flow no longer depends on ProjectGlobalScopeNode.

### Phase F5: Adapter Retirement and Final Parity
1. Remove ProjectGlobalScopeNode after confirming zero active product references.
2. Remove temporary dual-path shim logic.
3. Finalize parity matrix and migration notes.

Exit criteria:
1. ProjectModel is sole global root in active product code.
2. ProjectGlobalScopeNode removed.
3. Parity matrix and all gates are green.

## 6. Validation Gates
Per meaningful slice:
1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "JsonExportServiceProjectStateTests|JsonExportServiceRuntimeExportTests|JsonExportServiceRuntimeExportSnapshotTests|JsonExportServiceHideEmptyConfigurationPersistenceTests"
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "ValidationEngineRegistrationTests|ValidationIssueRunProcessorTests|SinglePlayerMarkerRuleTests|RuntimeExportIdUniquenessRuleTests|DuplicateNameInScopeRuleTests"
4. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "MainWindowViewModelImportGlobalsCommandTests|RoomTreeTraversalProjectionTests|GameObjectSelectionOptionDiscoveryServiceTests"
5. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"

Phase boundary hard gate:
1. dotnet test .\StoryboardDesigner.slnx

## 7. Risk Register and Mitigations
1. Risk: serialization leakage from ScopeNodeBase members into project JSON.
Mitigation: explicit DTO boundary and snapshot checks before and after inheritance shift.
2. Risk: scope traversal ordering regressions.
Mitigation: deterministic ordering assertions in parity harness.
3. Risk: hidden type-coupling to adapter concrete type.
Mitigation: F0 inventory plus F4 concrete-type removal sweep.
4. Risk: behavior drift in validation and hierarchy selection.
Mitigation: focused parity tests and run-level issue-path diff checks.
5. Risk: migration churn in alias compatibility behavior.
Mitigation: keep compatibility handling isolated in mappers; avoid new in-model alias accumulation.

## 8. Decisions and Notes Log
Use this section to record lock confirmations, intentional migrations, and any approved deviation from parity rules.

2026-08-02
1. Root enablement approach approved: ProjectModel will inherit ScopeNodeBase in F3.
2. Transition strategy approved: keep ProjectGlobalScopeNode only as a temporary delegating shim until F5 retirement.
3. Global child ordering parity lock approved: preserve current effective order exactly as observed today (strict no-drift).
4. Cutover mode approved: hard switch to ProjectModel root in product flows.
5. Migration rule approved: if this effort changes the project JSON format, perform an immediate one-time migration of all sample projects in the same effort.
6. Migration evidence rule approved: each sample migration change includes focused parity test execution in the same change.
7. Signaling decision approved: no additional schema/version signaling change is required at this time; sample migration is the compatibility action.
8. F5 retirement hard gate approved: before removing ProjectGlobalScopeNode, pass all focused gates in Section 6 and one full solution test run on the same branch tip.
9. PR packaging rule approved: any JSON format change and the resulting one-time migration of all sample projects ship together in one PR.
10. Merge gate approved for that PR: every sample project must open and round-trip save cleanly under automated tests before merge.
11. Round-trip assertion standard approved: use semantic JSON equality (not byte-for-byte equality) for sample migration gate tests.

## 9. Archive Readiness Rule
This plan is archive-ready only when one of these is true:
1. F0 through F5 complete with all gates green, parity evidence recorded, and ProjectGlobalScopeNode retired.
2. A documented defer or cancel decision is approved with rationale, accepted risks, and a linked follow-on plan.

## 10. Immediate Next Slice
1. Execute F0 inventory of ProjectGlobalScopeNode construction and type checks.
2. Land F1 parity harness additions for global ordering and path parity.
3. Run focused gates and full suite hard gate before starting F2.

## 11. Closeout Evidence (2026-08-03)
Phase completion summary:
1. F0 complete: ProjectGlobalScopeNode construction and type-check inventory completed.
2. F1 complete: parity harness coverage includes import globals parity and sample-project round-trip save stability.
3. F2 complete: authored JSON mapping boundary unchanged; no JSON format change introduced by this effort.
4. F3 complete: ProjectModel now implements global ScopeNodeBase root behavior.
5. F4 complete: product call sites cut over to ProjectModel root.
6. F5 complete: ProjectGlobalScopeNode removed with zero remaining C# references.

Validation evidence:
1. `dotnet build .\\StoryboardDesigner.slnx` passed.
2. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "JsonExportServiceProjectStateTests|JsonExportServiceRuntimeExportTests|JsonExportServiceRuntimeExportSnapshotTests|JsonExportServiceHideEmptyConfigurationPersistenceTests"` passed (`63 passed, 0 failed`).
3. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "ValidationEngineRegistrationTests|ValidationIssueRunProcessorTests|SinglePlayerMarkerRuleTests|RuntimeExportIdUniquenessRuleTests|DuplicateNameInScopeRuleTests"` passed.
4. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "MainWindowViewModelImportGlobalsCommandTests|RoomTreeTraversalProjectionTests|GameObjectSelectionOptionDiscoveryServiceTests"` passed.
5. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"` passed (`7 passed, 0 failed`).
6. `dotnet test .\\StoryboardDesigner.slnx` passed (`1115 passed, 0 failed`).

Migration note:
1. The project JSON format did not change in this effort, so no one-time sample migration was required.
