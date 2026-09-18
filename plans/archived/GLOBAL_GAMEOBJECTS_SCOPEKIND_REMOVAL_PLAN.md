# GlobalGameObjects ScopeNodeKind Removal Plan

## 0. Purpose
This plan defines a dedicated, staged effort to remove `ScopeNodeKind.GlobalGameObjects` and replace special-case player/global-object handling with clearer ownership and identity rules.

Primary intent lock from product direction:
- A game object is the same contract/type regardless of location in the scope tree.
- Player identity is determined by game property semantics (for example `isPlayer`), not by a dedicated scope-node kind.
- `Global` is a location/scope context, not a separate game object type system.

## 1. Current Status (2026-08-02)
- Runtime export now externalizes game objects by id into `GameObject/*.runtime.json` and scope nodes reference ids.
- Runtime project node is now treated as logical global scope and remains root bootstrap file by explicit design exception.
- `ScopeNodeKind.GlobalGameObjects` is still actively used in runtime mapping/session/command alias handling and in designer authoring structures.
- Completed in this plan so far:
- player selection/routing invariants are test-locked around `isPlayer=true` semantics.
- `player.*` set-property mutation routing no longer depends on `GlobalGameObjects` semantics.
- player marker detection is centralized in shared runtime helper `Storyboard.Shared/GameStateData/PlayerMarker.cs` and consumed by runtime call sites.

## 2. Goal
Retire `ScopeNodeKind.GlobalGameObjects` safely without breaking runtime behavior, export contracts, or authoring workflows.

## 3. Non-Goals
- No broad redesign of command language or gameplay systems outside affected seams.
- No ad-hoc shortcut edits to generated contract DTOs.
- No immediate hard break for existing exports unless explicitly versioned and locked.

## 4. Why This Is Needed
`GlobalGameObjects` currently mixes two concerns:
- player alias/runtime behavior routing
- global-level object container location

This causes drift from intended model where:
- player should be property-identified (`isPlayer`)
- global objects should be handled as location-owned objects, not a special scope-kind category.

## 5. High-Risk Usage Inventory (Initial)
Runtime-critical references to resolve during migration:
1. `Storyboard.Shared/GameServices/Bootstrap/CleanRuntimeBootstrapSnapshotMapper.cs`
- maps player objects under `ScopeNodeKind.GlobalGameObjects`.
2. `Storyboard.Shared/GameStateData/GameStateSession.cs`
- builds dedicated `_playerNode` with `ScopeNodeKind.GlobalGameObjects` and routes scope-variable mutation through it.
3. `Storyboard.Shared/GameServices/References/ScopeChainVariableMutationResolver.cs`
- resolves `player.*` alias to `ScopeNodeKind.GlobalGameObjects`.
4. `Storyboard.Shared/GameServices/References/SelfAliasReferenceValueEnricher.cs`
- canonical prefix mapping includes `GlobalGameObjects => "player."`.
5. `Storyboard.Shared/GameServices/Commands/GameCommandPreprocessorService.cs`
- traversal path has explicit handling for `GlobalGameObjects` branch.
6. `Storyboard.Shared.Contracts/RuntimeContracts/Enums/ScopeNodeKind.cs`
- enum value is contract-visible and schema-backed.

Designer-side references exist extensively (`ProjectModel.GlobalGameObjects`, view models, validation, import/export), but those can be migrated incrementally behind stable runtime behavior checkpoints.

## 6. Migration Principles
1. Behavior-first and compatibility-first.
2. Remove semantic special-casing before removing enum token.
3. Keep migration additive and reversible until lock-off.
4. Validate at each seam with targeted regression tests.
5. Do not merge schema-contract removal in same slice as large runtime behavior rewrites.

## 7. Proposed Staged Execution

### Phase A: Semantics Clarification and Guardrail Tests
- Add/expand tests that codify intended semantics:
- global location object handling is uniform with other scope locations.
- player behavior selection is property-driven (`isPlayer`) rather than scope-kind driven.
- `player.*` alias behavior remains stable during transition.

Deliverables:
- new/updated tests in shared runtime and app test suites.
- no contract changes yet.

Exit criteria:
- focused runtime suites green.

### Phase B: Runtime Routing Refactor (No Contract Enum Removal Yet)
- Refactor runtime scope traversal, alias resolution, and mutation routing to stop depending on `GlobalGameObjects` as semantic switch.
- Introduce explicit runtime helper abstraction for "player context selection" based on object/property signals.
- Keep `ScopeNodeKind.GlobalGameObjects` temporarily mapped for compatibility.

