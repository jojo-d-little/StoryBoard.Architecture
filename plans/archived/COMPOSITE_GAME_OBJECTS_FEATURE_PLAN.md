# Composite Game Objects Feature Plan

Status: Proposed
Owner: Pending
Last updated: 2026-06-29

## 1) Objective
Add first-class composite game object behavior so producers can author objects that are assembled from parts (and optionally broken back down), with minimal scripting burden.

Primary example:
1. Two broken key parts are active in gameplay.
2. A full key object exists as defined data but starts inactive.
3. Build operation consumes/deactivates parts and activates full key.
4. Optional reverse operation deactivates full key and reactivates parts.

## 2) Core Design Principles
1. Producer-first authoring: common recipes should be configured, not scripted.
2. Runtime consistency: activation state controls whether objects participate in gameplay.
3. Deterministic behavior: build/break outcomes are explicit and testable.
4. Boundary safety: reusable runtime logic should live in Storyboard.Shared.
5. Additive rollout: start with clear known-intent build commands, then expand discovery flows.

## 3) Feature Scope
In scope (v1):
1. New action types:
- BuildCompositeByTarget
- BuildCompositeByParts
- BreakCompositeItem
2. Composite recipe definition on target composite object.
3. Activation-state transitions for source parts and target composite item.
4. Inventory-based validation for required part presence.
5. Command routing for explicit known-intent target case (example: "fix key").
6. Command routing for explicit parts-driven case (example: "rub cloth on lamp").
7. Canonical multi-part parsing form (example: "use object A, object B, object C").
8. Deterministic ambiguity policy when part sets map to multiple recipes.
9. Optional reversibility per recipe.

Out of scope (v1):
1. Broad freeform crafting graph search beyond explicitly configured recipes.
2. Multi-step workstation/tool requirements.
3. Hint/progression UX for unknown recipes beyond deterministic command resolution.
4. Hybrid part semantics where one/more specific parts are always required and additional parts are satisfied by MinimumCount from an optional pool.

## 4) Terminology (Feature-Specific)
1. Composite Target
- The object produced by assembly.

2. Component Parts
- The required source objects for assembly.

3. Build Recipe
- Author-defined mapping: required parts -> composite target.

4. Part Requirement Mode
- AllRequired: every configured required part must be available.
- MinimumCount: at least X configured candidate parts must be available.

5. Build Intent Mode
- TargetIntent: player specifies intended composite target.
- PartsIntent: player specifies component parts and runtime resolves target recipe.

6. Reversible Recipe
- Recipe permitting inverse operation (break).

7. Active Participation
- Object is considered present/usable by runtime matching and inventory interactions.

8. Quantifiable Object
- A game object definition that supports counted quantities in world placements and inventory (example: flowers).

9. Quantity Stack
- Aggregated count for a quantifiable object in a container/scope (example: inventory shows "Flowers x5").

10. Verb-Driven Command Binding
- Commands begin with producer-authored verbs (example: build, break, fix) and resolve to actions attached to objects.

11. Mentioned Object Set
- Command parser output includes one primary object and a list of secondary objects directly referenced in the command.

## 5) Runtime Behavior Contract
### 5.1 Build Composite By Target
Preconditions:
1. Composite target is currently inactive.
2. Component part availability satisfies configured part requirement mode in required scope (default: player inventory).
3. Recipe is defined and valid.

Effects on success:
1. Each source component part is marked inactive (or removed from active gameplay set).
2. Composite target is marked active.
3. Ownership/location set for target (default: player inventory if parts were in inventory).
4. Success output line(s) emitted using producer-authored success echo text.
5. Optional linked actions execute as normal post-action behavior.

Failure modes:
1. Missing required parts.
2. Target already active.
3. Recipe invalid/misconfigured.
4. Scope mismatch for required parts.
5. Failure output line(s) emitted using producer-authored failure echo text.

### 5.2 Build Composite By Parts
Preconditions:
1. Parsed command yields a normalized part set (order-insensitive; quantity-aware if duplicates are enabled).
2. Exactly one eligible recipe matches using configured requirement mode (ExactPartSet, AllRequired, or MinimumCount) under scope policy.
3. Composite target for matched recipe is inactive.

