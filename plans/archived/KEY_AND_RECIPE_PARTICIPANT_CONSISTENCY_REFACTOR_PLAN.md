# Key and Recipe Participant Consistency Refactor Plan

Last updated: 2026-07-20
Status: Complete for v1 participant parity; compatibility policy retained for legacy composite selection semantics
Question count: 21

## Goal

Refactor both lockable key definitions and composite recipe definitions to the same participant-first model already proven by procedures:

1. Define participants first.
2. Configure per-participant requirement settings.
3. Add optional per-participant variable value requirements.
4. Use the same authoring and runtime semantics across Keys and Recipes.

## Scope Intent

1. Keys and Recipes move to participant-centric contracts.
2. No legacy compatibility path is required for old key/recipe contract shapes.
3. Old sample content can be rebuilt directly in updated sample projects; no conversion utility is planned.

## Core Model Direction

### Shared Participant Requirement Shape

1. Participant identity: object id + display name.
2. Match settings per participant: match kind + match value.
3. Satisfaction settings per participant.
4. Quantity setting per participant.
5. Consumption setting per participant.

### Optional Per-Participant Variable Requirements

1. Attached to a specific participant.
2. Variable name.
3. Operator.
4. Expected value.
5. Validation performed only after participant candidate resolution.

### Operator Baseline for v1

1. Equals
2. NotEquals
3. GreaterThan
4. GreaterThanOrEqual
5. LessThan
6. LessThanOrEqual

## Runtime Direction

1. Resolve participants first using the same deterministic matching policy used by procedures.
2. Evaluate participant-level requirements.
3. Evaluate variable-level requirements for resolved participants.
4. Apply consumption only on full success.
5. Fail transactionally (all-or-nothing) with deterministic diagnostics.

## UX Direction

1. Keys editor and Recipe editor both follow the same three-panel flow:
2. Participant list
3. Selected participant settings
4. Selected participant variable requirements

## Breaking Change Policy

1. Explicitly remove old key/recipe runtime fallback logic.
2. Explicitly remove old key/recipe authoring shapes from Designer.
3. Enforce new contracts with load-time validation errors for unsupported legacy payloads.

## Decision Questions to Lock Off

1. Should Keys and Recipes share one exact participant DTO/type, or separate DTOs with a shared evaluator core?
2. Should participant identity be only object id, or object id plus stable participant id for future edits/reordering?
3. For MatchKind, do Keys and Recipes both support all procedure match kinds in v1, or a subset?
4. Should SatisfactionMode choices be identical between Keys and Recipes in v1?
5. Should ConsumptionPolicy choices be identical between Keys and Recipes in v1?
6. For quantity rules, can one object satisfy multiple participant slots when quantity permits, for both Keys and Recipes?
7. Are variable requirements AND-only in v1 (no OR groups) for both Keys and Recipes?
8. For Equals/NotEquals, should evaluation be bool-first, then numeric, then case-insensitive string?
9. For Greater/Less operators, should non-numeric comparisons fail with a hard diagnostic and requirement failure?
10. How should missing variables behave: always fail, or configurable per requirement?
11. Should empty string be treated as a valid expected value distinct from missing variable?
12. Should variable names be free text in v1, or constrained to discovered variables from the selected participant object?
13. Do variable requirements execute against the resolved participant instance only, or against all matching instances for quantity > 1?
14. For quantity > 1, is variable requirement policy AllResolvedMustPass or AnyResolvedCanPass?
15. Should Keys evaluate variable requirements before or after explicit key-token filtering from command text?
16. Should Recipes evaluate variable requirements before consumption planning or during final mutation transaction?
17. What diagnostic surface is mandatory for v1: result code only, result code + reason token, or detailed per-requirement traces?
18. Should Unlock and Lock actions share the same key requirement evaluator with operation-specific policies only?
19. Should composite Build-by-Target and Build-by-Parts both adopt the same participant+variable requirement model in the first recipe cut?
20. Which sample projects must be updated first to unblock CI/test suites after the breaking contract switch?
21. Do we want a one-time internal conversion utility for team use, even if runtime has no legacy compatibility?

## Locked Decisions

