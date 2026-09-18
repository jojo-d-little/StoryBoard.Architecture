# Object Movement Restrictions Plan

Status: Locked for Implementation (2026-07-26)
Owner: StoryboardDesigner authoring + Storyboard.Shared runtime
Last updated: 2026-07-26

## Lock Summary (2026-07-26)

Plan lock state:

1. All initial lock-off questions are resolved.
2. Scope, data shape direction, runtime semantics, and UX labels are locked for first implementation cut.
3. The only explicit defer is save/restore persistence wiring for first/subsequent move tracking, to be implemented when save/restore game support is added.

Implementation readiness:

1. This plan is approved as implementation baseline.
2. Changes should proceed in phased slices from Section 9 with regression gates from Sections 10 and 12.

## 1. Purpose

Define and implement optional, per-object movement restrictions for movement actions while preserving existing real-world movement constraints and existing architecture boundaries.

This plan introduces a dedicated movement-restrictions sub-model on the object JSON model, plus a focused designer workflow for authoring restrictions without overcrowding the existing object appearance dialog.

## 2. Problem Statement

Current movement behavior uses real constraints only (board bounds, room bounds, support/stacking validity). If an object is movable, it can otherwise move in any direction and any distance.

We now need optional authored restrictions that can further limit legal movement by direction and max distance, with distinct rules based on:

1. First move vs subsequent moves.
2. Unstacked destination vs stacked destination.

This produces four movement-restriction categories:

1. First move + unstacked destination.
2. First move + stacked destination.
3. Subsequent move + unstacked destination.
4. Subsequent move + stacked destination.

## 3. Scope and Non-Goals

In scope:

1. Add object-level JSON sub-model for movement restrictions.
2. Keep the restrictions section optional/blank when not authored.
3. Support eight directions (`N`, `NE`, `E`, `SE`, `S`, `SW`, `W`, `NW`) per restriction category.
4. Support per-direction jump-over policy in the same restriction model.
5. Apply restrictions only to movement action evaluation.
6. Add designer authoring UX entry point from object appearance dialog to a dedicated movement restrictions dialog.
7. Add validation/tests for persistence, mapping, and runtime enforcement.

Out of scope for this plan:

1. Applying these restrictions to rotation.
2. Applying these restrictions to explicit stack action.
3. Changing baseline real constraints (bounds/support rules).
4. Replacing existing movable capability model.

## 4. Locked Baseline Semantics

1. Real constraints remain authoritative and always enforced.
2. Movement restrictions are additional constraints layered on top.
3. Accepted 2026-07-26: if a restriction section is absent/empty, behavior remains unchanged from current movement semantics.
4. Accepted 2026-07-26: `null` means no restriction rule for that direction/category.
5. Accepted 2026-07-26: `0` max distance means movement in that direction/category is explicitly disallowed.
6. Accepted 2026-07-26: restrictions include per-direction jump-over policy in addition to max distance.
7. Accepted 2026-07-26: first-move category is based on first successful move, not first attempt.
8. Accepted 2026-07-26: first/subsequent move tracking is part of game session state and should persist across save/restore when save-game support is implemented.
9. Accepted 2026-07-26: stacked vs unstacked category selection uses finalized runtime landing state (destination outcome), not initial intent.
10. Accepted 2026-07-26: if the applicable finalized-landing category rule disallows the move (distance cap or jump-over rule), the entire move fails with no partial success.
11. Accepted 2026-07-26: movement-restriction failures use one shared result code ("object movement restriction"); specific cause details are diagnostics/logging only.
12. Accepted 2026-07-26: authoring supports category-level bulk copy from one category to one or more target categories.
13. Accepted 2026-07-26: authoring/runtime model supports same-as flags so first-move can reuse subsequent rules and stacked-landing can reuse unstacked rules.
14. Accepted 2026-07-26: authoring supports per-category "fill all directions" to apply one value set across all eight directions.
15. Accepted 2026-07-26: diagonal directions are independently authored; any auto-fill behavior is editor convenience only and never implicit runtime derivation.
16. Accepted 2026-07-26: `maxDistance` valid range is 0-99; values above 99 are invalid.
17. Accepted 2026-07-26: `maxDistance` is integer-only at authoring and runtime; non-integer values are invalid.
18. Restrictions are evaluated only for movement actions.
19. Rotation and explicit stack action ignore this restriction model in v1.
20. Clarification 2026-07-26: movable is an authored object feature toggle in designer data; runtime does not author/edit movement restriction definitions.
21. Accepted 2026-07-26: when movable is off, authored restriction data is preserved but restriction editing controls are disabled in designer UX.
22. Accepted 2026-07-26: in linked/base scenarios, movement restrictions are definition-owned and linked instances do not support local overrides in v1.
23. Accepted 2026-07-26: clean export includes `movementRestrictions` using aligned field names and omits the section when no rules are authored.
24. Accepted 2026-07-26: no dedicated preview/effective-rule panel in v1; rely on matrix visibility plus same-as inherited read-only display.
25. Accepted 2026-07-26: explicit stack action remains exempt from movement restrictions; stack action behavior is governed by stack-specific rules.
26. Accepted 2026-07-26: rotation remains exempt from `movementRestrictions`; any future rotation limits require a separate explicit feature.
27. Accepted 2026-07-26: UX category labels are locked as:
1. First Move - Open Landing
2. First Move - Stacked Landing
3. Later Moves - Open Landing
4. Later Moves - Stacked Landing