Effects on success:
1. Matching recipe is resolved deterministically.
2. Required part objects are marked inactive.
3. Resolved composite target is marked active.
4. Success output line(s) emitted using producer-authored success echo text (including resolved target clarity when appropriate).

Failure modes:
1. No matching recipe for given part set.
2. More than one matching recipe and ambiguity policy is fail-with-hint.
3. Matched target already active.
4. Required parts not active/in-scope.
5. Failure output line(s) emitted using producer-authored failure echo text.

### 5.3 Break Composite Item
Preconditions:
1. Recipe is marked reversible.
2. Composite target is active and available in required scope.

Effects on success:
1. Composite target marked inactive.
2. Component parts restored by location-aware policy:
- If composite target is in inventory, restored parts go to inventory.
- If composite target is in room/world scope, restored parts go to that same room/world scope.
3. Success output line(s) emitted.

Failure modes:
1. Recipe not reversible.
2. Target inactive/not available.
3. Restoration conflict policy violation.

### 5.4 Command Resolution Contract (v1)
1. Routing remains primarily verb-driven.
2. Producers attach command actions to objects; for composites this may be the composite target object or one/more component part objects.
3. Command processing identifies all directly mentioned in-scope objects from input text.
4. Parser output contract for command context:
- `PrimaryObjectId` (single)
- `SecondaryObjectIds` (list)
5. Composite action resolution precedence:
- First, evaluate the primary object for a matching action for the parsed verb.
- If primary has a matching action, execute it immediately.
- If primary has no matching action, evaluate secondary objects in the exact order they were entered in the command.
- Execute the first matching secondary action found.
6. If no matching action is found on primary or any secondary, return normal not-understood/no-match diagnostics.
7. Command preprocessing is limited to object identification (primary + ordered secondary list); action choice occurs only in post-parse resolution.
8. Authoring uniqueness rule: a single scope object may have only one action for a given verb signature.
9. Directional/qualified variants are distinct signatures (example: look vs look east), so each may have one action on the same object.

## 6) Data Model Proposal
### 6.1 Recipe Definition
Add a composite recipe contract associated with target object:
1. RecipeId (Guid)
2. TargetObjectId (composite target)
3. RequiredParts (ordered entries; order is authoritative for consumption)
- Each entry includes: PartObjectId, RequiredQuantity (default 1), OptionalPart (default false for v1)
4. IsReversible (bool)
5. BuildVerbHints (optional list, example: fix, assemble, repair)
6. BuildSuccessMessage / BuildFailureMessage (optional)
7. BreakSuccessMessage / BreakFailureMessage (optional)
8. ScopePolicy (default inventory-only for v1)
9. PartRequirementMode (AllRequired or MinimumCount)
10. MinimumRequiredPartCount (required when PartRequirementMode is MinimumCount)

Deferred use case (post-v1): required-core plus optional-pool recipes:
1. Producers may mark some required parts as mandatory anchors (example: Engine must exist).
2. Producers may mark other parts as optional candidates toward a minimum threshold (example: Axles and Wheels contribute toward X).
3. Runtime rule target:
- All mandatory parts must be present.
- Optional candidate set must satisfy configured minimum count.
4. Existing `OptionalPart` contract field is reserved for this future behavior and remains non-functional in v1.

Ownership decision (v1):
1. Composite recipes are owned by target objects (no standalone recipe entities in v1).

Duplicate-part decision (v1):
1. Single row + quantity model is required.
2. Each `PartObjectId` may appear at most once in `RequiredParts`.
3. Repeated needs are expressed through `RequiredQuantity` on that single row.

### 6.2 Action Payload Additions
BuildCompositeByTarget action payload:
1. CompositeTargetObjectId
2. RecipeId or inline recipe reference
3. Optional strict part-count enforcement
4. SuccessEchoMessage (producer-authored)
5. FailureEchoMessage (producer-authored)

