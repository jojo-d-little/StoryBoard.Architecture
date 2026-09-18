# Traversal Assistance Plan

Status: Complete (Topic A/B/C scope complete; v2 questions deferred)
Owner: StoryboardDesigner authoring + Storyboard.Shared runtime
Last updated: 2026-07-05

Review note (2026-07-05):

1. Topic A, Topic B v1, and Topic C locks and implementation slices are complete.
2. Remaining potential work is explicitly v2/deferred and moved to plans/future/CONSOLIDATED_OUTSTANDING_PLAN.md.

Related side-quest plan:

1. Globals sidecar/import refresh workflow: see plans/future/GLOBALS_SIDECAR_IMPORT_PLAN.md.

## 1. Purpose

Define targeted authoring assistance behaviors for traversal workflows that prioritize deterministic outcomes and data integrity.

This plan currently covers three foundational topics:

1. Room movement behavior when traversals already exist.
2. Traversal Wizard bootstrap to accelerate typical traversal setup.
3. Room-tree traversal-leg projection for contextual leg editing.

## 2. Problem Statement

Traversal connections are tightly coupled to authored room layout intent.

When a room is moved after traversal definitions are authored, attempting automatic traversal correction is high risk because it can silently invalidate directional intent, openable side semantics, and shared-open-state pairings.

## 2.1 Current Focus

This plan is the active priority for traversal-related work.

Decision-lock phase is complete. Current focus is incremental implementation of Topic B wizard behavior using the locked contracts below.

## 2.2 Implementation Kickoff (2026-07-02)

Implementation has started with room-level command wiring for Traversal Wizard invocation from the hierarchy tree.

Immediate implementation slices:

1. Done: add room-level wizard ViewModel state model for 8 direction rows with disabled/read-only reason states.
2. Done: implement deterministic row analysis for immediate neighbors and existing traversal keep-state detection.
3. Done: add wizard dialog UX with inline row-state preview and bulk controls (Select All/Clear All).
4. Done: implement apply pipeline (add-only) with strict duplicate guard and explicit skip reasons.
5. Done: implement generated artifacts:
   - door naming: Door_{Direction} with room-local collision suffixing
   - door notes: "Door opens {Direction} from {SourceRoomName} to {DestinationRoomName}."
   - leg look action: "There looks to be a {DestinationRoomName} thru the door"
6. Done: implement post-apply feedback channels:
   - toast-style summary message
   - persistent output/diagnostics details with structured counts.
7. Done: add focused tests for locked contracts and under-1-second apply target in v1 single-room scope.

Status note (2026-07-02):

1. Wizard now executes add-only apply from room context action and keeps existing traversals unchanged.
2. v1 generated door artifacts and traversal-leg look seeds are active.
3. Focused utility, ViewModel, apply-pipeline, and projection regression tests are green.

## 3. Goals (Initial Topic)

1. Prevent silent traversal corruption when rooms are moved.
2. Keep move behavior explicit and producer-controlled.
3. Ensure post-move project state is deterministic and export-safe.
4. Keep implementation small and low-risk for first release.

## 4. Non-Goals (Initial Topic)

1. No automatic traversal remapping based on new room geometry.
2. No heuristic preservation of adjacency or directional intent.
3. No background repair of shared open-state pairings.
4. No broad traversal redesign in this phase.

## 5. Policy Lock: Room Movement With Existing Traversals

When a room with one or more connected traversals is moved:

1. Warn before committing the move.
2. Offer explicit option to cancel and restore original position.
3. If user confirms move, delete all traversals connected to that room.
4. Treat move plus traversal deletion as one atomic undoable operation.

Rationale:

1. Avoid silent, potentially wrong auto-repair.
2. Preserve deterministic author intent.
3. Ensure no stale or ambiguous traversal data survives.

## 6. UX Contract (Phase A)

## 6.1 Trigger Condition

Show destructive warning when a room move would affect at least one traversal connection.

## 6.2 Warning Dialog

Required actions:

1. Put it back (cancel move).
2. Move and delete traversals (confirm).

Required warning message intent:

1. Moving this room will delete all traversals connected to it.
2. This includes traversal-side openable pairing semantics tied to removed traversals.

## 6.3 Confirm Path Behavior

On "Move and delete traversals":

1. Keep new room position.
2. Delete every traversal connection where moved room is either endpoint.
3. Remove connection-local shared-open-state pairing configuration for removed connections.
4. Emit one non-blocking authoring info summary (count removed traversals).

## 6.4 Cancel Path Behavior

On "Put it back":

1. Restore the room to its original pre-drag position.
2. Leave all traversals unchanged.

