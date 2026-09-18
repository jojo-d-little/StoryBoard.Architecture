# Quantifiable Item Handling Plan

Status: Complete
Owner: Pending
Last updated: 2026-06-30

## 1) Objective
Establish a deterministic, future-ready runtime and authoring model for quantifiable items that supports:
1. Grouped stacks and true individual instances.
2. Composite consume/break restore behavior without item loss.
3. Predictable command routing when multiple similar instances exist.
4. In-room linked room-instance placement from an existing base item.

## 2) Problem Statement
1. Composite consume for quantifiable items decrements quantity but may not preserve consumed-part lineage for break restore.
2. Individual instance distribution mode is not consistently materialized as true runtime instances in scope trees.
3. Multiple instances with same display name create ambiguity for command forwarding and player-facing output.
4. Break restoration needs explicit rules when original source stack/instance is no longer in scope.
5. Authoring needs a clear "same as existing item" placement model that shares actions but allows local room-instance state.

## 3) Scope
In scope (v1):
1. Runtime materialization rules for GroupedStack vs IndividualInstances.
2. Composite consume + break restore contract for quantifiable parts.
3. Deterministic stack-anchor merge/create rules on break restore.
4. Child command forwarding mode for similar-instance dispatch (single vs all).
5. Regression coverage for stack/instance/composite interactions.
6. Linked room-instance authoring model for quantifiable items (applies to both `GroupedStack` and `IndividualInstances`).

Out of scope (v1):
1. Full parser disambiguation syntax for selecting a specific same-name instance (example: "marble 2").
2. In-room placement UI/coordinates.
3. Advanced stack metadata strategies beyond strict compatibility merge rules.
4. Full unlink/fork workflow for room-instance actions (advanced authoring, not required for v1).

## 4) Core Decisions
### 4.1 Runtime Representation (Option A)
1. `GroupedStack` uses one runtime anchor node with quantity.
2. `IndividualInstances` uses one runtime node per unit.
3. Runtime must preserve stable internal identity for each instance node.

### 4.2 Composite Consume/Break Restore
1. For `IndividualInstances`:
- Build consumes by reparenting consumed instance nodes under composite target.
- Break restores by reparenting those exact nodes back to restore scope.

2. For `GroupedStack`:
- Build consumes quantity from source stack and records consumed quantity provenance under composite target.
- Break restores quantity from provenance using deterministic restore target rules.

3. Consumed provenance is required so break can restore even when source anchor changed or is absent.

### 4.3 Break Restore Target Rules (Quantifiable)
When breaking a composite that consumed quantifiable parts:
1. If original source anchor is still in-scope and compatible, restore there.
2. Else restore to break execution scope (inventory if broken from inventory, room if broken from room).
3. If no compatible anchor exists in restore scope, create a new anchor instance there.
4. Multiple anchors of the same object type are allowed across different scopes.
5. Within one scope, compatible anchors auto-merge.

Compatibility for merge:
1. Same technical scope name (case-insensitive).
2. Both anchors are active quantifiable runtime anchors with positive quantity.

### 4.5 Sibling Proximity Auto-Merge (Stackable Quantifiable)
When two stackable quantifiable anchors of the same item type become direct siblings, they must auto-merge into a single anchor.

Definition of direct proximity:
1. Two candidate object nodes share the same immediate parent scope node.
2. Both are active runtime anchors.

Merge eligibility:
1. Same technical scope name (canonical runtime identity for this phase).
2. Quantifiable is true.
3. Active is true and quantity is positive.
4. Candidate anchors are direct siblings in the scope tree.

Merge behavior:
1. Keep one deterministic winner anchor and remove the loser anchor.
2. Winner quantity becomes the sum of both quantities.
3. Apply this whenever sibling duplicates can be introduced by runtime mutations.

Deterministic winner selection:
1. Prefer source/provenance anchor when present and eligible.
2. Otherwise choose by deterministic stable id ordering.

### 4.4 Child Command Forwarding for Similar Instances
Add an explicit dispatch mode for child forwarding:
1. `SingleMatchingChild` (default): forward to first eligible similar child only.
2. `AllMatchingChildren`: forward to all eligible similar children.

Deterministic selection order for `SingleMatchingChild`:
1. Scope tree traversal order.
2. Stable runtime id as tie-breaker.

### 4.6 Linked Room-Instance Model (Quantifiable)
Terminology (producer-facing):
1. `Base Item`: canonical item definition.
2. `Room Instance`: room-placed occurrence created from an existing base item.
3. `Linked Actions`: actions are inherited from the base item and are not edited on the room instance.

