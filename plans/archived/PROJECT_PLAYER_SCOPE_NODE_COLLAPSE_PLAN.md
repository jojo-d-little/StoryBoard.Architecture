# Project Player Scope Node Collapse Plan

## 0. Purpose
Collapse use of `ProjectPlayerScopeNode` into `ProjectGlobalScopeNode`-owned behavior so the project has one effective global scope root model while preserving existing alias/path behavior (`Global`, `Global / Player`, `Player`) and validation semantics.

## 0A. Current Status (2026-08-02)
1. Completed: `ProjectPlayerScopeNode` removed from product source; single global adapter flow is active through `ProjectGlobalScopeNode`.
2. Completed: transitional player-path alias emissions were removed from canonical/emitted scope-path builders.
3. Completed: remaining global-objects alias-root compatibility branches were removed or replaced with canonical `ProjectGlobalScopeNode` handling.
4. Completed: validation scope typing no longer maps any global alias root to `ScopeType.Player`; `ScopeType.Player` removed.
5. Validation evidence:
- `dotnet build .\StoryboardDesigner.slnx` passed.
- Focused regression gate passed (`50/50`) for validation/path/discovery/import-global suites.
- Playback smoke gate passed (`7/7`).
- Full hard gate passed: `dotnet test .\StoryboardDesigner.slnx` => `1104/1104` green.

Primary intent:
1. Remove duplicate `ScopeNodeKind.Global` adapter roles for the same project graph.
2. Keep `ProjectModel` as data owner during this effort.
3. Preserve behavior for traversal, validation path lookup, suppression, hierarchy mapping, and runtime identity outputs.

## 1. Current Problem
Current design uses two separate global-kind adapters:
1. `ProjectGlobalScopeNode` for top-level global grammar/catalog traversal.
2. `ProjectPlayerScopeNode` for player/global-objects branch identity.

This creates risks:
1. Ambiguity around what is "the" global root.
2. Extra branch logic for identifying player scope via type/name heuristics.
3. Higher chance of drift in alias/path behavior.

## 2. Target End State
1. `ProjectGlobalScopeNode` is the only active global-kind adapter used across product code.
2. Player/global-objects behavior is represented as a branch/alias concept under the global root, not as a second global-kind root adapter class.
3. Canonical scope paths no longer use player terminology.
4. Temporary compatibility aliases may exist during transition but are removed by plan completion.
5. `ProjectModel` remains the sole storage owner for global object state.
6. Ignore-rule ownership at root/global scope is unified: one root/global ignore list applies to root-level game objects and root-level scope issues.

## 3. Decision Locks
1. No runtime export contract/schema changes in this plan.
2. No command grammar behavior changes.
3. No broad UI redesign.
4. Keep `ProjectModel` inheritance unchanged in this plan (do not make it `ScopeNodeBase` yet).
5. Preserve deterministic behavior for validation path resolution and runtime identity generation.
6. Minimize project JSON shape churn; avoid file-format field renames/removals unless explicitly approved.
7. Preserve Import Globals behavior throughout all slices.

## 3A. Design Lock Questions
1. Single-root representation: Should player/global-objects semantics be modeled as a dedicated child branch node under `ProjectGlobalScopeNode`, or as alias projection behavior without a concrete branch node type?
2. Scope-kind uniqueness: Must there be exactly one active `ScopeNodeKind.Global` node instance per project traversal graph at runtime, or are temporary internal duplicates acceptable if not observable externally?
3. Canonical identity: Which path is canonical for player-object issues after collapse (`Global / Player / ...` vs `Global / ...`), and which paths are compatibility aliases only?
4. Alias lifetime: Are `Player` and `Global / Player` aliases permanent compatibility requirements, or can they be deprecated in a later phase once telemetry/tests show no dependency?
5. Ignore-rule ownership: Should player-scope ignore rules remain distinct (`GlobalObjectIgnoredValidationRuleIds`) from global-root ignores (`GlobalIgnoredValidationRuleIds`), or be merged under a single global ignore list?
6. Child-scope ordering contract: Is the current ordering in global traversal (game objects, templates, room templates, base objects, planets) a locked contract for deterministic behavior and snapshots?
7. Validation resolution precedence: When both canonical and alias paths match, what precedence should `ResolveScopeNodeForIssuePath` use, and should this precedence be codified by tests?
8. Hierarchy mapping contract: Should validation navigation always resolve player-object issues to the same hierarchy nodes as before, even if internal scope-node representation changes?
9. Runtime ID stability: Which runtime identities are considered immutable across this refactor (especially player-root related IDs), and where is that asserted by tests/snapshots?
10. Parent-scope semantics: After collapse, should top-level global game objects report parent scope as `ProjectGlobalScopeNode` directly, or a logical player branch adapter?
11. Adapter surface location: Should player-branch helper logic live inside `ProjectGlobalScopeNode`, or in a separate internal helper class to keep responsibilities isolated?
12. API exposure policy: Do we keep any public constructors/APIs that imply a standalone player-scope node, or make all player-branch semantics internal to global-scope traversal only?
13. Migration fallback policy: If a slice causes alias/path regressions, do we allow a temporary compatibility wrapper in product code, or must regressions be fixed without reintroducing a second global-kind adapter?
14. Done criteria precision: Is "no active `ProjectPlayerScopeNode` product references" sufficient, or do we also require zero tests asserting old concrete-type identity semantics?

