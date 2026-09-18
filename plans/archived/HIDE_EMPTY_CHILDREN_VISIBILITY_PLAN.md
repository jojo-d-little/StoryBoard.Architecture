# Hide Empty Children Visibility Plan

Status: Implemented (Archive Ready)
Owner: StoryboardDesigner.App tree authoring UX
Last updated: 2026-07-19

## Plan Maintenance (2026-07-19)

1. Implementation status verified against current code and regression coverage.
2. Remaining open decision D6 is now locked with current implemented behavior.
3. Plan is closed and ready for archival.

## Objective

Reduce hierarchy visual noise by hiding empty child groupings per node while keeping empty options easy to restore on demand.

## Initial Direction (From Session)

1. Persist a per-node authored flag: hideEmptyConfiguration.
2. Add project-level convenience actions: Hide Empty Children and Show Empty Children.
3. Add per-node context action: Hide Empty Options / Show Empty Options.
4. Global convenience action is not persisted as a project-wide mode; it only writes per-node values in one pass.

## Proposed Iteration Shape

1. Finalize semantics and persistence model.
2. Implement filtered-children projection logic in hierarchy building/refresh.
3. Add global and per-node tree context actions.
4. Add save/load + migration-safe defaults.
5. Add regression tests for visibility behavior and persistence.

## Question List (To Resolve One by One)

### A. Scope and Data Model

1. Which node types support hideEmptyChildren?
2. Do we store hideEmptyChildren on every scope-bearing node (Global, Planet, Country, Area, Room, Template Room, object nodes), or only selected node types?
3. Should hideEmptyChildren apply only to "configuration-style" child groups (Properties/Actions/Verbs/Directionals/etc.), or to every empty child node regardless of type?
4. Should leaf entries ever be suppressed by this feature, or only grouping/folder nodes?
5. For project root/global node, is default hideEmptyChildren false on new projects?
6. For non-root nodes, what default should be used on newly created nodes?

### B. What Counts as "Empty"

1. Is a group empty strictly when source collection count is 0?
2. For mixed groups (for example, children that can contain nested children), is "empty" based on immediate items only or recursive descendant emptiness?
3. If a group has hidden descendants but itself has no direct items, should it still be shown?
4. How should "empty" be evaluated for special groups like traversal or templates where availability may be action-driven?
5. Should validation badges/warnings force a node to stay visible even if empty?

### C. Interaction and UX

1. Exact per-node context menu wording: Show Empty Options vs Hide Empty Options?
2. Should the per-node action appear only on nodes that can own child option groups, or on all nodes?
3. Should toggling per-node hideEmptyChildren immediately refresh just that branch or the whole tree?
4. Should global actions be placed on Global root context menu only, or also in a top-level command/menu?
5. Should global action skip nodes where hiding is not meaningful?
6. After global hide/show action, should current expand/collapse states be preserved as much as possible?

### D. Persistence and Compatibility

1. Store hideEmptyChildren in native project file or sidecar state file?
2. If in project file, which authored model types gain the property?
3. Backward compatibility default for legacy files missing the field?
4. Do we need clean export changes? (expected: no, unless explicitly desired)
5. Are there any snapshot baselines that must be updated for project JSON shape changes?

### E. Global Convenience Actions

1. Should global hide/show recurse across all authored nodes including nested objects/templates?
2. Should global operation include template/base catalogs?
3. Should global operation be undoable in one step (if undo stack exists for tree operations)?
4. Should global operation prompt confirmation for very large projects?
5. Should global operation set root node value too, or only descendants?

### F. Guardrails and Testing

1. Which tests should be added first: model persistence, tree projection, context actions, or end-to-end selection restore?
2. Need regression coverage for "hidden by default deeper nodes, manually shown local branch" workflow?
3. Need coverage for node creation after hide is enabled (new child defaults and visibility)?
4. Need coverage for save/load roundtrip preserving per-node choices?
5. Need coverage that hidden empty nodes remain discoverable via per-node/global show actions?

## Candidate Decisions Log

- D1: Persisted location for hideEmptyConfiguration. Status: Agreed (2026-07-17).
	- Persist in native project file (not sidecar UI state).
- D2: Supported node type set. Status: Agreed (2026-07-17).
	- Enabled on primary scope nodes: Global, Planet, Country, Area, Room, and Template Room.
	- Enabled on Game Object nodes (including nested/global/template object variants).
	- Not enabled on leaf/utility nodes.
	- Special invariant: each node's Settings child is always visible (never hidden), because it is a non-expandable single editor entry.