## 6.5 Undo/Redo

1. One Undo restores both original room position and deleted traversals.
2. One Redo reapplies move and traversal deletion.

## 7. Data and Validation Contract

## 7.1 Deterministic Deletion Rule

Deletion target set:

1. All traversal connections with RoomAId == movedRoomId or RoomBId == movedRoomId.

## 7.2 Integrity Guarantees

After confirmed move:

1. No traversal references remain to moved room unless newly created later by author.
2. No dangling shared-open-state configuration remains for removed connections.
3. Save/export should not surface stale traversal diagnostics caused by the move itself.

## 8. Implementation Phases (Initial Topic)

## Phase A1: Decision and UX Lock

Deliverables:

1. Final dialog copy and action labels locked.
2. Trigger point in room drag/move workflow identified.

Validation gate:

1. Team signoff on destructive workflow and undo semantics.

## Phase A2: Editor Workflow Implementation

Deliverables:

1. Warning dialog integrated into room-move workflow.
2. Cancel/confirm paths implemented.
3. Traversal deletion service path added for moved-room endpoint match.
4. Atomic undo transaction for move plus deletion.

Validation gate:

1. Build passes.
2. Focused room-move workflow tests pass.

## Phase A3: Regression Coverage and Hardening

Deliverables:

1. Tests for cancel behavior, confirm behavior, and undo/redo atomicity.
2. Tests for deterministic deletion set.
3. Tests proving no dangling traversal/shared-mode state after confirm.

Validation gate:

1. StoryboardDesigner.App.Tests targeted suite passes.
2. Existing traversal regression tests remain green.

## 9. Acceptance Criteria (Initial Topic)

1. Moving a room with traversals always prompts the warning dialog before commit.
2. Cancel path restores original room position exactly.
3. Confirm path deletes all traversals connected to moved room.
4. Confirm path leaves no stale connection-local shared-open-state artifacts.
5. Move plus deletion is a single atomic undo/redo unit.
6. Save and clean export remain deterministic after confirmed move.

## 10. Test Strategy (Initial Topic)

1. Unit tests for deletion target set selection.
2. ViewModel/workflow tests for warning dialog branch behavior.
3. Undo/redo tests validating single-transaction behavior.
4. Regression tests for traversal command behavior after affected traversals are removed.

## 11. Risks and Mitigations

1. Risk: users may accidentally delete many traversals.
   - Mitigation: explicit destructive warning and clear action labels.
2. Risk: partial delete can create dangling state.
   - Mitigation: single deletion pass using endpoint match and post-operation integrity assertions.
3. Risk: user confusion after confirm.
   - Mitigation: deterministic summary message with removal count and undo availability.

## 12. Decision Lock (Topic A)

Status key:

1. Locked: approved for implementation.

| Decision Point | Locked Decision | Status |
| --- | --- | --- |
| Warning detail | Warning dialog includes exact count of traversals that will be deleted before confirm. | Locked |
| Permission handling | Operation follows standard edit-permission handling (no additional special lock layer in v1). | Locked |
| Post-confirm feedback | Show toast summary plus diagnostics panel entry with deterministic removal counts. | Locked |

Topic A completion note:

1. Topic A planning lock is complete and implementation-ready.

## 13. Topic B: Traversal Wizard Bootstrap

## 13.1 Problem Statement

After room placement (initial authoring or post-move rework), producers often need to rebuild many typical traversals, openable associations, and shared-property links manually.

Authoring this from scratch is repetitive and error-prone.

## 13.2 Goal

Provide a producer-invoked wizard popup that makes a room easy to connect to nearby rooms using a minimal deterministic input set, then lets producers tune details after apply.

## 13.3 Non-Goals (Topic B)

1. No background auto-authoring without explicit wizard confirmation.
2. No mutation of existing traversals unless producer explicitly chooses replacement behavior.
3. No attempt to infer every puzzle-specific or custom gameplay rule.
4. No hidden rewrites of existing shared-property relationships.

## 13.4 Wizard Contract (Phase B)

## 13.4.1 Entry Points

1. Room-level action: "Traversal Wizard" for the selected room.
2. Wizard processes one room at a time in v1.

## 13.4.2 Analysis Inputs

1. Selected source room (single-room scope).
2. Effective traversal mode (project/area/room/leg precedence).
3. Room coordinates and adjacency slots (4-way or 8-way based on effective layout).
4. Existing traversals for collision checks only.

## 13.4.3 Suggested Outputs