Deferred implementation note:

1. Until save/restore game is implemented, first/subsequent tracking remains runtime-session state.
2. Persisted save payload support for this state is explicitly planned for the future save/restore effort.

## 5. Data Model Direction (JSON)

### 5.1 Structure Goal

Model movement restrictions as a separate object sub-document on each game object, analogous to image variance and appearance subsections.

If no restrictions are authored, this section is omitted (or persisted as empty only if required by serializer policy).

The model must also support low-friction rule reuse through same-as flags to reduce repetitive data entry.

### 5.2 Working Shape (Draft)

```json
{
  "movementRestrictions": {
    "sameAs": {
      "firstUsesSubsequent": false,
      "stackedUsesUnstacked": false
    },
    "firstUnstacked": {
      "N": { "maxDistance": null, "allowJumpOver": null },
      "NE": { "maxDistance": null, "allowJumpOver": null },
      "E": { "maxDistance": null, "allowJumpOver": null },
      "SE": { "maxDistance": null, "allowJumpOver": null },
      "S": { "maxDistance": null, "allowJumpOver": null },
      "SW": { "maxDistance": null, "allowJumpOver": null },
      "W": { "maxDistance": null, "allowJumpOver": null },
      "NW": { "maxDistance": null, "allowJumpOver": null }
    },
    "firstStacked": {
      "N": { "maxDistance": null, "allowJumpOver": null },
      "NE": { "maxDistance": null, "allowJumpOver": null },
      "E": { "maxDistance": null, "allowJumpOver": null },
      "SE": { "maxDistance": null, "allowJumpOver": null },
      "S": { "maxDistance": null, "allowJumpOver": null },
      "SW": { "maxDistance": null, "allowJumpOver": null },
      "W": { "maxDistance": null, "allowJumpOver": null },
      "NW": { "maxDistance": null, "allowJumpOver": null }
    },
    "subsequentUnstacked": {
      "N": { "maxDistance": null, "allowJumpOver": null },
      "NE": { "maxDistance": null, "allowJumpOver": null },
      "E": { "maxDistance": null, "allowJumpOver": null },
      "SE": { "maxDistance": null, "allowJumpOver": null },
      "S": { "maxDistance": null, "allowJumpOver": null },
      "SW": { "maxDistance": null, "allowJumpOver": null },
      "W": { "maxDistance": null, "allowJumpOver": null },
      "NW": { "maxDistance": null, "allowJumpOver": null }
    },
    "subsequentStacked": {
      "N": { "maxDistance": null, "allowJumpOver": null },
      "NE": { "maxDistance": null, "allowJumpOver": null },
      "E": { "maxDistance": null, "allowJumpOver": null },
      "SE": { "maxDistance": null, "allowJumpOver": null },
      "S": { "maxDistance": null, "allowJumpOver": null },
      "SW": { "maxDistance": null, "allowJumpOver": null },
      "W": { "maxDistance": null, "allowJumpOver": null },
      "NW": { "maxDistance": null, "allowJumpOver": null }
    }
  }
}
```

Interpretation (draft):

1. `maxDistance` is max allowed distance in squares for that direction under that category.
2. `maxDistance: null` means no max-distance rule for that direction/category.
3. `maxDistance: 0` explicitly disallows movement in that direction/category.
4. `allowJumpOver` controls whether obstacle-overjump is allowed for that direction/category.
5. `allowJumpOver: null` means no authored jump-over rule for that direction/category.
6. Category absence means unrestricted for that category.
7. `sameAs.firstUsesSubsequent=true` means first-move categories resolve from subsequent categories.
8. `sameAs.stackedUsesUnstacked=true` means stacked-landing categories resolve from unstacked categories.
9. If both same-as flags are true, only one effective matrix needs to be authored.
10. `maxDistance` numeric values must be integers in the 0-99 range.

