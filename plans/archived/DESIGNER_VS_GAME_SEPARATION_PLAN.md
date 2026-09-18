# Designer vs Game Separation Plan

Status: Completed (Phases A-F + final validation)
Owner: Pending
Last Updated: 2026-06-28

## 1) Objective
Establish a strict architectural boundary between Designer-authoring concerns and Game-runtime concerns so runtime services can operate on clean runtime contracts/DTOs without depending on designer model types.

## 2) Problem Statement
Current architecture still has coupling points where GameServices rely on designer-facing scope abstractions and extensions. This prevents full separation and makes runtime services harder to evolve independently.

Observed coupling indicators:
- `GameCommandPreprocessRequest` currently accepts designer scope abstraction types.
- `GameCommandPreprocessorService` relies on scope behavior that is still implemented through designer model pathways.
- Clean DTOs now share structure (`CleanScopeNodeBase`) but do not yet provide complete runtime scope-tree behavior contract.

## 3) Target Architecture (End State)
Dependency direction:
- Designer -> Runtime Contracts <- Game Runtime

Key outcomes:
- GameServices consume runtime contracts only.
- Clean DTO scope nodes provide explicit up/down navigation behavior.
- No GameServices dependency on designer model namespaces.
- Scope vocabulary/token behavior is contract-driven.

## 4) Scope Contract Strategy
Create a runtime-neutral scope contract named `IRuntimeScopeNode` with only runtime-needed members.

Proposed contract members:
- `ScopeKind`
- `ScopeName`
- `ScopeNameInGame`
- `ScopeTokens`
- `ParentScope`
- `ChildScopes`
- `AdditionalVerbs` (or equivalent vocabulary accessor)
- `AdditionalDirectionals` (or equivalent vocabulary accessor)

Transitional rule:
- Keep existing designer interface (`IScopedAwareNode`) temporarily for backward compatibility.
- New and migrated GameServices paths must use runtime contract only.

## 5) Implementation Phases

### Phase A: Contract Extraction
Deliverables:
- Introduce runtime scope contract in shared/contracts location.
- Move/introduce `ScopeKind` into contract boundary if needed.
- Add contract-focused extension utilities for ancestor traversal and vocabulary aggregation.
- Add CI architectural guardrails preventing GameServices -> designer namespace/assembly references.

Exit criteria:
- Contract exists and has tests for traversal and vocabulary behavior.
- No direct designer-model switch logic in new contract helpers.
- Dependency guardrails are active and failing on forbidden references.

### Phase B: Clean DTO Scope Behavior Completion
Deliverables:
- `CleanScopeNodeBase` implements runtime scope contract.
- Add non-serialized `ParentScope` linkage support.
- Implement DTO child-scope projections for:
  - Planet -> Countries
  - Country -> Areas
  - Room -> GameObjects
  - GameObject -> ContainedObjects
- Add scope-tree attachment utility after DTO load/deserialize.

Exit criteria:
- Clean DTO graph can be navigated up/down purely via runtime scope contract.

### Phase C: Preprocessor Path Migration
Deliverables:
- `GameCommandPreprocessRequest` migrated to runtime scope contract.
- `GameCommandPreprocessorService` updated to contract-only usage.
- Existing behavior preserved for object-token mapping and directional/verb vocabulary.

Exit criteria:
- Preprocessor compiles and runs without designer model interface dependencies.
- Existing preprocessor behavior parity validated via tests.

### Phase D: Runtime Path Consolidation
Deliverables:
- Ensure runtime loader produces attached scope graph suitable for preprocessor and processor.
- Remove transitional adapters from preprocessor path once complete.
- Remove direct `IScopedAwareNode` usage from GameServices runtime path.

Exit criteria:
- Runtime command flow (preprocess/process/session) runs against clean DTO/runtime graph.
- Transitional designer interface is no longer referenced by GameServices runtime command flow.

### Phase E: Validation Split and Cleanup
Deliverables:
- Explicit split of authoring-time validation vs runtime invariants.
- Remove dead/legacy coupling code paths.
- Add architectural guardrails (tests/rules) preventing GameServices -> designer references.

Exit criteria:
- Separation is enforceable and regression-protected.