## 3B. Locked Design Decisions
1. Locked from Question 1: Use a single global/project root model with a standard game-object collection at root scope; do not model a separate player scope concept as a first-class scope adapter.
2. Terminology lock: Shift language away from player objects and global objects in designer architecture; prefer global-level game objects (or project-level game objects).
3. Player marker caveat lock: A player game object still exists as a normal game object identified by `isPlayer=true`. Runtime owns primary lookup/resolution behavior. If designer workflows need that lookup, add a reusable designer-side helper method rather than ad hoc query logic.
4. Locked from Question 2: Exactly one active `ScopeNodeKind.Global` node instance is allowed per project traversal graph in product code.
5. Locked from Question 3: During transition, `Global / Player / ...` and `Player / ...` may be accepted as compatibility aliases. End-state canonical and emitted scope paths must not include player terminology.
6. Locked from Question 4: Player-based aliases are transitional compatibility inputs only. They are not canonical outputs, are not persisted as canonical paths, and must be removed or unreachable in normal flows by plan completion.
7. Locked from Question 4 (removal gate): Alias removal is complete when focused/full gates are green, canonical/emitted paths contain no player terminology, and any residual parser fallback is isolated and explicitly deprecated.
8. Locked from Question 5: Converge root/global ignore ownership to a single list at global/project scope. Root-level game objects inherit/apply this same root/global ignore set; separate root-level game-object ignore list storage is retired.
9. Locked from Question 6: Keep current global child traversal order stable through this plan unless an explicit, separately tested ordering change is approved.
10. Locked from Question 6 caveat: Child lookup logic should trend toward explicit typed requests (ask for the specific child type needed) rather than broad traversal assumptions.
11. Locked from Question 7: Scope-path resolution precedence is canonical-first. Legacy alias paths are fallback-only during transition; when multiple fallback matches exist, select the most specific (longest) deterministic match.
12. Locked from Question 7 caveat: Alias-fallback resolution paths should be instrumented/logged as deprecation usage during transition.
13. Locked from Question 8: Validation-to-hierarchy navigation must preserve node-target parity with baseline behavior for equivalent issues throughout this plan.
14. Locked from Question 9: Keep JSON churn minimal. If unavoidable authored-project JSON shape changes are discovered, stop and discuss before proceeding, and implement a one-time sample project migration/update pass.
15. Locked from Question 9 caveat: `Import Globals` must remain behaviorally stable throughout migration; add/retain focused coverage and treat regressions as stop-the-line.
16. Locked from Question 10: Root-level game objects report parent scope directly as `ProjectGlobalScopeNode` (no intermediate player/global-objects pseudo-parent in end-state flows).
17. Locked from Question 11: Keep `ProjectGlobalScopeNode` focused on scope-graph responsibilities; place root-scope path/alias/lookup compatibility behavior in a separate internal helper/service with focused unit coverage.
18. Locked from Question 12: Alias-fallback removal requires a hard cutover gate with zero fallback usage plus green targeted regression suites and sample-corpus parity checks, including `Import Globals` and ignore-rule editing behaviors.
19. Locked from Question 13: For global-scope authored content, standardize on a dedicated scoped container with neutral member names (for example: `GameObjects`, `GameProperties`, `AvailableActions`, `IgnoredValidationRuleIds`, `Name`, `ProducerNotes`) instead of embedding scope prefixes in member names. Keep transitional aliases/mapping from legacy names until cutover gates are satisfied.
20. Locked from Question 14 clarification: Default naming policy is to drop `Global*` prefixes for authored-content members and use scope-neutral names. Any exception that remains specially prefixed must be explicitly discussed and approved as a deliberate special case.
21. Locked from Question 15: If any authored-project JSON format/shape changes are introduced, the change must include a clear JSON-churn note in the PR and a one-time in-place migration of sample and test projects, with migration validation captured in the same slice.