1. Candidate traversal connections for selected directional slots marked Yes by producer.
2. Fixed v1 traversal access baseline of TwoWay for each selected candidate.
3. Door mode per selected direction: door Yes/No (No means open traversal without seeded door endpoints).
4. When door is Yes, wizard creates one door endpoint object in each connected room.
5. When door is Yes, wizard links/shares door endpoint isOpen state with corresponding traversal-leg isPassable state by default.
6. When door is Yes, wizard supports door behavior mode per selected direction: Together (as one) or Per-side.
7. Seed one default Look action per traversal leg: "There looks to be a {DestinationRoomName} thru the door".
8. For generated door endpoints, assign deterministic room-local names using Door_{Direction} with numeric suffixing for same-room collisions.
9. For generated door endpoints, populate producer notes with readable routing context using: "Door opens {Direction} from {SourceRoomName} to {DestinationRoomName}."
10. Deterministic skip diagnostics for blocked/conflicting candidates.

## 13.4.4 Producer Control and Safety

1. Preview-first: wizard shows proposed additions/changes before apply.
2. Producer can accept all, accept subset, or cancel.
3. Apply operation is one atomic undoable transaction.
4. Conflicts are never auto-resolved silently; they are surfaced as explicit choices.

## 13.4.5 v1 Input Surface Lock (2026-07-02)

Wizard interaction in v1 is a popup dialog with only these producer inputs:

1. Wizard always renders all 8 direction rows (N, NE, E, SE, S, SW, W, NW).
2. Rows with no immediately adjacent sibling room are disabled and cannot be enabled.
3. In 4-way default mode, diagonal rows are shown but disabled by default.
4. In 4-way default mode, if a diagonal room is immediately adjacent, producer may explicitly override and enable that diagonal direction.
5. For each slot set to Yes: door Yes or No (default Yes).
6. If door is Yes: door behavior mode Together (as one) or Per-side (default Together).
7. If door is Yes: default door state Open or Closed (default Closed).
8. If door is Yes and default state is Closed: default lock state Locked or Unlocked (default Locked).
9. If a direction already has an authored traversal, wizard defaults to Keep Existing for that direction.
10. Directions with existing authored traversals render as informational/read-only rows that clearly state "Traversal already defined - left unchanged".

No additional per-connection wizard inputs are required in v1.

## 13.5 Rule Set v1 (Deterministic Baseline)

1. Wizard considers only immediate neighboring rooms in the 8-direction grid around the selected room.
2. Do not generate duplicate reciprocal rows; emit one canonical connection per unordered room pair.
3. If a direction already has an authored traversal, preserve it by default (Keep Existing) and do not generate a replacement in add-only mode.
4. In 4-way default mode, cardinal directions remain primary defaults; diagonals may still be explicitly enabled only when an immediate diagonal neighbor exists.
5. Generated traversal rows in v1 use fixed defaults that are easy to override: TwoWay, door Yes, Together mode, Closed, Locked.
6. If producer selects door No, wizard creates an open traversal and skips door endpoint/link setup for that direction.
7. If producer selects door Yes, wizard creates new door endpoints on both connected rooms for that generated traversal.
8. If producer selects door Yes, shared linkage from door isOpen to leg isPassable is enabled by default.
9. For each generated traversal leg, seed exactly one default Look action referencing the opposite room name.
10. Generated door object names use deterministic room-local format Door_{Direction} with numeric suffixing for same-room collisions.
11. Generated door producer notes use deterministic readable format: "Door opens {Direction} from {SourceRoomName} to {DestinationRoomName}."
12. Deterministically order suggestions by RoomAId, RoomBId, then base direction.
13. Existing-direction rows include explicit UI status text so producers know no additional wizard inputs are needed for those directions (for example, "Traversal already defined - left unchanged").
14. Replacement is out of scope for v1 wizard runs; producers who want replacement must manually delete existing traversal(s) first, then rerun wizard.

## 13.5.1 v1 Baseline Lock (2026-07-02)

For v1, the wizard applies one simple traversal profile only:

1. TwoWay traversal.
2. Door enabled by default.
3. Door behavior default is Together (as one).
4. Door state defaults to Closed and Locked.

Producers can quickly override door mode and door state per direction in the wizard.

Any deviation from this baseline is performed by the producer after wizard apply.

## 13.5.2 v1 Door and Leg Seed Lock (2026-07-02)

For each generated traversal in v1:

1. If door is Yes, wizard creates two new door endpoints (one in each connected room).
2. If door is Yes, wizard links/shares door isOpen with traversal-leg isPassable by default.
3. If door mode is Together, both sides operate as one shared pair; if Per-side, each side uses its own door behavior/state.
4. Generated door endpoint names use Door_{Direction} with room-local collision suffixing (for example Door_N, Door_N_2).
5. Generated door producer notes use: "Door opens {Direction} from {SourceRoomName} to {DestinationRoomName}."
6. Each traversal leg gets one seeded Look action with text: "There looks to be a {DestinationRoomName} thru the door".

Door reuse/discovery is deferred; producers may rename, replace, or delete generated door objects after apply.

## 13.6 Phase B Implementation Plan

## Phase B1: Rule and UX Lock

Deliverables:

1. Final v1 adjacency and suggestion rules.
2. Wizard options contract (add-only default, optional replace mode).
3. Preview grid/list schema with deterministic ordering.

Validation gate:

1. Team signoff on no-silent-mutation behavior.

## Phase B2: Suggestion Engine

Deliverables:

1. Deterministic suggestion service for candidate traversals.
2. Duplicate/conflict filtering against existing authored state.
3. Optional openable/shared suggestion generation with strict eligibility checks.

Validation gate:

1. Unit tests for adjacency analysis, filtering, and ordering.

## Phase B3: Wizard UI and Apply Workflow

Deliverables:

1. Producer preview and selective apply UX.
2. Atomic apply transaction and undo/redo integration.
3. Diagnostics summary for skipped/conflicting candidates.

Validation gate:

1. Workflow tests for cancel, partial apply, full apply, and undo.

## Phase B4: Regression and Hardening

Deliverables:

1. Regression coverage for add-only mode safety.
2. Regression coverage for optional replace mode guardrails.
3. Performance check at representative room counts using v1 single-room bounds (up to 8 traversal rows and up to 16 created door endpoints).

Validation gate:

1. Build passes.
2. Traversal and export regression suites remain green.
3. Apply operation completes in under 1 second after producer confirms input (v1 single-room scope).

## 13.7 Acceptance Criteria (Topic B)

1. Producer can run wizard without mutating data until explicit apply.
2. Suggested traversal set is deterministic for identical inputs.
3. Wizard does not create duplicate canonical connections.
4. Existing authored traversals are preserved in default add-only mode.
5. Apply and undo are atomic and reversible.
6. When door is Yes, shared door-isOpen to leg-isPassable linkage is applied by default; when door is No, no door/shared linkage is created.

## 13.8 Open Questions (Topic B)

1. Should replace mode in v2 be row-level in wizard, or remain outside wizard scope longer?
2. Should room-subset batch execution be added in v2, or remain strictly room-by-room longer?
3. Should v2 add additional shared-property templates beyond the default door isOpen <-> leg isPassable linkage?

## 13.9 Design Points To Lock (Topic B)

Use this as the deferred decision ledger before implementation begins.

Status key:

1. Open: decision not yet locked.
2. Candidate Default: recommended starting point for first implementation.
3. Locked: approved for implementation.