### Phase E.5: Shared Contracts DLL Extraction
Deliverables:
- Create a new shared DLL project for finalized contracts/helpers used by both Designer and Runtime/Simulator.
- Move only stabilized shared types/helpers (no mixed behavior changes in same phase).
- Update project references so Designer and Runtime/Simulator consume shared DLL contracts.
- Add build validation that Runtime/Simulator assemblies have no compile-time references to designer model assemblies/namespaces.

Exit criteria:
- Shared DLL project builds and is referenced by both sides.
- Runtime/Simulator compile without direct designer model references.
- No behavior change regressions introduced by extraction move.

### Phase F: Independent Simulator Executable Proof
Deliverables:
- Create a separate executable project for game simulation/runtime validation.
- Wire it to runtime contracts/services only (no designer model/project references).
- Load clean export payloads and run command simulation flows end-to-end.
- Add CI/build step that compiles this executable independently.

Exit criteria:
- Separate simulator executable builds and runs using clean DTO/runtime paths.
- Project has zero compile-time references to designer model namespaces/assemblies.
- A passing smoke/integration test demonstrates command simulation in this executable.

## 6) Testing Plan

### Contract tests
- Ancestor traversal
- Self-and-ancestor traversal
- Child traversal integrity
- Vocabulary aggregation behavior

### Preprocessor parity tests
- Object token resolution
- NameInGame alias handling
- Directional parsing behavior
- Frame-based primary/secondary object resolution

### Integration tests
- Clean export DTO load -> attach scope tree -> preprocess/process flow
- Negative-path graph integrity tests (duplicate parent, cycle attempt, disconnected node)
- Backward-compat load tests for existing clean export samples

## 7) Risks and Mitigations

Risk: Hidden coupling in utility methods
- Mitigation: dependency scan + explicit forbidden namespace references in runtime services.

Risk: Behavior drift during migration
- Mitigation: parity test cases frozen before each migration step.

Risk: DTO navigation complexity for room/area linkage
- Mitigation: define one canonical attachment utility and document assumptions.

Risk: Contract churn
- Mitigation: additive-first contract evolution policy and version notes.

## 8) Top 10 Design Decisions (Locked for v1)
1. Runtime scope contract is read-only navigation.
2. `ScopeKind` is moved and owned at the runtime contract boundary.
3. Parent linkage is explicit after load/deserialize via an attachment pass and remains non-serialized.
4. One canonical scope attachment utility is the single source of truth for parent/child wiring.
5. Attachment utility enforces invariants: single parent, acyclic graph, and deterministic child order.
6. Vocabulary aggregation is contract-driven and deterministic: self first, then ancestors nearest-to-farthest, with case-insensitive de-duplication.
7. Transitional designer interface (`IScopedAwareNode`) is adapter-only and must not be used directly by migrated GameServices paths.
8. Architectural dependency guardrails (forbidden GameServices -> designer references) are enabled in CI starting in Phase A.
9. Contract versioning policy is additive-first for v1; any breaking change requires explicit version bump and migration notes.
10. Sequencing remains phased: namespace/contracts migration first, shared contracts DLL extraction in Phase E.5, and independent simulator proof in Phase F.