BuildCompositeByParts action payload:
1. RequiredPartObjectIds or recipe selector constraints
2. MatchMode (default ExactPartSet; supports AllRequired and MinimumCount when configured)
3. AmbiguityPolicy (default FailWithHint)
4. Optional resolved-target output template
5. SuccessEchoMessage (producer-authored)
6. FailureEchoMessage (producer-authored fallback)
7. IncompleteRecipeEchoMessage (optional; used when known recipe is partially satisfied)
8. SpecifyTargetEchoMessage (optional; used when provided parts could build multiple targets)
9. TargetMismatchEchoMessage (optional; used when requested target does not match provided parts)
10. NoMatchingRecipeEchoMessage (optional; used when no recipe is related to provided parts)

### 6.2.1 Echo Template Variable Contract (v1 proposal)
Template placeholders are runtime-populated tokens available in success/failure echoes.

Supported scalar tokens:
1. {requestedTarget}
2. {requestedTargetCount}
3. {missingPartsCount}
4. {candidateTargetsCount}
5. {providedPartsCount}

Supported list tokens:
1. {missingParts}
2. {candidateTargets}
3. {providedParts}

Supported indexed list access:
1. {missingParts[0]}
2. {missingParts[1]}
3. {candidateTargets[0]}
4. {providedParts[0]}

Optional list formatting helpers:
1. {missingParts|join:", "}
2. {candidateTargets|join:" or "}
3. {providedParts|join:", "}

Token behavior rules:
1. Token names are case-insensitive.
2. Indexing must be non-negative integer.
3. Out-of-range indexed tokens resolve to empty string.
4. Unknown tokens resolve to empty string in permissive mode (default) and may be surfaced in strict validation mode.
5. Literal braces may be escaped using double braces: {{ and }}.
6. Specialized failure echoes fall back to FailureEchoMessage when a specialized echo is blank.

BreakCompositeItem action payload:
1. CompositeTargetObjectId
2. RecipeId reference

Command processing context additions:
1. PrimaryObjectId
2. SecondaryObjectIds (ordered list as parsed)

### 6.3 Activation/State Integration
Use existing object activation state semantics:
1. Parts start active.
2. Composite target starts inactive.
3. Build toggles parts inactive + target active.
4. Consumption model decision (v1): parts are inactivated but remain in existing container/world lists.
5. Runtime participation is controlled by active state rather than list removal.
6. Break toggles target inactive + parts active.

### 6.4 Deterministic Consumption Policy (MinimumCount)
When `PartRequirementMode = MinimumCount` and more than X candidate parts are available:
1. Evaluate candidate parts in recipe order (top to bottom in `RequiredParts`).
2. Consume the first X eligible parts encountered.
3. Ignore remaining eligible parts for that build execution.
4. Use the same ordered policy for both target-driven and parts-driven build execution.

### 6.5 Quantifiable Object and Placement Model (v1)
Object definition additions:
1. IsQuantifiable (bool).
2. InventoryDisplayMode = AggregatedQuantity when IsQuantifiable is true.

World placement model:
1. One shared object definition may have multiple room placements.
2. Each placement stores its own `Quantity` value.
3. Example: Room A has Flowers quantity 3, Room B has Flowers quantity 2.
4. Each placement also stores a distribution mode:
- `GroupedStack`: one grouped placement entry with Quantity = X.
- `IndividualInstances`: X separate placement entries, each with Quantity = 1.
5. Placement entries include forward-compatible location metadata placeholder (optional in v1, populated later when in-room locations are introduced).

Inventory/runtime semantics:
1. Picking up quantifiable objects increments inventory quantity stack keyed by ObjectId.
2. Inventory UI shows one row with quantity (example: Flowers x5).
3. Composite consumption decrements quantity rather than toggling unique instances.
4. Quantity stack entry is removed when count reaches zero.

## 7) Command and Authoring UX Proposal
### 7.1 Known-intent Target Flow (v1)
Producer configures command phrase/action link such as:
1. Verb: fix
2. Qualifier: key
3. Linked action: BuildCompositeByTarget targeting full-key recipe
4. Action may be attached to composite target object or eligible component part object(s) per producer design.

Runtime then validates inventory parts and performs build transition.

### 7.2 Parts-driven Flow (v1)
Producer configures command phrase/action link such as:
1. Verb: rub or use
2. Qualifier pattern: cloth on lamp (or canonical multi-part form)
3. Linked action: BuildCompositeByParts with exact-part-set recipe matching
4. Actions may be attached to one or more referenced parts; runtime uses parsed primary + secondaries for deterministic eligibility.