- D3: Empty evaluation semantics and visibility exceptions. Status: Agreed (2026-07-17).
	- hideEmptyChildren applies broadly to empty grouping children under a primary scope node.
	- Only grouping/folder nodes are suppressible; leaf authored entries are never suppressed.
	- Empty is evaluated by direct source emptiness (source count == 0 for that grouping).
	- Emptiness evaluation is immediate-only (non-recursive).
	- Groupings with zero direct items are treated as empty and hidden when hideEmptyChildren is enabled (except explicit always-visible exceptions).
	- Special/action-driven groups follow the same direct-count emptiness rule (no additional special-case emptiness logic at this time).
	- Runtime authoring updates (for example, traversal wizard creating legs/mappings) must raise MVVM change notifications so a previously hidden-empty grouping automatically becomes visible once it gains children.
	- Empty grouping visibility is not overridden by validation state by default; add only targeted exceptions later if proven necessary.
	- Always-visible exceptions:
		- Settings child node.
		- Geography progression folders that host the next scope level:
			- Planet -> Countries folder.
			- Country -> Areas folder.
			- Area -> Rooms folder.
- D4: Clean export impact. Status: Agreed (2026-07-17).
	- No clean export/schema changes.
	- hideEmptyConfiguration remains designer-side authored project data only.
- D5: Default values for new and legacy data. Status: Agreed (2026-07-17).
	- hideEmptyConfiguration is explicit opt-in only.
	- Default is false (not hidden) for new projects/nodes.
	- Default is false for legacy files where the field is missing.
	- New nodes do not inherit hideEmptyConfiguration from parents; they always start unhidden.
- D6: Validation visibility override behavior. Status: Agreed (2026-07-19).
	- Validation indicators do not force-show hidden empty configuration groupings by default.
	- Visibility follows hide-empty rules unless a future targeted exception is explicitly introduced.

- D7: Per-node context action wording. Status: Agreed (2026-07-17).
	- Use one dynamic per-node action label (single action slot):
		- Hide Empty Configuration when current node is showing empty configuration groups.
		- Show Empty Configuration when current node is hiding empty configuration groups.
	- Do not use a generic Toggle Empty Configuration label.

- D8: Per-node action visibility scope. Status: Agreed (2026-07-17).
	- Show per-node hide/show empty configuration action only on primary scope nodes:
		- Global, Planet, Country, Area, Room, Template Room.

- D9: Per-node refresh strategy. Status: Agreed (2026-07-17).
	- Per-node hide/show operations refresh only the affected node branch by default.
	- Preserve selection and broader tree expansion state.
	- Permit targeted fallback to full-tree rebuild only if a specific workflow cannot be handled by branch refresh.

- D10: Global action placement. Status: Agreed (2026-07-17).
	- Expose global convenience actions in both locations:
		- Global root context menu.
		- Top-level command/menu entry.

- D11: Global action applicability filter. Status: Agreed (2026-07-17).
	- Global hide/show operations skip non-applicable nodes.
	- Apply only to primary scope nodes.
	- Do not apply to leaves, utility nodes, or always-visible exception folders.

- D12: Global action expansion-state behavior. Status: Agreed (2026-07-17).
	- Preserve existing expand/collapse state as much as possible after global hide/show operations.
	- Apply only minimal expansion-state changes required by visibility updates.

- D13: Persisted flag naming. Status: Agreed (2026-07-17).
	- Use hideEmptyConfiguration (not hideEmptyChildren).

- D14: Legacy/additive compatibility framing. Status: Agreed (2026-07-17).
	- No migration step is required.
	- Behavior is additive and opt-in:
		- If hideEmptyConfiguration is absent, tree shows empty configuration groups.
		- Empty configuration is hidden only when the setting is explicitly present/enabled.

- D15: Project-boundary scope. Status: Agreed (2026-07-17).
	- This feature is designer-only (StoryboardDesigner.App behavior/persistence).
	- No Storyboard.Shared contract/runtime changes.
	- No Storyboard.Simulator runtime behavior changes.

- D16: Snapshot/baseline update scope. Status: Agreed (2026-07-17).
	- Update only designer native project persistence/load-save baselines that assert exact project JSON shape.
	- No clean export snapshot updates are expected.

- D17: Global recursion scope. Status: Agreed (2026-07-17).
	- Global Hide/Show Empty Configuration recurses across all authored nodes that support hideEmptyConfiguration.
	- Includes primary scope nodes, game object nodes (including nested objects), template authored trees, and base catalogs.

- D18: Global operation undo behavior. Status: Agreed (2026-07-17).
	- Global Hide/Show Empty Configuration should be undoable in one step (single logical transaction) when undo infrastructure is available.

- D19: Global operation confirmation UX. Status: Agreed (2026-07-17).
	- No confirmation prompt by default, including large projects.
	- Execute immediately and rely on single-step undo for recovery.

- D20: Global root inclusion. Status: Agreed (2026-07-17).
	- Global Hide/Show Empty Configuration applies to root node and all applicable descendants.