Current next-item burn-down order (runtime-first):
1. `Storyboard.Shared/GameServices/Commands/GameCommandPreprocessorService.cs`
- remove explicit `GlobalGameObjects` traversal branch by treating global/player objects via object traversal and marker-based player resolution.
2. `Storyboard.Shared/GameStateData/GameStateScopeNode.cs`
- remove parent-kind special casing that maps `GlobalGameObjects` to runtime parent identity.
3. `Storyboard.Shared/GameStateData/GameStateSession.cs`
- remove remaining semantic checks where `GlobalGameObjects` affects object resolution/routing decisions.
4. `Storyboard.Shared/GameServices/Bootstrap/CleanRuntimeBootstrapSnapshotMapper.cs`
- align bootstrap mapping so global/player object ownership no longer requires dedicated `GlobalGameObjects` semantics.

Deliverables:
- runtime code paths no longer require `if kind == GlobalGameObjects` for core semantics.
- compatibility adapters preserved.

Exit criteria:
- parity tests pass for command processing and variable mutations.

### Phase C: Export/Bootstrap/Session Alignment
- Ensure bootstrap mapping and session tree construction model global object ownership without requiring a dedicated `GlobalGameObjects` semantic node.
- Keep root global bootstrap design exception intact unless explicitly re-scoped.

Deliverables:
- cleaner ownership wiring from runtime project/global scope to game objects.
- no gameplay regression.

Exit criteria:
- playback regression suite and runtime guardrails pass.

### Phase D: Contract and Enum Retirement Decision
- Evaluate whether contract enum token can be removed immediately or must remain as read-compat alias.
- If removal approved, update schema enum and regen/lock contracts.
- Add compatibility normalization for older payloads if required.

Deliverables:
- approved contract decision note.
- schema/codegen/lock updates if removal proceeds.

Exit criteria:
- lock-contract clean, build clean, focused suites clean.

### Phase E: Designer Model Cleanup
- Migrate designer naming/model plumbing from `GlobalGameObjects` toward clearer location-based naming.
- Keep UX labels stable where needed while internal naming transitions.

Deliverables:
- reduced drift between runtime semantics and authoring model names.

Exit criteria:
- designer validation and project load/save/export flows pass.

### Phase F: Final Hard Removal of `ScopeNodeKind.GlobalGameObjects`
Preconditions (must all be true before deletion):
1. Runtime (`Storyboard.Shared`) no longer uses `GlobalGameObjects` as behavioral switch.
2. Runtime compatibility paths (if any) are explicit and covered by tests.
3. Designer (`StoryboardDesigner.App`) and tests no longer require model/runtime enum token usage.
4. Contract decision from Phase D is approved and documented.

Execution steps:
1. Remove enum token from schema source and regenerate contracts.
2. Regenerate staged DTOs and run contract lock (`lock-contract`) acceptance workflow.
3. Remove remaining compile-time references across runtime, designer, and tests.
4. Rebuild and run full validation gates.

Exit criteria:
1. `grep` across `**/*.cs` returns no `ScopeNodeKind.GlobalGameObjects` references (except intentionally archived/generated historical artifacts if retained outside product build).
2. Solution build + runtime-focused suite + export tests are green.
3. Plan sign-off notes include migration behavior and compatibility window outcome.

## 8. Validation Gates (Per Phase)
1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "JsonExportServiceRuntimeExportTests|JsonExportServiceRuntimeExportValidationTests|GameSimulatorPlaybackRegressionTests"`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`

## 9. Open Decisions To Lock Before Enum Deletion
1. Should `player.*` remain first-class alias syntax after removal? If yes, what scope target contract does it bind to?
2. Is there always exactly one active player object, or can there be many objects with `isPlayer=true`?
3. If multiple `isPlayer=true` objects exist, what deterministic selection and failure policy applies?
4. Should legacy `GlobalGameObjects` enum token remain accepted for read compatibility for a deprecation window?
5. What contract versioning policy applies to enum token removal in runtime schema?

Decision default proposed for lock:
1. `player.*` remains supported and binds to the currently selected `isPlayer=true` game object context.
2. Selection policy is deterministic-first-match by existing traversal order, with diagnostics when no player exists.

## 10. Suggested First Slice (When Starting This Effort)
Start with a narrow seam: `player.*` alias + scope mutation routing.
- files to target first:
- `Storyboard.Shared/GameServices/References/ScopeChainVariableMutationResolver.cs`
- `Storyboard.Shared/GameStateData/GameStateSession.cs`
- `Storyboard.Shared/GameServices/References/SelfAliasReferenceValueEnricher.cs`