Runtime resolves target from part set, then performs build transition.

### 7.3 Multi-part Command Semantics
Support these parser-facing forms:
1. Preposition form: "use cloth on lamp"
2. Join form: "combine objectA with objectB"
3. List form: "use objectA, objectB, objectC"

Canonicalization rules:
1. Normalize to object-id part set.
2. Ignore order by default.
3. Respect count when duplicate parts are explicitly required.
4. Parser captures one primary object and list of secondary mentioned objects for action routing.

### 7.4 Ambiguity Policy
For parts-driven matching:
1. Zero matches: fail with clear missing/invalid parts message.
2. One match: execute recipe.
3. Multiple matches: fail with deterministic ambiguity diagnostics (v1 default).
4. Optional future policy: explicit recipe priority tie-break.

### 7.5 Producer Experience Targets
1. Wizard-like recipe editor for selecting target and required parts.
2. Reversible toggle exposed clearly.
3. Validation warnings for impossible/ambiguous recipes.
4. Minimal required scripting for standard behavior.

### 7.6 Designer UI Workflow (v1 First)
Primary authoring flow:
1. Producer creates component parts as normal game objects.
2. Producer creates or edits the intended composite target object.
3. Producer enables "This object is a composite target" in object editor.
4. Producer selects required parts from a searchable list of inventory-able objects.
5. Producer configures build mode(s), reversibility, and success/failure echo messages.

Composite section layout (object editor):
1. Composite toggle: enable/disable composite behavior for current target object.
2. Required parts picker:
- Available list: inventory-able objects (search + filters).
- Selected list: required parts for this target recipe.
- Actions: add/remove/reorder part (reorder expected to be uncommon but supported).
3. Build behavior:
- Enable BuildCompositeByTarget.
- Optional enable BuildCompositeByParts.
- Part requirement mode: AllRequired or MinimumCount.
- Minimum required count (X) when MinimumCount selected.
- Scope policy and ambiguity policy fields.
4. Quantifiable support:
- Object-level toggle: quantifiable.
- Room placement editor supports per-placement quantity.
5. Echo messages:
- Build success echo (required).
- Build failure echo (required).
- Break success/failure echoes (required when reversible).
6. Reversible settings:
- IsReversible toggle.
- Break-specific policy fields when enabled.

Room tree authoring flow for quantifiable objects:
1. In room tree, right-click room game objects node.
2. Menu options:
- Add New Object (existing behavior).
- Add Existing Quantifiable Object (new; enabled only when at least one quantifiable object definition exists).
3. Add Existing Quantifiable Object dialog:
- Select quantifiable object from list.
- Enter quantity X.
- Choose placement mode:
	- Single grouped stack (one placement with quantity X).
	- Multiple individual instances (X placements, quantity 1 each).
4. Validation blocks confirm when X < 1.
5. If no quantifiable objects exist, menu item is disabled with hint text.
6. Reordering/placement fine-tuning remains future-facing until in-room location authoring is implemented.

Picker filtering and metadata:
1. Default list includes only inventory-able objects.
2. Optional show-all diagnostic view can be added later (not required for v1).
3. Each row should show object name and stable identifier to reduce selection mistakes.
4. Show live eligibility meter (for example: 3 of 5 configured parts currently required).
5. Quantifiable parts show quantity semantics indicator.

Inline validation UX:
1. Block self-reference (target cannot be one of its own required parts).
2. Block missing part references on save.
3. Warn on duplicate part selection unless duplicates are explicitly supported.
4. Warn when parts-driven matching can be ambiguous under current policy.
5. Block save when required success/failure echoes are empty for enabled build modes.
6. Block save when MinimumCount is selected and X is not within [1..configured candidate part count].
7. For quantifiable part requirements, required quantity must be >= 1.
8. For non-quantifiable parts, required quantity must remain 1 in v1.
9. Add Existing Quantifiable Object flow is restricted to quantifiable object definitions only.