- D21: Test implementation order. Status: Agreed (2026-07-17).
	- Test sequence:
		1. Model persistence tests.
		2. Tree projection/visibility filtering tests.
		3. Context-action tests (per-node and global actions).
		4. End-to-end selection/restore tests.

- D22: Core workflow regression coverage. Status: Agreed (2026-07-17).
	- Add regression coverage for the primary workflow:
		- Global hide operation.
		- Manual local branch reshow behavior.
	- Add regression coverage for top-to-bottom tree navigation after:
		- Global hide operation.
		- Global reshow operation.

- D23: Node-creation default behavior coverage. Status: Agreed (2026-07-17).
	- Add regression coverage for node creation after hide operations have been used.
	- Assert new nodes default hideEmptyConfiguration = false (never hidden by default).
	- Assert new nodes initially show empty configuration groups until explicitly hidden.

- D24: Persistence roundtrip coverage. Status: Agreed (2026-07-17).
	- Add explicit save/load roundtrip regression coverage for per-node hideEmptyConfiguration values.
	- Include representative primary scope, game object, nested object, and template nodes.

- D25: Discoverability recovery coverage. Status: Agreed (2026-07-17).
	- Add regression coverage that hidden-empty groups remain recoverable through:
		- Per-node Show Empty Configuration.
		- Global Show Empty Configuration.

## Implementation-Ready Summary

### Finalized Feature Contract

1. Persist per-node hideEmptyConfiguration in native project data.
2. Supported nodes:
	- Primary scope nodes (Global, Planet, Country, Area, Room, Template Room).
	- Game object nodes (including nested/global/template object variants).
3. Default behavior:
	- Explicit opt-in only.
	- Missing/unspecified value means show empty configuration.
	- New nodes always start with hideEmptyConfiguration = false.
4. Visibility semantics:
	- Hide only grouping/folder configuration nodes when direct item count is 0.
	- Never hide leaf authored entries.
	- Always-visible exceptions:
		- Settings child.
		- Geography progression folders (Planet->Countries, Country->Areas, Area->Rooms).
	- Same direct-count rule for special/action-driven groups.
	- If workflow actions add children (for example traversal wizard), MVVM updates must auto-reveal newly non-empty groups.
5. Context actions:
	- Per-node dynamic action label:
		- Hide Empty Configuration / Show Empty Configuration.
	- Per-node action appears only on nodes that support hideEmptyConfiguration.
	- Global actions appear both on Global root context menu and top-level command/menu.
6. Global action behavior:
	- Applies to root plus all applicable descendants.
	- Recurses through primary scopes, objects, templates, and base catalogs.
	- Skips non-applicable nodes.
	- Preserves expansion state as much as possible.
	- No confirmation prompt.
	- One-step undo transaction when undo infrastructure is available.
7. Boundaries:
	- Designer-only feature.
	- No Shared/simulator runtime/export contract changes.

### Implementation Slices

1. Model + persistence slice:
	- Add hideEmptyConfiguration to supported authored model types.
	- Wire JSON serialization/deserialization defaults (false when absent).
2. Tree projection slice:
	- Apply visibility filtering with agreed emptiness and exception rules.
	- Ensure branch refresh preserves selection/expansion context.
	- Ensure add/update workflows can auto-reveal when groups become non-empty.
3. Command/action slice:
	- Add per-node dynamic hide/show action.
	- Add global hide/show commands in root context and top-level menu.
	- Implement recursive apply with applicability filtering.
4. Test slice (ordered):
	- Persistence tests.
	- Projection/filter tests.
	- Context/global action tests.
	- End-to-end navigation/restore tests, including hide then reshow and discoverability recovery.

## Phased Execution Plan

### Phase 1 - Model and Persistence Foundations

Goal:
1. Introduce hideEmptyConfiguration to all agreed authored node models and persist/load it safely.

Scope:
1. Add property to supported models (primary scopes + object variants).
2. Ensure defaults are false when absent.
3. Keep serialization additive and migration-free.

Exit criteria:
1. Save/load roundtrip preserves per-node values.
2. Legacy files without field load with show-by-default behavior.
3. No Shared/simulator/export contract changes.

### Phase 2 - Tree Filtering and MVVM Refresh Behavior

Goal:
1. Apply hide/show logic in hierarchy projection while preserving usability guarantees.

Scope:
1. Hide only empty grouping nodes by direct count.
2. Respect always-visible exceptions (Settings and geography progression folders).
3. Preserve branch selection/expansion as much as possible during refresh.
4. Ensure workflow-driven additions (for example traversal wizard) auto-reveal groups that become non-empty.

Exit criteria:
1. Branch-level filtering works for all supported node families.
2. New content appearing in previously hidden groups reappears automatically.
3. No hidden leaf-content regressions.