| Design Point | Why It Matters | Options | Candidate Default | Status |
| --- | --- | --- | --- | --- |
| Wizard scope entry level | Controls UX complexity and graph-size performance risk | Single selected room; room subset batch; area-wide batch | Single selected room | Locked |
| Mutation mode in v1 | Defines safety boundary for authored data | Add-only; Add + replace | Add-only | Locked |
| Replace behavior guardrail | Determines risk of unintended rewrites | No replace in v1; explicit replace mode with per-row confirmation | No replace in v1 (manual delete-first workflow for replacement) | Locked |
| Shared-property suggestion default | Controls accidental coupling risk | Off by default; On by default | On by default when door is Yes (door isOpen <-> leg isPassable linkage) | Locked |
| Openable binding inference strictness | Balances convenience vs false-positive wiring | Strict (only unambiguous); permissive heuristic; none | No inference; explicit producer door choices in wizard | Locked |
| Traversal access default for generated candidates | Establishes baseline gameplay assumptions | Fixed profile; mixed/template-driven; prompt per run | Fixed profile defaults: TwoWay + door Yes + Together + Closed + Locked | Locked |
| Door endpoint strategy for generated traversals | Affects determinism and implementation complexity | Always create new doors; attempt reuse; mixed with fallback | Create new door endpoints on both rooms when door is Yes | Locked |
| Auto-seeded traversal-leg action content | Defines immediate usability after apply | No seeded action; single default Look action; multi-action template | Single default Look action: "There looks to be a {DestinationRoomName} thru the door" | Locked |
| Adjacency rule source | Ensures deterministic candidate generation | Effective mode only; immediate-neighbor only with 8-row UI; custom radius | Immediate-neighbor only with always-visible 8-row UI and 4-way diagonal override only when adjacent | Locked |
| Existing traversal collision handling | Prevents duplicate/contradictory authored state | Keep existing by default; overwrite existing; prompt per collision | Keep existing by default with deterministic skip/keep reason | Locked |
| Existing-direction UI clarity | Prevents confusion during room-by-room runs with partial setup | Silent skip; row-level message; row-level read-only state plus message | Row-level read-only state plus explicit "Traversal already defined - left unchanged" message | Locked |
| Suggestion ordering contract | Needed for deterministic previews and tests | Stable deterministic sort; UI-collection order | Stable deterministic sort: fixed direction order (N, NE, E, SE, S, SW, W, NW), with destination room ID as deterministic fallback tie-breaker (typically no-op in single-room mode) | Locked |
| Wizard input surface complexity | Drives usability and support burden | Minimal directional toggles + conditional door state fields; expanded advanced fields | Minimal directional toggles + conditional door Yes/No, Together/Per-side, Open/Closed, and Locked/Unlocked | Locked |
| Preview granularity | Impacts producer confidence and correction speed | Separate preview step; inline row-state preview | Inline row-state preview (no separate preview step in v1) | Locked |
| Apply transaction boundary | Affects undo reliability and partial failure behavior | One atomic apply; batched per room; incremental apply | One atomic apply (single-room v1 scope makes batched-per-room equivalent/no-op) | Locked |
| Diagnostics surfacing | Determines fixability of skipped candidates | Toast only; panel only; both | Both: toast summary + persistent output/diagnostics details | Locked |
| Performance envelope target | Prevents UI regressions on large maps | No hard target; target room counts and max suggestion count | Under 1 second from apply-confirm to completion in v1 single-room scope (up to 8 traversal rows / 16 doors) | Locked |
| Template support in v1 | Impacts extensibility and initial complexity | No templates; built-in single template; user-defined templates | No templates in v1 (prescriptive wizard only) | Locked |
| Telemetry/usage capture | Helps future tuning of defaults | None; opt-in local counters; full analytics | None for v1; revisit opt-in local counters in v2 after workflow stabilizes | Locked |

## 13.10 Pre-Implementation Lock Checklist (Topic B)

Before coding Topic B, lock these decisions explicitly in this file:

1. Scope entry level and mutation mode.
2. Shared/openable suggestion defaults and strictness.
3. Collision handling, ordering contract, and apply transaction boundary.
4. Diagnostics channel and performance target.
5. Template strategy for v1.

Implementation start criterion:

1. No Open decision remains that changes data mutation semantics or deterministic output behavior.

## 13.11 Additional v1 Wizard Decisions (To Be Resolved)

Use this ledger to walk through optional v1 usability features one by one and lock intentionally.

Status key:

1. Open: decision not yet locked.
2. Candidate Default: recommended starting point.
3. Locked: approved for implementation.

| Decision Point | Why It Matters | Options | Candidate Default | Status |
| --- | --- | --- | --- | --- |
| Preview row detail level | Producer confidence before apply | Minimal per-direction state; full per-row output summary | Inline row state is the preview in v1; no separate preview mode | Locked |
| Skip-reason surfacing | Reduces confusion when rows are not applied | Hide skips; show generic count; show deterministic per-row reason | Deterministic per-row reason with inline disabled-row message (for example "Traversal already defined") | Locked |
| Created door naming convention | Affects readability and cleanup effort | No naming convention; deterministic generated names; prompt user names | Deterministic room-local names: Door_{Direction} with collision suffixing | Locked |
| Generated door producer notes content | Improves author readability and traceability | No notes; short static note; readable source-to-destination note | "Door opens {Direction} from {SourceRoomName} to {DestinationRoomName}." | Locked |
| Bulk selection controls | Speeds common workflows | Per-row toggles only; add Select All/Clear All controls | Add Select All and Clear All | Locked |
| Post-apply summary detail | Confirms operation outcome | Toast only; diagnostics only; both with structured counts | Both with structured counts | Locked |
| Post-apply focus behavior | Impacts discoverability of generated content | No focus change; focus source room; focus first created leg | Focus source room and expand Traversal Legs | Locked |
| Add-only idempotency guard | Prevents duplicate artifacts on rerun | Best effort; strict duplicate guard with explicit skip | Strict duplicate guard with explicit skip | Locked |
| Undo guarantee messaging in wizard | Reduces perceived risk | No explicit note; inline note in dialog; confirmation-only note | Inline note in dialog | Locked |
| Preview/apply interaction model | Controls accidental mutation risk | Single-step immediate apply; separate preview then apply; inline row-state then apply | Inline row-state then explicit apply (no separate preview mode) | Locked |
| Large-area performance UX guard | Protects responsiveness | No cap; soft warning; hard cap with override | Not applicable in v1 single-room scope; revisit only if multi-room execution is introduced | Locked |