## 8) Validation Rules
1. No recipe may reference missing object ids.
2. Target object cannot also be a required part in same recipe.
3. Required parts must be unique by `PartObjectId` (no duplicate rows).
4. Reversible recipes must define valid restore policy.
5. Build/break action payload must reference valid recipe/target.
6. Parts-driven recipes must define unambiguous part-set matching or accept explicit ambiguity policy.
7. Both build action types must define non-empty success and failure echo messages.
8. MinimumCount mode must define X and ensure 1 <= X <= number of configured candidate parts.
9. MinimumCount mode must not be combined with strict exact-set-only matching.
10. Quantifiable object placements must allow multiple room placements with independent quantities.
11. Quantifiable inventory aggregation must be deterministic by ObjectId.
12. Room-tree add-existing flow must only allow quantifiable object selection.
13. Distribution mode must be persisted (`GroupedStack` vs `IndividualInstances`) for each quantifiable room placement.
14. Echo template token syntax must validate successfully for all configured echo fields.
15. Indexed token expressions must use valid non-negative indexes.
16. Unknown tokens are allowed only when permissive mode is enabled.

## 9) Execution Phases
### Phase C1: Contracts and enums
1. Add new action types in shared enums/contracts.
2. Add composite recipe DTO/model contracts.
3. Add serialization support.
4. Add quantifiable object and placement quantity contracts.
5. Add placement distribution mode enum and serialized field.
6. Extend command parse contract to include `SecondaryObjectIds` list.

Exit criteria:
1. Build passes.
2. Contract tests added.

### Phase C2: Runtime execution core
1. Implement BuildCompositeByTarget execution path.
2. Implement BuildCompositeByParts execution path.
3. Implement BreakCompositeItem execution path.
4. Apply activation transitions and inventory/scope validation.
5. Add deterministic diagnostics for fail and ambiguity cases.
6. Implement requirement-mode evaluation (AllRequired vs MinimumCount).
7. Implement quantity-aware availability and consumption for quantifiable parts.
8. Implement verb-driven multi-object action resolution using primary + secondaries context.

Exit criteria:
1. Runtime unit/integration tests pass for success/failure branches.

### Phase C3: Designer authoring support
1. Add producer UI for recipe definition.
2. Add action editor fields for build/break actions.
3. Add validation feedback in authoring flow.
4. Implement composite target section in object editor with inventory-able part picker.
5. Require success/failure echo message fields for enabled build modes.
6. Add part-requirement mode controls and threshold validation.
7. Add quantifiable toggle and per-room placement quantity editing.
8. Add room-tree context menu command: Add Existing Quantifiable Object.
9. Add add-existing dialog with quantity and distribution mode controls.

Exit criteria:
1. Producer can configure example broken-key scenario without manual JSON edits.
2. Producer can complete parts-first workflow end-to-end without leaving object editor.

### Phase C4: Simulator and regression coverage
1. Add simulator fixture scenarios for build/break.
2. Add replay/regression tests with expected output and state assertions.

Exit criteria:
1. Focused regression suite passes including new composite scenarios.

### Phase C5: Optional discovery use-case design
1. Design-only phase for unknown-output crafting/discovery model.
2. Define command UX and ambiguity-resolution policy.

Exit criteria:
1. Approved follow-up plan document for discovery behavior.