### Phase 3 - Per-Node Actions

Goal:
1. Provide local hide/show controls at supported nodes with clear wording and deterministic outcomes.

Scope:
1. Add single dynamic context action label:
	- Hide Empty Configuration when currently showing.
	- Show Empty Configuration when currently hiding.
2. Expose only on nodes supporting hideEmptyConfiguration.
3. Execute as branch refresh operations.

Exit criteria:
1. Action appears only where valid.
2. Toggle updates model state and tree projection immediately.
3. Neighbor branches remain visually stable.

### Phase 4 - Global Convenience Actions

Goal:
1. Enable one-step broad hide/show operations for the whole project tree.

Scope:
1. Add global commands in both global-root context menu and top-level command/menu.
2. Apply to root + all applicable descendants, including templates/base catalogs.
3. Skip non-applicable nodes.
4. Preserve expansion state as much as possible.
5. No confirmation prompt.
6. Use one-step undo transaction when available.

Exit criteria:
1. Global hide/show behaves predictably across large trees.
2. Local per-node overrides still work after global runs.
3. Undo reverts global operation in one step (where infrastructure supports it).

### Phase 5 - Regression Test Suite and Hardening

Goal:
1. Lock behavior with deterministic tests focused on the approved workflow.

Scope:
1. Persistence tests.
2. Projection/filter tests.
3. Per-node/global action tests.
4. Top-to-bottom navigation tests after hide and after reshow.
5. Node-creation default tests (new nodes always unhidden).
6. Discoverability recovery tests via per-node/global show actions.

Exit criteria:
1. All new targeted tests pass.
2. Existing relevant suites pass without regression.
3. Feature behavior matches decisions D1-D25.

## Baseline Effort Estimates

Estimation basis:
1. Single developer, focused execution.
2. Includes coding, tests, and local validation.
3. Excludes unrelated interruptions.

Per-phase baseline ranges:
1. Phase 1 - Model and Persistence Foundations: 6-10 hours.
2. Phase 2 - Tree Filtering and MVVM Refresh Behavior: 10-16 hours.
3. Phase 3 - Per-Node Actions: 6-10 hours.
4. Phase 4 - Global Convenience Actions: 8-14 hours.
5. Phase 5 - Regression Test Suite and Hardening: 10-18 hours.

Total baseline range:
1. 40-68 hours.

Primary uncertainty drivers:
1. Tree refresh edge cases under branch-only updates.
2. Regression stabilization across traversal/template/base and nested object paths.

## Plan Completion Checkpoint (Estimate vs Actual)

Use this section at feature closure to compare baseline estimates against actual effort and outcomes.

Comparison method:
1. For each phase, record Actual (hrs).
2. Convert each baseline range to a midpoint target for normalized comparison:
	- midpoint = (min + max) / 2
3. Compute phase delta values:
	- Delta vs midpoint = Actual - midpoint
	- Delta vs min = Actual - min
	- Delta vs max = Actual - max
4. Compute phase variance percentage against midpoint:
	- Variance % = ((Actual - midpoint) / midpoint) * 100
5. Compute total rollup using summed mins, maxes, and midpoint:
	- Total midpoint = (Total min + Total max) / 2
	- Total delta vs midpoint = Total actual - Total midpoint
	- Total variance % = ((Total actual - Total midpoint) / Total midpoint) * 100
6. Interpret total variance bands:
	- On target: within +/-10%
	- Moderate variance: +/-10% to +/-25%
	- High variance: beyond +/-25%
7. In Notes, record the primary reason for each phase variance (scope growth, regression hardening, discovery, rework, etc.).

Completion metadata:
1. Completion date: ____________________
2. Owner: ____________________
3. Branch/commit: ____________________

Phase-by-phase comparison:

| Phase | Baseline (hrs) | Actual (hrs) | Delta (hrs) | Notes |
|---|---:|---:|---:|---|
| Phase 1 - Model and Persistence Foundations | 6-10 |  |  |  |
| Phase 2 - Tree Filtering and MVVM Refresh Behavior | 10-16 |  |  |  |
| Phase 3 - Per-Node Actions | 6-10 |  |  |  |
| Phase 4 - Global Convenience Actions | 8-14 |  |  |  |
| Phase 5 - Regression Test Suite and Hardening | 10-18 |  |  |  |

Rollup:
1. Total baseline range: 40-68 hours.
2. Total actual hours: ____________________
3. Net delta vs baseline range: ____________________

Outcome checks:
1. All phase exit criteria met: Yes / No.
2. Decision-log conformance (D1-D25): Yes / No.
3. Post-release regressions linked to this feature: Yes / No.
4. Follow-up actions required: ____________________

## Next Step

Walk through questions in order, starting with A1-A3, and lock decisions in this document before implementation begins.
