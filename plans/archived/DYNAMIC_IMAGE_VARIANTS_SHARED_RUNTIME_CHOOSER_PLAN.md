# Dynamic Image Variants And Chooser Plan

Status: Closed (Archived)
Owner: StoryboardDesigner.App authoring workflows + Storyboard.Shared runtime selection contract
Last updated: 2026-07-11

## Closeout Summary (2026-07-11)

1. Implementation slices A through F are complete.
2. Guided chooser script authoring now uses a proper editor workflow with variable assistance and variant-name insertion guidance (replacing raw text-only editing).
3. Build and automated test gates are green for closeout.
4. Plan is closed and moved to archived.

## 1. Purpose

Add support for entities to define multiple named images and choose the active image dynamically from game properties.

1. Support arbitrary named image variants per entity (not fixed slots).
2. Support chooser logic that resolves which variant name to use at runtime.
3. Keep room designer preview usable without runtime state by allowing manual variant selection.
4. Keep feature entity-agnostic (door open/closed is common example, not a special case).

## 2. Why This Plan Exists

Current image support is single-image oriented and cannot represent state-driven visuals.

Symptoms:

1. Producers cannot define stateful visual variants for one object.
2. Runtime cannot switch images based on object/room/game properties.
3. Room designer has no concept of selecting among multiple variants.
4. No validation exists for chooser output names versus available variant names.

## 3. Goals

1. Allow producers to define a named list of image variants for an entity.
2. Allow producers to author chooser logic using property-aware script editing.
3. Resolve chosen image variant at runtime/simulator from live property values.
4. Allow manual variant pick in room designer preview (no runtime property dependency).
5. Use a hard switch to named variant arrays for object imagery in this phase.
6. Preserve designer/runtime boundaries and clean export contract safety.

## 4. Non-Goals (Phase 1)

1. No bespoke door-only special behavior.
2. No automatic animation blending or transition effects.
3. No required simulation of property values in room designer.
4. No custom scripting language expansion beyond existing script grammar patterns.
5. No legacy single-image compatibility/migration path in phase 1.

## 5. Core Design Direction

### 5.1 Variant Data Model

1. Add an arbitrary list of named image variants on supported entities.
2. Each variant has at minimum:
   1. VariantName
   2. FullImagePath
   3. IsDefault (exactly one default when multiple variants exist)
3. Variant names are unique per entity using case-insensitive matching.
4. If exactly one variant exists, it is the de facto default.
5. Gray-map and normal-map variant fields are out of scope for this phase.

### 5.2 Chooser Script Model

1. Chooser script output is a variant name string.
2. Script authoring pattern mirrors existing echo script ergonomics.
3. Expected practical complexity:
   1. If/else-if/else chains
   2. Multi-variable checks in a condition
4. Chooser scope can read object plus parent chain properties: room, area, country, planet, global.
5. Runtime evaluates chooser script against live property context.
6. If chooser output is missing/invalid, fallback behavior is deterministic and validated.

### 5.3 Room Designer Behavior

1. Room designer does not evaluate chooser scripts in phase 1.
2. Each renderable room child object can be manually assigned a preview variant via dropdown.
3. Manual preview variant affects authoring preview only and does not rewrite chooser logic.
4. Include/exclude behavior continues to work independently of variant selection.
5. Manual preview variant selection is transient for the current editor session only.
6. Room designer opens with each object showing its default variant.

### 5.4 Runtime/Simulator Behavior

1. Runtime resolves active variant name per render pass or state change trigger.
2. Resolution uses chooser script when configured, otherwise default variant.
3. Resolution must be stable when properties are missing or script returns unknown values.
4. Runtime host remains independent from designer-only VM/UI concerns.

## 6. Lockoff Questions (Must Resolve Before Full Implementation)

1. DIV-01: Entity scope
   1. Phase 1 object-only, or objects plus room imagery elements?
2. DIV-02: Variant schema
   1. Exact DTO/model shape, required fields, and ordering guarantees.
3. DIV-03: Naming rules
   1. Case sensitivity, whitespace policy, duplicate handling.
4. DIV-04: Fallback policy
   1. Chooser output not found behavior (first variant, explicit default, or hidden).
5. DIV-05: Legacy compatibility
   1. Mapping from existing FullImagePath and related fields into variant set.
6. DIV-06: Chooser script location
   1. Stored per entity as script text, plus optional normalized representation.
7. DIV-07: Script grammar envelope
   1. Confirm allowed constructs for phase 1 and parser reuse approach.
8. DIV-08: Validation severity
   1. Warning vs error for unknown variant names, empty lists, malformed chooser.
9. DIV-09: Room designer preview state
   1. Where manual preview variant choice is persisted (project content vs sidecar UI state).
10. DIV-10: Export contract
   1. Phase 1 native persistence only or additive clean-export fields now.

## 6.1 Lock Decisions (Finalized 2026-07-11)

1. DIV-L01: Phase 1 scope is objects only.
2. DIV-L02: Variant schema is a named list with VariantName + FullImagePath only; gray/normal maps are excluded.
3. DIV-L03: One default image is supported; if only one variant exists it is the de facto default.
4. DIV-L04: Variant names are unique per object using case-insensitive matching; names are trimmed, non-empty, and display casing is preserved.
5. DIV-L05: Unknown chooser output fallback order is explicit default, then single variant, then first variant in list; validation warning is emitted.
6. DIV-L06: Hard switch from legacy single-image format to named variant array; no backward-compatibility migration path.
7. DIV-L07: Chooser script is stored per object as script text; empty chooser means use default image.
8. DIV-L08: Chooser script scope supports object + parent chain (room, area, country, planet, global).
9. DIV-L09: Chooser grammar is limited to if/else-if/else with multi-variable checks and variant-name output.
10. DIV-L10: Validation for phase 1 is warning-oriented (non-blocking) for unknown outputs, missing variants, missing default with multiple variants, and malformed chooser.
11. DIV-L11: Room designer manual preview variant selection is transient and non-persistent; default image is shown when designer opens.
12. DIV-L12: Phase 1 export contract is native authoring persistence only; Shared/Simulator clean-export enhancements are a follow-on plan.