## 10) Test Plan
### Unit tests
1. Build succeeds with complete part set and inactive target.
2. Build fails when any part missing.
3. Build fails when target already active.
4. Parts-driven build succeeds with exact matching part set.
5. Parts-driven build fails on zero match.
6. Parts-driven build fails on multi-match ambiguity (default policy).
7. Break succeeds for reversible recipe.
8. Break fails for non-reversible recipe.
9. Break fails when target inactive.
10. Build-by-target success path emits configured success echo.
11. Build-by-target failure paths emit configured failure echo.
12. Build-by-parts success path emits configured success echo.
13. Build-by-parts failure paths emit configured failure echo.
14. MinimumCount build succeeds when at least X candidate parts are available.
15. MinimumCount build fails when available part count is less than X.
16. Invalid MinimumCount configuration is rejected by validation.
17. Quantifiable part requirement succeeds when inventory stack meets required quantity.
18. Quantifiable part requirement fails when inventory stack is below required quantity.
19. Break restores parts to inventory when target was in inventory.
20. Break restores parts to room when target was in room scope.
21. Add-existing quantifiable with GroupedStack creates one placement entry with quantity X.
22. Add-existing quantifiable with IndividualInstances creates X placement entries with quantity 1.
23. Non-quantifiable objects are excluded from add-existing quantifiable selection.
24. Parser captures primary object plus all directly referenced secondary objects.
25. Verb-driven resolution executes action attached to target object when uniquely eligible.
26. Verb-driven resolution executes action attached to part object when uniquely eligible.
27. Authoring validation rejects duplicate action bindings for the same object and same verb signature.
28. Verb-driven resolution treats qualified verb signatures as distinct commands (example: look and look east).
29. Build-by-parts incomplete-recipe branch emits IncompleteRecipeEchoMessage when configured.
30. Build-by-parts ambiguity branch emits SpecifyTargetEchoMessage when configured.
31. Build-by-parts target-mismatch branch emits TargetMismatchEchoMessage when configured.
32. Build-by-parts no-related-recipe branch emits NoMatchingRecipeEchoMessage when configured.
33. Specialized failure echoes fall back to FailureEchoMessage when the specific field is not configured.
34. Indexed template tokens render expected list elements and return empty string for out-of-range indexes.
35. Join helper template tokens render deterministic list text.

### Integration tests
1. End-to-end command phrase "fix key" triggers build and state transitions.
2. End-to-end command phrase "rub cloth on lamp" resolves parts-driven target and transitions state.
3. Save/load roundtrip preserves recipe definitions.
4. Clean export/runtime bootstrap preserves required recipe fields.
5. Save/load roundtrip preserves PartRequirementMode and MinimumRequiredPartCount.
6. Save/load roundtrip preserves quantifiable flags and per-room placement quantities.
7. Save/load roundtrip preserves quantifiable placement distribution mode.

### Regression tests
1. Existing command/action paths remain unchanged.
2. Existing linked-action behavior remains stable.

## 11) Open Design Questions
None for v1 command resolution.

## 11.1 Approved Command Model Decisions (v1)
1. Command routing is verb-driven.
2. Composite-related actions are attached to objects chosen by producer (target object and/or part object(s)).
3. Parser contract is one primary plus a list of secondary mentioned objects.
4. Resolution is deterministic: primary first, then secondaries in command order.
5. Per-object action bindings are unique by verb signature; directional/qualified variants are treated as distinct signatures.

## 12) Risks and Mitigations
Risk:
1. Activation toggles can create inconsistent inventory/container state.
Mitigation:
1. Centralize transition logic and assert invariants post-transition.

Risk:
1. Command ambiguity when multiple recipes share same target name/verb.
Mitigation:
1. v1 requires explicit action link and deterministic command phrase routing.

Risk:
1. Parts-driven matching can create ambiguous multi-recipe results.
Mitigation:
1. Enforce exact part-set matching plus explicit ambiguity policy; default to fail-with-hint.

Risk:
1. MinimumCount can create non-deterministic part consumption if more than X parts are available.
Mitigation:
1. Recipe order is authoritative; consume first X eligible parts in list order.

Risk:
1. Authoring complexity for producers.
Mitigation:
1. Provide guided editor defaults and validation.

Risk:
1. Quantifiable stack behavior can drift from non-quantifiable object behavior.
Mitigation:
1. Keep quantity-stack transition logic explicit and covered by dedicated tests.

## 13) Definition of Done (v1)
1. Producers can author build/break composite behavior with UI support.
2. Runtime supports deterministic build and break transitions.
3. Activation semantics are clear and enforced.
4. Regression tests cover key success/failure and preserve existing behavior.
5. Documentation/terminology updated for composite vocabulary.
6. Both target-driven and parts-driven composite build paths are covered by tests.
7. Producer-authored success/failure echo messages are enforced and validated for both build paths.
8. Part requirement mode (AllRequired and MinimumCount) is supported, validated, and covered by regression tests.
9. Quantifiable object behavior (multi-room quantities, inventory stacks, quantity-aware composite consume/restore) is supported and covered by tests.

## 14) Suggested First Slice
1. Implement Phase C1 and C2 for a single happy-path recipe (broken key example).
2. Add focused tests before broad UI authoring work.
3. Validate state transitions and diagnostics thoroughly.