Reason:
- high semantic value
- lower blast radius than immediate enum/schema removal
- enables concrete behavior tests early

## 11. Completion Criteria
Plan is complete when all are true:
1. Runtime semantics no longer depend on `ScopeNodeKind.GlobalGameObjects` as behavioral switch.
2. Player identity/routing is property-driven per locked rules.
3. Contract decision for enum token is implemented and validated.
4. Regression gates pass and no architecture-boundary violations are introduced.
5. `ScopeNodeKind.GlobalGameObjects` enum member is removed from active product contracts and all active call sites.

## 11A. GlobalGameObjects_Model_Removal (Designer Model Alignment)

Purpose:
Align designer authored model shape with runtime contract direction by making `ProjectModel` the single owner of global scope game objects and removing the extra `GlobalGameObjects` container model.

Direction lock:
1. `ProjectModel` remains the single global-scope owner in designer model.
2. Top-level authored game objects live directly on `ProjectModel` as `GameObjects` (or approved equivalent).
3. `GlobalGameObjects` type is removed from active designer model usage.
4. Runtime/export compatibility is preserved during transition (legacy load aliases remain where needed).

Phase M1: Model Surface Introduction (Additive)
1. Add `ProjectModel.GameObjects` and move top-level object ownership semantics to that collection.
2. Move any required metadata currently on `GlobalGameObjects` to explicit `ProjectModel` properties (for example actions, ignored-rule ids) or approved replacement location.
3. Keep temporary compatibility shims only where needed to avoid broad breakage in one slice.

Exit criteria M1:
1. Build is green.
2. No behavioral changes in load/save/export flows.

Phase M2: Traversal/Scope Adapter Realignment
1. Update `ProjectGlobalScopeNode` to project `ProjectModel.GameObjects` directly in `ChildScopes`.
2. Remove assumptions that a child node of type `GlobalGameObjects` must exist.
3. Keep global scope adapter behavior intact for validation and hierarchy traversal.

Exit criteria M2:
1. Scope traversal tests and validation engine paths pass.
2. Hierarchy rendering still shows top-level game-object branch correctly.

Phase M3: Service/ViewModel/Validation Migration
1. Migrate `StoryboardDesigner.App` services, viewmodels, and validation rules from `project.GlobalGameObjects.*` to the new `ProjectModel` global-object surface.
2. Update `PropertyResolutionScope.GlobalGameObjects` naming to approved replacement while preserving editor behavior.
3. Remove remaining compile-time references to `GlobalGameObjects` in active designer code.

M3 risk classification:
1. High risk due to breadth of touch points across authoring workflows, validation traversal, and UI command surfaces.
2. M3 must be executed in small, reversible slices with test pass required per slice.