## 8.1) Phase A/B Decisions Required To Proceed (Locked)
1. Runtime contract name is `IRuntimeScopeNode`.
2. Runtime enum name is `RuntimeScopeKind` and contains: `Global`, `Templates`, `Player`, `Planet`, `Country`, `Area`, `Room`, `GameObject`.
3. Runtime contract namespace/folder for now is `StoryboardDesigner.App.RuntimeContracts` under a dedicated `RuntimeContracts/` folder (same assembly until Phase E.5 extraction).
4. `IRuntimeScopeNode` is read-only and contains only: `RuntimeScopeKind ScopeKind`, `string ScopeName`, `string ScopeNameInGame`, `IEnumerable<string> ScopeTokens`, `IRuntimeScopeNode? ParentScope`, `IEnumerable<IRuntimeScopeNode> ChildScopes`, `IEnumerable<string> AdditionalVerbs`, and `IEnumerable<string> AdditionalDirectionals`.
5. Runtime helpers (`EnumerateAncestors`, `EnumerateSelfAndAncestors`, `GetValidVerbs`, `GetValidDirectionalQualifiers`) move to contract-oriented extensions and must not use designer-type switch statements.
6. Vocabulary normalization for helpers is fixed as: trim, remove blank, case-insensitive distinct, stable deterministic output (alphabetical order).
7. `IScopedAwareNode` remains mutable for designer workflows, but any Phase A/B runtime-facing API additions must depend only on `IRuntimeScopeNode`.
8. `CleanScopeNodeBase` will implement `IRuntimeScopeNode` with a non-serialized `ParentScope` property and derived `ChildScopes` projection.
9. `CleanScopeNodeBase` will expose `ScopeName` as `Name`, `ScopeNameInGame` as `Name` by default, `ScopeTokens` defaulting to `Name` plus `ScopeNameInGame` when present.
10. Child scope projections for DTOs are fixed as: `CleanPlanetDto -> Countries`, `CleanCountryDto -> Areas`, `CleanRoomExportV1Dto -> GameObjects`, and `CleanGameObjectDto -> ContainedObjects`.
11. `CleanAreaDto` room children are resolved by attachment utility through `RoomIds` using loaded `CleanRoomExportV1Dto` sidecars (not inline serialization).
12. Attachment utility behavior is strict on topology (duplicate parent/cycle/disconnected graph) and fail-fast via explicit diagnostics + failure result; missing optional room sidecar references are treated as non-fatal diagnostics in v1.
13. Attachment pass runs exactly once after DTO load/deserialize and before preprocess/processor usage.
14. Phase B includes tests for parent linkage, child projection integrity, deterministic traversal, and negative graph integrity cases.

## 9) Milestone Checklist
- [ ] Approve this plan document
- [x] Lock top 10 design decisions (v1 baseline)
- [x] Finalize runtime scope contract shape
- [x] Lock Phase A/B decisions required to proceed
- [x] Implement Phase A
- [x] Implement Phase B
- [x] Implement Phase C
- [x] Implement Phase D
- [x] Implement Phase E
- [x] Implement Phase E.5
- [x] Implement Phase F
- [x] Final architecture/dependency validation

## 10) Review Gate
No implementation work begins until this plan is explicitly approved.

## 11) Suggested First Vertical Slice (Post-Approval)
- Implement runtime scope contract + DTO parent/child attachment utility.
- Migrate only `GameCommandPreprocessRequest` and `GameCommandPreprocessorService`.
- Validate with parity-focused tests before broader runtime migration.

## 12) Decision Log
- 2026-06-28: Top 10 design decisions above were locked for v1 planning baseline.
- 2026-06-28: Phase A/B execution decisions were locked (contract shape, enum ownership, DTO attachment semantics, and validation behavior).
- 2026-06-28: Phase A implemented (runtime contract types, runtime traversal/vocabulary helpers, adapter bridge, and contract tests).
- 2026-06-28: Phase B implemented (DTO runtime scope contract behavior, parent/child attachment utility with integrity validation, loader integration, and Phase B tests).
- 2026-06-28: Phase C implemented (`GameCommandPreprocessRequest` and `GameCommandPreprocessorService` migrated to `IRuntimeScopeNode` with adapter-based call-site updates and parity tests passing).
- 2026-06-28: Phase D implemented (runtime session scope nodes now implement `IRuntimeScopeNode`, global root vocabulary propagated, and preprocess call sites use runtime nodes directly without adapter usage in command flow).
- 2026-06-28: Phase E implemented (added architecture guardrail tests enforcing no designer-scope dependency in preprocessor path and no adapter usage at processor preprocess call sites).
- 2026-06-29: Final architecture/dependency validation completed. Evidence: simulator project guardrail tests (`ArchitectureSeparationGuardrailsTests`) now enforce no `StoryboardDesigner.App` dependency in `Storyboard.Simulator` project/source and verify shared-manager startup composition; standalone simulator project build (`dotnet build .\\Storyboard.Simulator\\Storyboard.Simulator.csproj`) passes; shared manager host fixture test (`SharedManagerHostFixtureTests`) passes.