## 4. Non-Goals
1. Do not migrate `ProjectModel` to inherit from `ScopeNodeBase` in this plan.
2. Do not remove legacy alias strings (`Global / Player`, `Player`) yet.
3. Do not redesign catalog/template/base object structure.

## 5. Phased Execution

### Phase A: Baseline + Alias Lock
1. Inventory all `ProjectPlayerScopeNode` references and all alias checks that map player scope semantics.
2. Add/confirm tests that lock alias parity for:
- path lookup
- validation issue node resolution
- hierarchy selection mapping
3. Capture baseline behavior for scope-path generation and suppression behavior.

Exit criteria:
1. Reference inventory complete.
2. Alias/path behavior covered in focused tests.

### Phase B: Introduce Single-Root Player Semantics in Global Scope
1. Add explicit player-branch semantics to `ProjectGlobalScopeNode` (or helper owned by it) without removing old node yet.
2. Ensure player branch name, ignored-rule mapping, and game-object child behavior are available via global-root semantics.
3. Keep compatibility shim for `ProjectPlayerScopeNode` temporarily while call sites migrate.

Exit criteria:
1. `ProjectGlobalScopeNode` can express player branch semantics without requiring `ProjectPlayerScopeNode` as the primary dependency.
2. No behavior drift in focused gates.

### Phase C: Migrate Call Sites
1. Replace direct construction/consumption of `ProjectPlayerScopeNode` with `ProjectGlobalScopeNode`-owned player branch semantics.
2. Update validation/lookup/hierarchy services to avoid second-root assumptions.
3. Maintain alias outputs unchanged.

Priority targets:
1. validation issue run processing and scope path aliases
2. hierarchy scope-node mapping
3. dialog/file command scope resolution
4. runtime adapter parent/scope identity handling

Exit criteria:
1. Product code no longer requires `ProjectPlayerScopeNode` for normal flow.
2. Focused and manual checks pass.

### Phase D: Remove Compatibility Node
1. Remove `ProjectPlayerScopeNode` type if no active references remain.
2. Remove temporary compatibility helpers introduced during migration.
3. Keep alias compatibility behavior.

Exit criteria:
1. No references to `ProjectPlayerScopeNode` in product code.
2. One effective global scope adapter model (`ProjectGlobalScopeNode`) in active use.

## 6. Slice Strategy
For each phase, execute in small slices:
1. One edit cluster.
2. Run focused tests.
3. Compare alias/path behavior.
4. Continue only if green.

Phase boundary hard gate:
1. Run full test suite at each phase transition.
2. Do not advance phases on red suite.

Rollback policy:
1. Revert current slice only if regressions appear.
2. Avoid stacking unrelated edits.

## 7. Validation Gates
Run after each meaningful slice:
1. `dotnet build .\\StoryboardDesigner.slnx`
2. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "JsonExportServiceProjectStateTests|JsonExportServiceRuntimeExportTests|JsonExportServiceRuntimeExportSnapshotTests|JsonExportServiceHideEmptyConfigurationPersistenceTests"`
3. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "ValidationEngineRegistrationTests|ValidationIssueRunProcessorTests|SinglePlayerMarkerRuleTests|RuntimeExportIdUniquenessRuleTests|DuplicateNameInScopeRuleTests"`
4. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "MainWindowViewModelImportGlobalsCommandTests|RoomTreeTraversalProjectionTests|GameObjectSelectionOptionDiscoveryServiceTests"`
5. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`