M3 required automated test depth:
1. Run targeted regression packs after each M3 slice:
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "JsonExportServiceProjectStateTests|JsonExportServiceRuntimeExportTests|JsonExportServiceRuntimeExportSnapshotTests|JsonExportServiceHideEmptyConfigurationPersistenceTests"`
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "ValidationEngineRegistrationTests|ValidationIssueRunProcessorTests|SinglePlayerMarkerRuleTests|RuntimeExportIdUniquenessRuleTests|DuplicateNameInScopeRuleTests"`
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "MainWindowViewModelImportGlobalsCommandTests|MainWindowViewModelBaseObjectMovementRestrictionsTests|QuantifiableRenamePropagationTests|RoomTreeTraversalProjectionTests|GameObjectSelectionOptionDiscoveryServiceTests"`
2. Run Section 8 validation gates at end of each M3 sub-slice bundle.
3. Keep red-green evidence in plan notes with date, slice id, and failing/passing test ids.

M3 required manual test script (per sub-slice):
1. Load an existing project that contains legacy `globalGameObjects` payload naming and confirm successful hydration.
2. In hierarchy/project explorer, verify top-level game objects are visible, selectable, and expandable.
3. Create, rename, and delete a top-level game object; save, reload, and confirm persistence.
4. Move at least one object between global scope and room scope (where supported) and verify parentage/render ordering remains correct.
5. Edit global-scope actions/properties affecting top-level game objects and verify validation issues update live.
6. Run runtime export and verify exported global object set/count parity against pre-change baseline.
7. Run simulator playback smoke path for one representative scenario that references player/global objects.

M3 manual sign-off policy:
1. No M3 slice merges without both automated and manual checklist completion.
2. If any manual step regresses, rollback the current slice and re-stage with narrower scope.
3. Record sign-off in plan notes with tester initials/date and baseline project used.

Exit criteria M3:
1. Designer app tests that cover load/save, hierarchy, validation, and object workflows pass.
2. Runtime export parity is preserved.
3. Manual test script passes for at least one legacy project and one current project.

Phase M4: Compatibility Tightening and Type Removal
1. Remove temporary compatibility members introduced in M1 once call sites are fully migrated.
2. Delete `GlobalGameObjects` model type and obsolete viewmodel wrappers tied only to that type.
3. Keep tolerant import/load mappings for legacy persisted payload names (`globalGameObjects`) until deprecation decision is approved.

Exit criteria M4:
1. No active `GlobalGameObjects` type references remain in `StoryboardDesigner.App` or `StoryboardDesigner.App.Tests` (excluding intentional legacy fixture payload strings).
2. Validation gates in Section 8 pass.
3. Plan status updated with compatibility window decision.

Risks and mitigations:
1. Risk: metadata loss when removing container fields.
- Mitigation: explicitly map each field (`Variables`, `AvailableActions`, ignored-rule ids, naming) to destination property before deletion.
2. Risk: hierarchy/validation assumptions on concrete container type.
- Mitigation: migrate via `ProjectGlobalScopeNode` adapter first, then remove concrete-type checks.
3. Risk: breaking existing saved project files.
- Mitigation: keep load aliases and sidecar fallback behavior until deprecation is locked.

## 12. Post-Completion Follow-Up Investigation (Parent/Child Linkage Semantics)
Run this only after Section 11 completion criteria are met.

Purpose:
Validate whether runtime parent linkage metadata (`RuntimeParentScopeNodeId`) correctly and consistently represents intended parent/child semantics across all scope kinds, including non-object hierarchy edges (for example Planet -> Country -> Area -> Room).

Why this is a follow-up:
Current plan scope is removal of `GlobalGameObjects` behavioral coupling. Broad parent-linkage semantics changes could introduce avoidable risk if mixed into enum-removal slices.

Investigation checklist:
1. Trace all runtime consumers of `RuntimeParentScopeNodeId` and classify whether each expects:
- container/object placement lineage only, or
- full hierarchy parent identity across all scope kinds.
2. Validate invariants for bootstrap descriptors vs runtime-mutated nodes:
- compare mapped `RuntimeScopeNodeDescriptor.RuntimeParentScopeNodeId` values against values after runtime reparent operations.
3. Create focused tests for representative edges:
- Global -> Planet
- Planet -> Country
- Country -> Area
- Area -> Room
- Room -> GameObject
- GameObject -> GameObject (contained object)
4. Decide and lock one canonical contract:
- `RuntimeParentScopeNodeId` remains object/container lineage only, or
- `RuntimeParentScopeNodeId` becomes universal parent id.
5. If canonical contract changes:
- stage as separate enhancement plan,
- update naming/comments for clarity,
- migrate consumers/tests in a dedicated slice,
- run full validation gates and playback regression.

Exit criteria for follow-up:
1. Decision note recorded with approved canonical semantics.
2. Tests cover the approved semantics and pass.
3. No regression in runtime command/action behavior or export contract guarantees.

## 13. Post-Completion Follow-Up Investigation (Runtime Snapshot Naming Clarity)
Run this only after Section 11 completion criteria are met.

Purpose:
Review `RuntimeGameWorldSnapshot.PlayerObjects` naming for clarity and accuracy after `GlobalGameObjects` retirement work settles.

Why this is a follow-up:
Renaming snapshot contract members has broad call-site impact across runtime, simulator, and tests. This should be isolated from enum-removal slices to reduce risk.

Investigation checklist:
1. Confirm semantic scope of `PlayerObjects` in runtime snapshot:
- verify it represents global-scope objects (not all game objects in the world).
2. Evaluate candidate names (for example `GlobalObjects`, `GlobalScopeObjects`) against existing runtime semantics and host readability.
3. If rename is approved, stage a dedicated rename slice:
- update `RuntimeGameWorldSnapshot` member name,
- update all constructor named arguments and access sites,
- preserve behavior and ordering contracts.
4. Add/update focused tests to protect snapshot mapping intent and regression behavior.

Exit criteria for follow-up:
1. Decision note records keep-vs-rename outcome and rationale.
2. If renamed, all affected call sites are updated and tests pass.
3. Runtime-focused validation gate remains green after the naming change.