Rules:
1. Applies to quantifiable items in both distribution modes: `GroupedStack` and `IndividualInstances`.
2. Shared from `Base Item` (non-editable on room instance): actions/command handling.
3. Local on `Room Instance` (editable per placement): room/scope placement, active state, quantity, game-property values.
4. Base-item action edits propagate to linked room instances.

## 5) Data/Contract Changes
1. Add runtime provenance contract for quantifiable composite consumption:
- consumed object definition id
- consumed quantity (for grouped)
- consumed instance ids (for individual)
- source scope/anchor identifiers (best effort)

2. Add forwarding dispatch enum to action contract:
- `SingleMatchingChild`
- `AllMatchingChildren`

3. Keep producer-facing checkbox in dialog mapped to enum:
- unchecked -> `SingleMatchingChild`
- checked -> `AllMatchingChildren`

4. Add room-instance linkage metadata for placed quantifiable objects:
- base object identifier reference
- link behavior mode for actions (v1: linked only)

## 6) Runtime Behavior Contract
### 6.1 Materialization
1. `GroupedStack`: one runtime node with quantity >= 1.
2. `IndividualInstances`: X runtime nodes each representing one unit.
3. Simulator scope tree must display individual nodes for `IndividualInstances`.

### 6.2 Composite Build
1. Validate availability according to distribution mode.
2. Record consumed provenance before state mutation completion.
3. On failure after partial mutation attempt, rollback to preserve consistency.

### 6.3 Composite Break
1. Restore from recorded provenance.
2. Apply restore target chain from section 4.3.
3. Ensure no net quantity loss/duplication after full build+break cycle under deterministic rules.

### 6.4 Forwarding Behavior
1. For broad verbs like `look`, default single-dispatch avoids duplicated repeated output.
2. Producers may opt into all-dispatch when desired behavior is explicit.

### 6.5 Stackable Sibling Merge Normalization
1. After runtime mutations that can create sibling duplicates, normalize sibling anchors under the affected parent.
2. If two eligible stackable quantifiable siblings match, merge them immediately.
3. Ensure no duplicate stack anchors of the same item type remain under one parent after normalization.

## 7) Authoring UX
1. Keep existing quantifiable distribution mode controls.
2. Add forwarding checkbox in command forwarding UI:
- Label: "Forward to all matching similar children"
3. Tooltip explains default single dispatch and when to enable all.
4. Validation: setting applies only when child forwarding mode is enabled.
5. Keep add flow entry at: "Add Existing Quantifiable Object".
6. Add linked-placement guidance in dialog copy:
- "Creates a Room Instance linked to the Base Item's actions."
7. Use distinct iconography for linked room instances:
- Base item icon remains unchanged.
- Room instance icon uses link badge to indicate linked actions.
8. In room-instance context, action editing affordances are disabled/hidden with explanatory message.

## 8) Test Plan
### Unit tests
1. GroupedStack materializes one runtime anchor with quantity.
2. IndividualInstances materializes one runtime node per unit.
3. Composite build (grouped) decrements quantity and records provenance.
4. Composite break (grouped) restores quantity from provenance.
5. Break restore uses source anchor when available.
6. Break restore creates new anchor in restore scope when source anchor unavailable.
7. Same-scope compatible stack anchors merge deterministically.
8. Composite build+break roundtrip preserves total quantity invariants.
9. Child forwarding single mode dispatches one similar child.
10. Child forwarding all mode dispatches all eligible similar children.
11. Room drop merge: dropping stack A beside stack B of same type merges into one stack with summed quantity.
12. Inventory merge: moving similar stackable items into player scope as siblings merges to one anchor.
13. Break-return merge: break restoration into a scope with a compatible sibling stack merges deterministically.
14. Same-scope inactive sibling is not eligible for merge.
15. Linked room-instance action edits are blocked on room instances and allowed on base items.

### Integration tests
1. End-to-end grouped quantifiable composite build/break from inventory scope.
2. End-to-end grouped quantifiable composite break when original source anchor out-of-scope.
3. End-to-end individual-instance composite build/break preserving concrete instance identity.
4. Simulator scope tree shows expanded individual instances for `IndividualInstances` mode.
5. Sibling stack auto-merge is reflected in runtime scope tree after drop/pickup/break flows.
6. Add Existing Quantifiable Object creates a linked room instance from an existing base quantifiable item.
7. Linked room-instance behavior works for both `GroupedStack` and `IndividualInstances` placement modes.
8. Base-item action changes are reflected for linked room instances across different rooms.

### Regression tests
1. Non-quantifiable composite behavior unchanged.
2. Existing command forwarding behavior unchanged when new dispatch setting is left at default.
3. Existing base-item editing workflows remain unchanged.