1. Keys and Recipes use separate DTOs and shared design patterns/evaluator semantics.
2. Participant identity uses object id only; no synthesized participant ids or names.
3. Keys and Recipes support the same MatchKind set as procedures in v1.
4. SatisfactionMode choices are identical between Keys and Recipes in v1.
5. ConsumptionPolicy choices are identical between Keys and Recipes in v1.
6. One object can satisfy multiple participant slots when quantity permits.
7. Variable requirements are AND-only in v1, with participant required/optional status and definition-level minimum optional satisfactions.
8. Equals and NotEquals evaluate in order: bool, numeric, then case-insensitive string.
9. Greater/Less support numeric-first comparison with lexical fallback; lexical fallback emits warning diagnostics, not hard errors.
10. Missing/null operand data evaluates comparisons to false with diagnostics; not a hard error.
11. Empty string is a valid value and is distinct from missing variable.
12. Variable names follow the same guided-list pattern as procedure editor based on selected participant.
13. Variable requirements evaluate against all resolved instances by default, with explicit aggregation modes for quantity scenarios.
14. Quantity-aware variable evaluation mode is explicit tri-mode: AllResolvedMustPass, AnyResolvedCanPass, SumOfResolvedCanPass.
15. Keys evaluate variable requirements after key-token filtering and participant resolution; unlock gets distinct variable-mismatch failure reason.
16. Recipes evaluate requirements pre-planning and recheck transactionally before mutation commit.
17. Mandatory diagnostics are result code plus reason token; detailed traces are optional via verbosity controls.
18. Unlock and Lock share one key evaluator with operation-specific policy inputs only.
19. Build-by-Target and Build-by-Parts adopt the same participant plus variable requirement model in first recipe cut.
20. Sample migration priority: BaseItemTesting, ObjectPlayLevel1, ObjectPlayLevel2, ObjectPlayLevel3, WorkshopTutorial, then Birmingham.
21. No one-time internal conversion utility; keep migration clean, direct, and legacy-free.

## Proposed Delivery Phases

1. Phase 1: Decision lock-off for questions above.
2. Phase 2: Shared contract/evaluator scaffolding in Shared (participant + variable requirement primitives).
3. Phase 3: Key definition refactor (designer model, serialization, runtime evaluation, diagnostics, tests).
4. Phase 4: Key-focused manual testing and review gate (authoring flow, unlock/lock outcomes, failure messaging, diagnostics review).
5. Phase 5: Key course-correction pass from manual review findings, then re-run validation gates.
6. Phase 6: Composite recipe refactor (designer model, serialization, runtime evaluation, tests) after key gate sign-off.
7. Phase 7: Cross-editor UX consistency pass (procedure/key/recipe flow alignment) and final diagnostics hardening.

## Implementation Audit (2026-07-20)

1. Phase 1: Done.
2. Phase 2: Done.
3. Phase 3: Done.
4. Phase 4: Done.
5. Phase 5: Done.
6. Phase 6: Done.
7. Phase 7: Done.

### Audit Notes

1. Key runtime parity is implemented (participant resolution, variable requirements, quantity evaluation modes, and dedicated variable-mismatch result codes).
2. Procedure runtime already provides the shared participant-first evaluator behavior baseline.
3. Composite designer authoring UX is participant-oriented and supports per-part variable requirements.
4. Composite clean export and project persistence now carry per-part variable requirements end-to-end.
5. Composite runtime payloads now include participant requirement descriptors and evaluate per-part variable requirements for selected parts during BuildCompositeByTarget and BuildCompositeByParts.
6. To preserve existing gameplay behavior, legacy part-selection/count semantics remain unchanged; participant variable requirements are enforced when declared.

## Closure Notes

1. Composite participant descriptors are now propagated in runtime payload contracts while preserving required-part-id compatibility for existing content.
2. Composite participant variable requirements now fail deterministically with diagnostic traces during build actions.
3. Focused regressions and runtime guardrail suites passed after implementation.

## Validation Gates

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\Storyboard.Shared.Tests\Storyboard.Shared.Tests.csproj
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
4. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|ArchitectureSeparationGuardrailsTests"