Working agreement for this ledger:

1. These are optional v1 usability decisions and should not expand core mutation semantics beyond locked Topic B rules.
2. Lock decisions in this section only after explicit review in sequence.

## 13.12 Future Template Concepts (Post-v1)

Template support is intentionally excluded from v1. Candidate future uses:

1. Fast style presets for generated door behavior (for example "open corridor", "locked checkpoint", "puzzle gate").
2. Team-level naming and messaging conventions (door labels, default look text variants).
3. Optional policy bundles that prefill Together/Per-side and open/locked defaults while still allowing per-row overrides.
4. Environment-specific presets (for example indoor/hallway versus outdoor/pathway flavor).
5. Studio-specific authored template packs after core v1 workflow proves stable.

## 14. Topic C: Room-Tree Traversal-Leg Projection

## 14.1 Problem Statement

Traversal ownership is correctly area-level, but producers also need room-centric discoverability and editing context for per-leg behaviors (for example, leg-specific actions and echo content).

## 14.2 Core Model Lock

1. Traversal connections remain canonical area-level data.
2. Room-tree traversal-leg nodes are projected views over canonical traversal-side data.
3. No duplicate traversal persistence is introduced under room nodes.

## 14.3 Tree Interaction Lock

1. Room tree always displays a Traversal Legs folder per room.
2. Room tree traversal-leg nodes support contextual edits only (for example, leg actions, leg-side messaging, diagnostics review).
3. Room tree traversal-leg nodes do not support add/remove traversal operations.
4. Create/remove traversal connection operations remain map/area designer only.

## 14.4 MVVM Projection Contract

1. Same underlying model is exposed through two viewmodels (area-centric and room-centric projections).
2. Edits from either projection route to the same command/mutation paths.
3. Projection nodes maintain stable identity keys (connectionId + side) for deterministic selection/expansion and undo behavior.

## 14.5 UX and Command Semantics

1. If producer attempts traversal create/delete from room tree context, action is unavailable (disabled/hidden).
2. Room-tree UI includes clear affordance to open corresponding map-designer traversal editor for structural edits.
3. Leg projection label includes destination room context to reduce ambiguity.
4. Leg projection node primary label is traversal direction name (for example North, South, East, West).
5. Leg projection node tooltip includes destination room name.
6. Double-click on room-tree traversal leg node shows guidance popup: structural traversal edits are done in map/area designer.
7. Room-tree leg projection edit scope in v1 is limited to actions and game-properties/variables only.
8. Map-jump action opens/focuses the corresponding area map editor and ensures the related area context is loaded.
9. After map-jump handoff, user performs manual map navigation to locate and edit the target traversal structurally.

## 14.6 Acceptance Criteria (Topic C)

1. Traversal legs are visible under room tree as projected children with stable identity.
2. Editing leg-side actions/messages from room tree updates the canonical area traversal data.
3. No tree path exists to create or delete traversal connections from room context.
4. Map-designer create/delete operations are immediately reflected in room-tree leg projections.
5. No projection drift occurs after save/load, undo/redo, or multi-edit sequences.

## 14.7 Design Points To Lock (Topic C)

| Design Point | Why It Matters | Options | Candidate Default | Status |
| --- | --- | --- | --- | --- |
| Projection node placement | Discoverability vs tree noise | Always show folder; show only when legs exist | Always show folder | Locked |
| Projection ordering | Deterministic authoring and tests | Direction order; destination-name order; connection-id order | Canonical direction order, then destination name, then connection id tie-breaker | Locked |
| Structural-edit affordance | Reduce confusion around where to add/remove traversals | Disabled commands; hidden commands; read-only notice | Disabled commands + map jump + double-click guidance popup | Locked |
| Leg edit scope in room tree | Prevent accidental structural edits | Actions plus game-properties/variables only; plus lock/open state; full side edit | Actions plus game-properties/variables only in v1 | Locked |
| Map-jump affordance | Fast handoff to structural editor | Context-menu action; inline link button; both | Both context-menu and inline button; opens/focuses target area map editor only (manual map navigation afterward) | Locked |

## 14.8 Tree IA Normalization (UI-Only Projection)

To reduce tree noise and improve consistency, apply a visual grouping pattern to high-fanout nodes.

Scope (v1):

1. Country nodes.
2. Area nodes.
3. Room nodes.
4. Game object nodes.