## 7. Implementation Slices

### 7.0 Phase Grouping Summary

1. Total phases: 3
2. Total slices: 6
3. Phase 1 Authoring Foundations: Slice A, Slice B
4. Phase 2 Authoring Logic And Preview: Slice C, Slice D
5. Phase 3 Runtime And Hardening: Slice E, Slice F

### Slice A: Contracts And Persistence Foundation

Tasks:

1. Add image variant list model/DTO structures.
2. Add chooser script field(s) to supported entities.
3. Add deterministic load/save round-trip for named variant arrays only.
4. Remove single-image format dependency from object image authoring paths.

Exit criteria:

1. Named variant data round-trips deterministically.
2. Object image authoring paths use named variants only.

### Slice B: Variant Authoring UX

Tasks:

1. Build variant list editor in object property workflow.
2. Support add/remove/reorder/rename variants.
3. Add variant-name validation and inline guidance.
4. Keep edits within MVVM boundaries.

Exit criteria:

1. Producers can author named image sets safely.
2. Invalid duplicate names are blocked or normalized by policy.

## 7.1 Progress Update (2026-07-11)

1. Slice A status: Complete.
2. Slice B status: In progress.
3. Completed in this session:
   1. Object properties request/apply flows now round-trip and persist ImageVariants + ImageVariantChooserScript.
   2. Linked-instance edit policy now enforces definition ownership for image variant fields.
   3. Object property dialog now includes variant authoring controls for add/remove/rename/set-default.
   4. Existing appearance editor is now scoped to selected variant image path.
   5. Chooser script field is exposed in object property dialog and saved through existing request path.
   6. Variant editor now supports manual reordering (move up/down) to stabilize authored ordering.
   7. Chooser editor now shows inline warnings when referenced variant names are not present in the current variant list.
   8. Added regression tests covering variant normalization rules: single-default enforcement and case-insensitive duplicate-name collapse preserving first occurrence order.
4. Validation run after UI update:
   1. dotnet build .\StoryboardDesigner.slnx (pass; existing xUnit2031 warnings unchanged)
   2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~CompositePartOptionPopulationTests" (pass)
   3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep" (pass)
5. Remaining Slice B work:
   1. Add direct dialog-level tests for variant editor interaction events (move/reorder and chooser hint update behavior).

### Slice C: Chooser Authoring UX

Tasks:

1. Add chooser editor modeled after echo script authoring ergonomics.
2. Add property-aware token/variable picker integration.
3. Add variant-name aware suggestions and validation hints.
4. Provide script linting feedback in editor workflow.

Exit criteria:

1. Producers can author and save chooser logic.
2. Chooser references available variant names with guidance.

### Slice D: Room Designer Manual Variant Preview

Tasks:

1. Add per-room-child variant dropdown for preview selection.
2. Apply selected variant to rendered preview object imagery.
3. Keep include/exclude and drag behavior unchanged.
4. Keep preview variant selection session-local and non-persistent.

Exit criteria:

1. Producer can manually choose visible variant in room designer.
2. Selection updates preview immediately and safely.

### Slice E: Runtime And Simulator Resolution

Tasks:

1. Implement chooser evaluation pipeline for runtime image resolution.
2. Resolve variant by script output against available variants.
3. Apply deterministic fallback behavior using lock decision order.
4. Add simulator host integration tests for state-driven image switching.

Exit criteria:

1. Runtime image variant changes reflect live property state.
2. Missing/invalid outputs degrade safely and predictably.

### Slice F: Validation And Regression Hardening

Tasks:

1. Add rules for unknown chooser outputs, empty variants, malformed chooser script.
2. Add persistence, room-designer, chooser, and runtime regression tests.
3. Add architecture guardrail tests where boundary seams are touched.

Exit criteria:

1. Feature is covered by deterministic tests.
2. No boundary regressions introduced.

## 8. Test And Validation Gates

Between-slice gate:

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj

Per-slice minimum gate:

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "Image|Variant|Chooser|RoomDesigner|Validation|Serialization"
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"

Runtime-boundary gate (when Shared runtime selection contracts are touched):

1. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"

Manual signoff gate:

1. Author variants for an object (open/closed/damaged example) and verify persistence.
2. Author chooser script and verify validation guidance for invalid variant names.
3. Verify room designer manual variant dropdown updates preview as expected.
4. Verify runtime/simulator resolves variant from live property changes.

## 9. Initial Acceptance Checklist

1. Producer can define arbitrary named image variants on supported entities.
2. Producer can author chooser script that resolves to variant names.
3. Runtime uses chooser output to switch visible image variant.
4. Room designer supports manual variant selection independent of runtime property state.
5. Unknown chooser output follows deterministic fallback and emits validation issue.
6. Room designer preview variant selection is session-only and resets to default on reopen.
7. Serialization remains deterministic and does not churn unrelated JSON shape.
8. Hard switch behavior is documented and accepted for current project/test content.

## 10. Follow-Up Candidates

1. Optional simulated chooser evaluation in room designer using temporary property values.
2. Optional richer chooser diagnostics (coverage/unreachable branch hints).
3. Optional multi-entity variant support expansion (room pieces and overlays) after object-first stabilization.