## 6. Runtime Evaluation Direction

For each movement action attempt, runtime computes evaluation context:

1. Is this the object's first successful move in current runtime session?
2. Does destination result in stacked landing or unstacked landing?
3. What is intended movement direction?
4. What is intended distance?

Then runtime resolves the applicable category and direction cap:

1. If no cap is authored, distance restriction check passes.
2. If cap exists, requested distance must be `<= cap`.
3. If no jump-over rule is authored, jump-over handling uses existing movement behavior.
4. If jump-over rule is authored, it applies for that direction/category.
5. Real constraints still run and can independently fail movement.
6. Category selection uses finalized landing state.
7. If the applicable finalized category rule disallows the move, the full move fails.

Recommended ordering:

1. Evaluate restriction-cap check early when enough context is known.
2. Keep existing real-constraint checks deterministic and unchanged.
3. Emit one shared movement-restriction result code on restriction failure and include specific cause details in diagnostics.

## 7. Designer UX Direction

### 7.1 Entry Point

Keep basic movable controls in object appearance dialog.

Add a dedicated button, for example: `Edit Movement Restrictions...`

Clicking opens a purpose-built movement restrictions dialog.

### 7.2 Dialog Responsibilities

1. Present all four categories in a compact but readable layout.
2. Allow per-direction max-distance input for each category.
3. Allow quick clear/reset actions (per cell, per category, full section).
4. Clearly indicate that blank means unrestricted.
5. Provide directional labels that reduce data-entry mistakes.
6. Provide same-as toggles:
7. First move same as subsequent.
8. Stacked landing same as unstacked landing.
9. When same-as is on, dependent sections are visually read-only and show inherited values.
10. Provide per-category "Fill All Directions" action for fast matrix entry.

### 7.3 Usability Guardrails

1. Prioritize fast repetitive entry (keyboard-friendly navigation).
2. Avoid crowding appearance dialog with the full matrix.
3. Keep defaults blank to preserve current behavior.
4. Include concise in-dialog legend for first/subsequent and stacked/unstacked semantics.

## 8. Persistence and Contract Expectations

1. Native project JSON stores authored movement restrictions in object sub-document.
2. Accepted 2026-07-26: clean export carries `movementRestrictions` in object payload with aligned field names.
3. Clean export behavior should be additive and non-breaking (schema additions only).
4. Accepted 2026-07-26: omit `movementRestrictions` entirely when no rules are authored.
5. Preserve deterministic property ordering in serialized output.

## 9. Implementation Phases (Initial)

### Phase 1 - Contracts and Model

1. Add shared model contracts for movement restriction sub-document and category/direction entries.
2. Add JSON serialization mapping that omits the section when empty.
3. Add baseline unit tests for round-trip persistence.

### Phase 2 - Runtime Enforcement

1. Add runtime resolver for restriction category based on first/subsequent and stacked/unstacked landing context.
2. Add direction-cap evaluation and movement failure result code.
3. Add runtime tests for all four categories and eight-direction coverage.

### Phase 3 - Designer Authoring UX

1. Add movement restrictions button to object appearance dialog.
2. Implement dedicated movement restrictions dialog and view model.
3. Add validation and save/cancel behavior.
4. Add designer tests for authoring workflow and persistence.

### Phase 4 - Regression and Hardening

1. Validate no behavior changes for objects with no restriction payload.
2. Validate restrictions do not affect rotation and explicit stack action.
3. Run focused runtime guardrail tests and playback regression gates.

## 10. Test Strategy (Initial)

1. JSON round-trip tests for blank, partial, and full restriction payloads.
2. Runtime movement tests for each category with positive and negative direction-cap cases.
3. Cross-check tests proving real constraints still fail correctly regardless of restriction payload.
4. Regression tests proving unrestricted objects remain behaviorally unchanged.
5. Designer workflow tests for launching dialog, editing matrix, persisting, and clearing.

## 11. Design Questions To Lock Off (Remaining)

All lock-off questions in this initial plan set are resolved.

## 12. Exit Criteria For First Implementation Cut

1. Objects with no restrictions behave exactly as before.
2. Objects with authored restrictions enforce directional max-distance by applicable category.
3. Authoring flow is usable from appearance dialog via dedicated restrictions dialog.
4. Rotation and explicit stack action remain unaffected.
5. Build and required regression gates pass.