Mandatory phase-end hard gate:
1. `dotnet test .\\StoryboardDesigner.slnx`

## 8. Manual Regression Checklist
1. Validate global/player path resolution from representative validation issues.
2. Validate hierarchy selection from validation report entries targeting player/global-object paths.
3. Create/rename/remove top-level global object and verify persistence.
4. Confirm ignore-rule suppression still works for global and player aliases.
5. Export runtime and verify deterministic scope identity behavior remains unchanged.

## 9. Completion Criteria
1. `ProjectPlayerScopeNode` removed or fully inert.
2. `ProjectGlobalScopeNode` is the single effective global adapter in use.
3. No active player terminology remains in canonical scope paths.
4. Compatibility aliases using player terminology are removed or fully confined to transitional fallback paths not used in emitted/canonical outputs.
5. Focused and full hard gates pass.
6. Follow-up decision recorded for any future `ProjectModel : ScopeNodeBase` migration.
7. Root/global ignore rules are unified and applied consistently for root-level game-object validation behavior.

## 10. Immediate Next Slice Proposal
1. Phase A Slice 1: create inventory and add a focused alias parity test set for validation path lookup and hierarchy resolution.
2. Run focused gates.
3. If green, begin Phase B with minimal helper introduction in `ProjectGlobalScopeNode`.

## 11. Phased Gate Matrix (Automation vs Manual Stop)

Use this matrix as the operational rule for all remaining work in this plan.

### Gate Levels
1. `Level F` (Focused automation): run `dotnet build` plus the focused test commands in Section 7.
2. `Level A` (All automated): run full suite `dotnet test .\\StoryboardDesigner.slnx`.
3. `Manual Stop`: pause coding and execute the manual checklist in Section 8 before phase advancement.

### Phase-by-Phase Flow
1. **Phase A (Baseline + Alias Lock)**
	- Per slice: run `Level F`.
	- Phase-end: run `Level A`.
	- Stop for manual review: **No** (unless `Level F` or `Level A` reveals path/hierarchy ambiguity requiring visual verification).

2. **Phase B (Single-Root Semantics Introduction)**
	- Per slice: run `Level F`.
	- Phase-end: run `Level A`.
	- Stop for manual review: **Yes, mandatory** after `Level A` passes.
	- Manual focus: global/player alias resolution, hierarchy navigation parity, top-level object CRUD persistence.

3. **Phase C (Call-Site Migration)**
	- Per slice: run `Level F`.
	- Mid-phase checkpoint (after first migration cluster lands): run `Level A`.
	- Phase-end: run `Level A` again.
	- Stop for manual review: **Yes, mandatory** at phase-end `Level A` pass.
	- Manual focus: validation-to-hierarchy mapping, ignore-rule editing behavior, Import Globals parity.

4. **Phase D (Compatibility Node Removal)**
	- Pre-removal slice: run `Level F`.
	- Removal slice: run `Level F` immediately after deletion/refactor.
	- Phase-end: run `Level A`.
	- Stop for manual review: **Yes, mandatory** before merge/finalization.
	- Manual focus: exported/runtime identity parity and alias fallback behavior boundaries.

### JSON-Shape Change Override Gate
If any slice changes authored-project JSON shape/format:
1. Upgrade that slice to immediate `Level A` (do not wait for phase-end).
2. Stop for manual review in the same slice.
3. Execute one-time in-place sample/test project migration in the same slice.
4. Record PR-visible JSON churn note and migration validation evidence before continuing.

### Stop/Continue Rule
1. Continue within a phase only when the required automation level for that step is green.
2. Advance to the next phase only when phase-end `Level A` is green and any required `Manual Stop` is completed.
3. Any red automation or manual regression is stop-the-line; fix before new scope is added.

## 12. Closeout Status (2026-08-02)
1. Plan scope complete.
2. `ProjectPlayerScopeNode` removed from product source; canonical/emitted paths no longer use player terminology.
3. Focused and full hard gates passed at closeout (`dotnet test .\StoryboardDesigner.slnx`: 1104/1104).
4. Follow-on architectural work for making `ProjectModel` the root scope node is tracked separately in `plans/active/PROJECTMODEL_ROOT_SCOPE_PARITY_PLAN.md`.
