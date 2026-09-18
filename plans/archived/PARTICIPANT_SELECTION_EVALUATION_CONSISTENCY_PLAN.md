# Participant Selection and Evaluation Consistency Plan

Last updated: 2026-07-20
Status: Decision lock complete; implementation-ready
Question count: 14

## Goal

Bring Keys, Composite Recipes, and Procedures to one consistent participant model for:

1. Participant definition and authoring defaults.
2. Participant resolution and quantity allocation.
3. Participant variable requirement evaluation.
4. Diagnostics and result signaling.

## Scope

1. In scope: participant selection and evaluation semantics for lock/unlock, composite build-by-parts/build-by-target, and procedure invoke.
2. In scope: designer authoring parity for participant fields and variable requirement editing.
3. Out of scope: unrelated action types and non-participant navigation/gameplay rules.

## Delivery Phases

1. Phase 1: Consistency baseline audit.
- Produce a side-by-side matrix for Keys, Composite, Procedures across defaults, match-kind behavior, satisfaction modes, quantity semantics, optional handling, variable evaluation timing, and diagnostics.
- Freeze current behavior in focused tests before refactor.

2. Phase 2: Canonical participant contract.
- Define one canonical participant semantics contract in Shared (with feature-specific adapters as needed).
- Normalize naming differences such as OptionalPart vs IsOptional and Quantity vs RequiredQuantity through adapter boundaries.

3. Phase 3: Canonical evaluator extraction.
- Extract shared runtime evaluator components for:
- candidate discovery and filtering,
- quantity allocation,
- variable requirement evaluation (all/any/sum),
- deterministic diagnostic fragments.
- Keep action-specific result-code mapping and mutation/consumption behavior separate.

4. Phase 4: Feature integration pass.
- Wire lock/unlock, composite builds, and procedure invoke to the canonical evaluator path.
- Remove duplicated evaluator branches where behavior is intended to be identical.

5. Phase 5: Designer authoring consistency pass.
- Align participant editor behavior (defaults, editable fields, coercions, match-kind handling) across the three editors.
- Introduce shared participant editor control only after behavior parity is proven.

6. Phase 6: Regression and rollout.
- Add parity tests that run identical participant scenarios through all three features where applicable.
- Run full build and focused regression suites.
- Update docs and plans with locked behavior decisions.

## Implementation Phase Plan

1. Phase A: Contract and naming unification.
- Deliverables:
- Introduce canonical participant DTO/descriptor naming across designer models, clean export DTOs, and runtime descriptors.
- Add adapter shims only where needed for in-flight internal call sites during migration.
- Exit criteria:
- Keys, composites, and procedures reference the same participant field names and value semantics end-to-end.

2. Phase B: Runtime canonical evaluator.
- Deliverables:
- Extract shared evaluator components for candidate filtering, satisfaction-first resolution, deterministic ordering, quantity allocation, and variable requirement evaluation.
- Integrate lock/unlock, composite build-by-parts/build-by-target, and procedure invoke onto shared evaluator path.
- Add dedicated composite participant-variable failure result codes and registry mapping.
- Exit criteria:
- All three features resolve/evaluate participants through one canonical evaluator pipeline.

3. Phase C: Designer authoring behavior parity.
- Deliverables:
- Align participant editor defaults and behavior (match-kind editing, conversion rules, optional handling, quantity behavior) across all three authoring workflows.
- Remove forced ObjectId coercion in lock/composite editor logic.
- Exit criteria:
- Equivalent participant definitions can be authored the same way in procedure, key, and composite editors.

4. Phase D: Shared participant editor UI.
- Deliverables:
- Build reusable participant editor control and shared participant editor viewmodel.
- Host with thin wrappers for procedure, key, and composite contexts.
- Keep host-specific labels/help text and action-specific hints in wrappers.
- Exit criteria:
- Shared control is used by all three editors with no behavior drift.

5. Phase E: Export and import consistency.
- Deliverables:
- Ensure unified participant definitions serialize and deserialize identically for project persistence and clean export.
- Apply explicit contract version bump and migration notes for clean export breaking changes.
- Exit criteria:
- Roundtrip and clean export tests pass using unified participant model.

6. Phase F: Parity regression and rollout hardening.
- Deliverables:
- Add parity matrix tests that run equivalent participant scenarios across lock/composite/procedure.
- Add deterministic diagnostics assertions for base reason token and verbosity-gated detail.
- Run full validation gates and runtime-focused regression suites.
- Exit criteria:
- All validation gates pass and parity tests confirm no semantic drift.

## Validation Gates

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\Storyboard.Shared.Tests\Storyboard.Shared.Tests.csproj
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|ArchitectureSeparationGuardrailsTests|CompositeBuildActionTests"

## Design Questions to Lock

1. Should default SatisfactionMode be identical for all three features, and if so which default (ExplicitMentionRequired or PossessionRequired)?
2. Should MatchKind support be identical in authoring and runtime for all three features, including ObjectType flows?
3. Should lock/composite editors continue forcing ObjectId internally, or allow full match-kind semantics exactly like procedures?
4. Should participant matching always include satisfaction filtering before quantity allocation in all three features?
5. Should composite continue preserving legacy selected-part behavior, or migrate fully to canonical participant resolution?
6. Should optional participant semantics be identical (including minimum optional thresholds), and if yes do composites need AnyNOfM-style thresholds?
7. Should variable requirements be evaluated only for resolved participants in all cases, including composite minimum-count modes?
8. Should participant variable failures map to dedicated result codes for composite actions, matching lock/procedure specificity?
9. Should lexical fallback for non-numeric comparisons remain allowed everywhere, or be tightened to numeric-only for numeric operators?
10. Should missing variables always evaluate to false with diagnostics across all three, or be configurable per requirement?
11. Should participant DTO naming be unified internally now, while preserving external clean contract compatibility adapters?
12. Should participant ordering and tie-break rules be identical and explicitly deterministic across all evaluators?
13. Should diagnostics verbosity policy be standardized (base reason token always, detailed traces behind verbosity setting)?
14. Should shared UI be one reusable participant editor control with host-specific wrappers, or shared viewmodel logic with separate views?

## Locked Decisions

1. Defaults are identical across all three features, and the producer can easily change the choice in authoring UI.
2. MatchKind support is consistent across all three in authoring and runtime.
3. Lock and composite no longer force ObjectId internally; full MatchKind semantics are supported consistently.
4. Satisfaction filtering runs before quantity allocation in all three features.
5. Composite moves to canonical participant resolution now.
6. Optional semantics are identical across all three, including minimum optional thresholds.
7. Variable requirements are evaluated only against the resolved participant set in all three features.
8. Composite actions get dedicated participant-variable failure result codes.
9. Numeric operators remain numeric-first with lexical fallback and warning diagnostics, consistently across all three.
10. Missing variables evaluate to false with diagnostics across all three.
11. Participant naming and shape are unified end-to-end across designer UI, export, and runtime.
12. Ordering and tie-break rules are deterministic and identical across all evaluators.
13. Diagnostics policy is standardized: base token always present; detailed traces gated by verbosity.
14. Shared UI direction is a reusable participant editor control with host-specific wrappers.

## Completion Criteria

1. Same participant input scenario yields equivalent selection/evaluation decisions across keys, composites, and procedures where action policy is intended to match.
2. Any intentional differences are explicitly documented as policy-level exceptions.
3. Shared evaluator components are used by all three features for the agreed common semantics.
4. Shared participant authoring behavior is consistent before introducing visual-level control reuse.