Layout rule:

1. Each scoped node exposes exactly two immediate UI children:
   - Configuration
   - Children
2. Configuration contains editor-facing/configuration entries currently shown directly under the node.
3. Children contains structural descendants currently shown directly under the node.

Hard boundaries:

1. This is a UI projection and navigation structure change only.
2. No persistence/model ownership changes.
3. No scope-chain/runtime contract changes implied by this grouping.

Acceptance criteria (Tree IA):

1. Scoped nodes render only Configuration and Children as immediate children.
2. Existing editor actions remain reachable with no loss of capability.
3. Existing selection/edit workflows remain deterministic after save/load.
4. No serialization/export changes occur from this UI-only refactor.

Design points to lock (Tree IA):

| Design Point | Why It Matters | Options | Candidate Default | Status |
| --- | --- | --- | --- | --- |
| Empty group rendering | Balances discoverability and noise | Always show both groups; show only non-empty groups | Always show both groups for consistency | Locked |
| Group ordering | Predictable scanning and keyboard nav | Configuration first; Children first | Configuration first | Locked |
| Context menu targeting | Avoid ambiguity in add actions | Add actions on parent; add actions on Children group; both | Both parent and Children group route to same command | Locked |
| Migration of existing tree tests | Prevent regressions in navigation and selection | Incremental updates; full tree baseline refresh | Full baseline refresh + targeted regressions | Locked |

## 14.9 Pre-Implementation Lock Checklist (Topic C)

1. Finalize projection node visibility and ordering rules.
2. Finalize exactly which leg properties are editable in room tree v1.
3. Finalize structural-edit affordance behavior and map-jump UX.
4. Confirm test matrix for projection consistency and no-drift guarantees.
5. Lock Tree IA group behavior (empty-group rendering, ordering, and context-menu routing).

Implementation start criterion:

1. Core model lock and tree interaction lock remain unchanged during implementation.

## 14.10 Topic C Implementation Delta (2026-07-02)

Completed in current implementation slice:

1. Room-tree projection nodes shipped:
   - Configuration and Children groups applied to scoped nodes in tree IA path.
   - Room-level Traversal Legs folder rendered as always-visible projection container.
   - Traversal leg nodes rendered with direction label and destination tooltip context.
2. Structural-edit boundary affordances shipped:
   - Room-tree leg context menu includes map-handoff action.
   - Inline map button added on traversal leg projection rows.
   - Double-click on traversal leg opens guidance popup and hands off to area map editor.
3. Context routing updates shipped:
   - Children group add-actions route to the same owner-node mutations as parent actions.
   - Scoped action resolution supports traversal leg projected action editing.
4. Persistence and undo-snapshot safety updates shipped:
   - Traversal leg available actions are included in snapshot deep-copy and JSON project state mapping.

Validation completed for this slice:

1. `dotnet build .\StoryboardDesigner.slnx` passed after lock-release of running app host.
2. Focused guardrail tests passed:
   - `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "AreaNavigationEditorTabViewModelTests|ArchitectureSeparationGuardrailsTests"`
3. Topic C targeted tests added and passed:
   - `RoomTreeTraversalProjectionTests` covers group-node layout contract and traversal-leg projection ordering contract.
4. Runtime-focused validation gate now passes after triage/fix:
   - `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`
5. Follow-up regressions resolved in same slice:
   - Restored runtime invocation-context token seeding so `currentRoom.*` shortcut echo references resolve again.
   - Refreshed Birmingham playback recording snapshots using `UPDATE_PLAYBACK_SNAPSHOTS=1` to match current deterministic clean-export output.

## 14.11 Echo Editor Single-Source Clarification (2026-07-02, Historical)

Historical note:

1. This section captures pre-outcome-channel behavior during an intermediate migration phase.
2. Final state now uses canonical `SuccessEchoMessage`/`FailureEchoMessage`; legacy `OptionalEchoMessage` has been removed from production paths.

Issue addressed:

1. EchoMessage actions exposed two editable message paths in the dialog (`EchoMessage` main panel plus `OptionalEchoMessage` via top button), which read like preview-vs-editor for one field and could produce duplicated simulator output.

Behavior change applied:

1. For `EchoMessage` action type, the top action-echo button now edits the same primary `EchoMessage` field.
2. `EchoMessage` action save path clears `OptionalEchoMessage` to prevent dual-script persistence for this action type.
3. Runtime execution now skips optional-echo evaluation when action type is `EchoMessage`, so older projects with both fields populated no longer emit duplicate lines.