## 9) Risks and Mitigations
Risk:
1. Quantity drift or duplication under partial failures.
Mitigation:
1. Provenance-first mutation and rollback-safe state transitions.

Risk:
1. Output spam when multiple similar children exist.
Mitigation:
1. Default to single dispatch; explicit all-dispatch opt-in.

Risk:
1. Runtime complexity increase from dual grouped/instance paths.
Mitigation:
1. Keep shared invariants and centralized consume/restore helpers.

Risk:
1. Producer confusion between base items and room instances.
Mitigation:
1. Use explicit terminology, link-badge iconography, and disabled action-edit affordances with clear guidance text.

## 10) Phased Execution
### Phase Q1: Contracts + Materialization
1. Add provenance + forwarding dispatch contracts.
2. Ensure runtime `IndividualInstances` materialization is true per-instance.
3. Add simulator scope tree visibility assertions.

Exit criteria:
1. Runtime/simulator clearly distinguishes grouped vs individual modes.

Status:
1. Complete.

### Phase Q2: Composite Quantifiable Provenance
1. Implement grouped consume provenance recording.
2. Implement grouped break restore target chain + anchor create/merge rules.
3. Verify roundtrip invariants.
4. Implement sibling-proximity auto-merge normalization for stackable quantifiable anchors across runtime mutation paths.

Exit criteria:
1. Grouped quantifiable composite break reliably restores consumed quantities.

Status:
1. In progress.
2. Done: grouped consume provenance recording and persistence lifecycle (set on build, clear on successful break).
3. Done: restore target chain supports source-anchor preference, compatible-anchor fallback, and create-new-anchor fallback.
4. Done: sibling-proximity auto-merge normalization implemented in runtime mutation paths (reparent/container transfers/restore-anchor creation).
5. Done: merge compatibility aligned to technical scope name identity only (player-facing nameInGame excluded).
6. Done: focused regressions added for reparent/restore-anchor/container put/container remove sibling-merge paths.
7. Done: broadened roundtrip invariant coverage for composite quantifiable edge scenarios (including sibling-normalization interactions).

### Phase Q3: Forwarding Dispatch Mode
1. Add UI checkbox + action contract mapping.
2. Implement single/all similar-child dispatch behavior.
3. Add focused routing tests.

Exit criteria:
1. Producers can choose single vs all behavior deterministically.

Status:
1. In progress.
2. Done: runtime contract and execution path now support explicit `SingleMatchingChild` (default) vs `AllMatchingChildren` dispatch for forwarded similar children.
3. Done: focused routing regressions added for default single dispatch and explicit all-dispatch behavior.
4. Done: producer-facing action editor now exposes dispatch selection and persists the option through authoring/export mapping.

### Phase Q4: Linked Room Instances For Quantifiable Items
1. Implement "Add Existing Quantifiable Object" as linked room-instance placement from a base item.
2. Enforce shared-vs-local edit boundaries:
- shared (linked): actions
- local (room-instance): placement, quantity, active state, property values
3. Add visual distinction with link-badge icon for room instances.
4. Add guardrails and regression coverage for both `GroupedStack` and `IndividualInstances`.

Exit criteria:
1. Producers can place linked room instances from existing quantifiable base items.
2. Linked-action restrictions are enforced and clearly communicated in UI.
3. Behavior is verified for both quantifiable distribution modes.

Status:
1. In progress.
2. Done: room-instance linkage metadata foundation added (stable object identity + base-item linkage + linked-actions mode flag) and persisted through project save/load.
3. Done: linked room-instance action editing is now blocked in scoped-action workflows with clear status guidance to edit on the base item.
4. Done: hierarchy now shows a link-badge visual cue for room instances to distinguish them from base items.
5. Done: runtime mapping resolves linked room-instance actions from the base item for both `GroupedStack` and `IndividualInstances`.
6. Done: clean export resolves linked room-instance actions from the base item, preventing stale local action leakage.
7. Done: focused regressions added/validated for grouped + individual linked-action inheritance behavior.

## 10.1) Current Implementation Snapshot
1. Q1 is complete.
2. Q2 is complete.
3. Q3 is complete.
4. Q4 is complete.

## 11) Definition of Done
1. Grouped and individual quantifiable runtime behaviors are explicit and validated.
2. Composite build/break for quantifiable parts preserves deterministic restoration.
3. Source-out-of-scope break restoration works without item loss.
4. Child forwarding single-vs-all similar dispatch is implemented and tested.
5. Documentation and plan status updated with implementation evidence.
6. Linked room-instance placement from existing quantifiable base items is implemented and validated for both distribution modes.