Validation completed:

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameCommandProcessorLinkedActionsTests.Process_SetVariableAction_AlsoEmitsOptionalEcho|GameCommandProcessorLinkedActionsTests.Process_HighDiagnostics_EmitsResolvedTokenTableOnlyOncePerCommand_ForEchoMessageWithoutSecondaryOptionalOutput|GameCommandProcessorLinkedActionsTests.Process_ForwardsCommandToTraversalLegChildren_AndResolvesSelfIsPassable|RoomTreeTraversalProjectionTests"`

## 15. Immediate Lock Sequence (Execution Order)

Use this order to finalize the plan before any traversal-adjacent feature work resumes.

## 15.1 Lock Set A (Topic A: Room Move Safety)

Status: Complete

1. Warning dialog includes exact deletion count before confirm.
2. Operation follows standard edit-permission model (no special lock layer in v1).
3. Post-confirm summary goes to toast plus diagnostics panel.

Exit criterion:

1. Topic A open questions are resolved and marked Locked.

## 15.2 Lock Set B (Topic B: Wizard v1 Scope)

Status: Complete (v1)

1. Scope entry: single selected room only in v1.
2. Mutation mode: add-only in v1.
3. Replace mode: no replace in v1; manual delete-first workflow when replacement is desired.
4. Shared linkage default: when door is Yes, door isOpen <-> leg isPassable linkage is on by default.
5. Openable inference: none in v1; wizard uses explicit producer door choices.
6. Generated traversal defaults: TwoWay + door Yes + Together + Closed + Locked.
7. Wizard input surface: per-direction Yes/No, then door Yes/No, Together/Per-side, Open/Closed, and (if Closed) Locked/Unlocked.
8. Existing authored directions default to Keep Existing in add-only mode.
9. Door endpoint strategy: create new endpoints on both connected rooms when door is Yes.
10. Door naming convention: Door_{Direction} with room-local collision suffixing.
11. Door producer notes convention: "Door opens {Direction} from {SourceRoomName} to {DestinationRoomName}."
12. Leg action seed: create one default Look action per leg using destination room name.
13. Existing-direction UI rows are read-only and explicitly labeled as traversal already defined/left unchanged.
14. Direction rows: always show 8; non-adjacent rows disabled; in 4-way defaults diagonals are disabled unless producer explicitly overrides where a diagonal neighbor exists.
15. Ordering: deterministic stable sort using fixed direction order (N, NE, E, SE, S, SW, W, NW) with destination room ID fallback tie-breaker.
16. Diagnostics surfacing: toast summary plus persistent output/diagnostics details.
17. Apply: one atomic transaction with single undo step.
18. Performance target: under 1 second from apply-confirm to completion in v1 single-room scope.
19. Template support: none in v1 (prescriptive wizard behavior only).
20. Telemetry/usage capture: none in v1.

Exit criterion:

1. Topic B table rows impacting data mutation or determinism are marked Locked.

Validation note (2026-07-02):

1. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "AreaNavigationEditorTabViewModelTests|GameStateSessionPlayerScopeTests|TraversalValidationServiceTests|CompositeBuildActionTests|TraversalWizardApplyPipelineTests"` passed (57/57).
2. xUnit2031 warning sites in the above test classes were refactored to predicate overload assertions.
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"` passed (63/63).
4. Test-project restore warning noise from external vulnerability-feed lookup was removed by setting `NuGetAudit=false` in `StoryboardDesigner.App.Tests.csproj`; subsequent focused and runtime gate reruns completed without NU1900 output.
5. Full-suite confidence rerun initially surfaced one deterministic clean-export snapshot drift in `JsonExportServiceCleanExportSnapshotTests.ExportCleanProjectV1_MatchesSingleRoomSnapshotBaseline`; baseline was intentionally refreshed via `UPDATE_CLEAN_EXPORT_SNAPSHOTS=1` and revalidated without env override.
6. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj` now passes end-to-end (243/243).

## 15.3 Lock Set C (Topic C: Projection UX Boundaries)

Status: Complete

1. Room-tree leg nodes remain edit-only projections.
2. No create/delete traversal operations in room tree.
3. Room tree supports direct jump/open to map traversal editor for structural edits.
4. v1 leg edit scope in room tree: actions and game-properties/variables only.
5. Tree IA normalization uses exactly two immediate UI children (Configuration, Children) for country/area/room/game object nodes.

Exit criterion:

1. Topic C command boundary and edit-scope rows are marked Locked.

## 15.4 Start-Implementation Gate

Implementation may begin only when:

1. Lock Set A, B, and C exit criteria are all met.
2. No Open item remains that can change persistence shape, mutation semantics, or deterministic ordering.
