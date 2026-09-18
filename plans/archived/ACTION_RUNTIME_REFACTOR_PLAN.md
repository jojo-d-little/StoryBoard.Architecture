# Action Runtime Refactor Plan

Status: Closed (Implemented and verified; R4 fallback retirement complete)
Owner: Storyboard.Shared runtime action execution pipeline
Last updated: 2026-07-05

Review note (2026-07-05):

1. Plan reviewed and closed after implementation summary and validation evidence confirmed complete.
2. Any future runtime action enhancements should be tracked in a new plan, not by reopening this one.

## 1. Purpose

Refactor runtime action data handling so action-type-specific fields are modeled with clearer ownership and safer execution pathways, reducing reliance on a single broad descriptor shape.

## 2. Why This Plan Exists

Current runtime action representation centers on a single broad descriptor model that contains fields for many unrelated action families.

Primary concerns:

1. Weak type ownership boundaries in runtime action data.
2. More conditional branching across action execution paths.
3. Higher change risk when adding or evolving action families.
4. Lower clarity between runtime contract shape and execution semantics.

## 3. Timing Decision

This work was executed as the planned follow-on runtime effort after designer payload stabilization.

Execution order completed:

1. Designer payload stabilization completed.
2. Runtime payload introduction and executor cutover completed.
3. Compatibility fallback retirement completed.

## 4. Goals

1. Introduce explicit runtime action-family data ownership.
2. Reduce broad blob-style field multiplexing in runtime execution paths.
3. Keep runtime behavior equivalent during staged migration.
4. Preserve deterministic command processing and linked-action behavior.
5. Stage migration with additive, compatibility-safe steps.

## 5. Non-Goals (Initial Runtime Slices)

1. No immediate rewrite of all runtime executor structure into one class per action type.
2. No unplanned breaking changes to external runtime contracts.
3. No simultaneous redesign of simulator UX or authoring UX.
4. No coupling of runtime implementation details back into designer project types.

## 6. Architecture Guardrails

1. Keep Storyboard.Shared free of WPF or designer-host concerns.
2. Keep Storyboard.Simulator dependent only on Storyboard.Shared runtime contracts.
3. Preserve existing runtime contract compatibility unless a deliberate versioning decision is made.
4. Prefer additive adapters/accessors before destructive shape removals.
5. Keep runtime-simulator scope focused on execution semantics; do not import designer-only validation complexity into Shared runtime contracts.
6. Enforce no compile-time dependency from Storyboard.Shared to StoryboardDesigner.App namespaces/types; treat any such dependency as a blocker.
7. Verify this boundary continuously with architecture guardrail tests in runtime validation gates.

## 7. High-Level Target Shape

Planned direction:

1. Retain stable runtime action envelope fields required for dispatch.
2. Introduce typed runtime payload families for action-type-specific data.
3. Centralize compatibility reads and action-type applicability rules.
4. Keep runtime execution using typed access pathways rather than wide descriptor reads.

Candidate payload families (subject to Slice 0 lock):

1. LinkedFlow runtime payload.
2. Synonym runtime payload.
3. Echo runtime payload.
4. Check/Set game property runtime payloads.
5. Container transfer runtime payload.
6. Navigation runtime payload.
7. Composite runtime payloads by action family.

## 7.1 Designer Payload Reference Review (Input Only, No Shared Coupling)

This plan may use StoryboardDesigner.App payload modeling as a reference for shape hints only.

Boundary rule:

1. No code sharing, type reuse, namespace dependency, or runtime contract ownership transfer from StoryboardDesigner.App into Storyboard.Shared.
2. Runtime contracts and accessors remain independently defined in Storyboard.Shared.

Runtime scope discipline:

1. Use designer payload structures as shape/reference hints for fields, names, and data types only.
2. Do not mirror designer-only concerns (authoring-time validation richness, UI-driven diagnostics metadata, schema-management affordances) unless runtime execution explicitly requires them.
3. Default to the smallest runtime payload surface that preserves behavior and deterministic execution.

Observed designer-side patterns worth mirroring conceptually:

1. Action-type-to-payload schema map with deterministic mapping entries.
2. Compatibility accessors that read typed payload first and fallback to legacy broad fields.
3. Explicit payload-family records for LinkedFlow, Synonym, Echo, ContainerTransfer, Navigate, CompositeByTarget, CompositeByParts, BreakComposite.
4. Optional capability facets per payload family (script fields, linked-actions projection, reference tokens) rather than one giant monolithic interface.

Implications for runtime Slice R0 ownership lock:

1. Define a runtime ownership matrix per action type using "Envelope fields" vs "Typed payload fields" vs "Derived/runtime-only values".
2. Include an explicit compatibility read-precedence rule in runtime accessors (Typed payload value vs legacy envelope field vs default).
3. Keep payload-family boundaries aligned with runtime execution semantics, not designer authoring concerns.

Known asymmetry to account for in runtime lock questions:

1. Designer currently maps both PutObjectInContainer and RemoveObjectFromContainer to the same container-transfer payload shape.
2. Designer schema currently leaves SetFlag without its own payload type entry.
3. Runtime must decide whether to preserve these asymmetries, normalize them, or introduce runtime-only refinements.

Additional lock-on questions added from this review:

1. Should runtime preserve a single shared container-transfer payload for both put/remove actions, or split payload identity while retaining shared fields?
2. Should runtime keep SetFlag in envelope-only compatibility mode, or introduce a typed runtime payload to eliminate null-payload special cases?
3. What is the runtime accessor precedence contract when typed and legacy fields disagree?
4. Which payload capability facets are required in runtime (for example linked-action projection, script-field enumeration, reference-token surface) and which are designer-only?

## 8. Migration Strategy (High Level)

### 8.1 Slice R0: Discovery and Contract Lock

1. Inventory all runtime action field reads and writes in Shared and Simulator.
2. Define runtime ownership matrix by action type.
3. Identify contract-facing data that must remain backward-compatible.
4. Define validation and regression gate list for runtime behavior equivalence.
5. Produce explicit lock decisions for the additional questions introduced by designer reference review (container-transfer symmetry, SetFlag payload stance, accessor precedence, capability facets).

Exit criteria:

1. Ownership matrix approved.
2. Contract compatibility constraints documented.
3. Baseline runtime regression suite identified.

### 8.2 Slice R1: Introduce Runtime Payload Contracts and Accessors

1. Add typed runtime payload models and accessors.
2. Keep descriptor shape operational via compatibility access adapters.
3. Add tests proving accessor parity with existing descriptor semantics.

Exit criteria:

1. No visible runtime behavior change.
2. Runtime-focused tests green.

### 8.3 Slice R2: Execution Path Cutover

1. Update runtime executor logic to consume typed accessors.
2. Keep compatibility fallback paths where needed during transition.
3. Confirm linked-action and ordering semantics remain unchanged.

Exit criteria:

1. Runtime execution parity maintained.
2. Focused runtime regression filters green.

### 8.4 Slice R3: Optional Executor Decomposition (Only If Justified)

1. Evaluate whether large executor methods should split by action family.
2. Decompose only where complexity/testability benefits are measurable.
3. Avoid churn-only class proliferation.

Exit criteria:

1. Measurable maintainability improvement, or this slice is skipped.

### 8.5 Slice R4: Bridge and Compatibility Retirement

1. Remove temporary runtime migration helpers no longer needed.
2. Keep only explicitly approved backward-read compatibility behavior.
3. Remove obsolete transitional tests and markers.

Exit criteria:

1. No temporary migration markers remain in production paths.
2. Full runtime validation gates green.

## 9. Validation Defaults

Primary validation commands to run during runtime slices:

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"

### 9.1 Session Replay Parity Pack (Mandatory Throughout R0-R4)

Run this pack continuously during the runtime refactor, not only at slice closeout.

Required cadence:

1. Run after any PR that touches runtime action payload contracts, compatibility accessors, or executor dispatch/processing behavior.
2. Run at each slice checkpoint (R0, R1, R2, R3 if used, R4).

Mandatory tests:

1. GameSimulatorPlaybackRegressionTests
2. GameCommandProcessorFixtureTests
3. SharedManagerHostFixtureTests

Primary command:

1. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameSimulatorPlaybackRegressionTests|GameCommandProcessorFixtureTests|SharedManagerHostFixtureTests"

## 10. Risks and Mitigations

1. Risk: Runtime contract drift.
Mitigation: Additive-first changes and explicit contract lock in Slice R0.

2. Risk: Behavioral regressions in command execution.
Mitigation: Focused runtime regression gates at each slice.

3. Risk: Over-refactor of executor structure.
Mitigation: Keep executor decomposition optional and evidence-driven.

## 11. Definition of Done

1. Runtime action data ownership is explicit and type-aligned.
2. Runtime execution behavior remains equivalent for supported fixtures.
3. Transitional runtime migration helpers are retired.
4. Architecture separation guardrails remain green.
5. Plan evidence logs commands, outcomes, and compatibility decisions.

## 12. Design Lock Questions (Concrete Walkthrough Set)

Use these questions in Slice R0 to produce explicit lock decisions before implementation.

For each question, record:

1. Decision (one sentence).
2. Owner.
3. Impacted code surfaces.
4. Required tests.
5. Status: Open / Locked / Verified.

### 12.1 Runtime Contract Boundaries

1. Which RuntimeCommandActionDescriptor fields are envelope-stable and must remain first-class dispatch fields?
2. Which fields move to typed payload ownership for each action family?
3. Which fields are runtime-derived/transient and should not be persisted as payload data?
4. Are any currently public/runtime-visible fields deprecated in place, and what is the deprecation/retirement timeline?

### 12.2 Payload Family Shape

1. Do PutObjectInContainer and RemoveObjectFromContainer share one runtime payload type or use two payload types with shared internal shape?
2. Does SetFlag remain envelope-only, or get a typed runtime payload for symmetry and accessor consistency?
3. For composite actions, do we keep three distinct payload families (ByTarget, ByParts, Break) or collapse common parts into shared sub-structures?
4. Which payloads require dedicated runtime reference-token surfaces vs no token surface at runtime?

### 12.3 Accessor and Compatibility Semantics

1. What is canonical accessor precedence when typed payload and legacy envelope fields disagree?
2. Is fallback direction always typed -> legacy -> default, or are there family-specific exceptions?
3. What defaulting policy applies for missing strings, GUIDs, bools, and collections per action family?
4. At what slice do we emit diagnostics (if any) for conflicting typed/legacy values?

### 12.4 Execution Parity and Behavior Guarantees

1. Which observable behaviors are non-negotiable parity guarantees (output lines, ordering, diagnostics, side effects)?
2. Which linked-action ordering semantics must remain bit-for-bit equivalent?
3. Which command preprocessor/executor interactions are considered compatibility-critical?
4. Which behavior differences (if any) are explicitly allowed and how will they be documented?

### 12.5 Scope Discipline (Avoid Over-Engineering)

1. Which designer-only payload concerns are explicitly out of runtime scope and must not be mirrored?
2. Which runtime capability facets are truly required (linked actions, script-field access, token metadata), and which are unnecessary?
3. What is the minimal runtime payload surface that preserves simulator behavior without authoring-centric complexity?
4. What review check prevents accidental introduction of designer-validation abstractions into Shared?

### 12.6 Migration and Retirement Controls

1. What exact conditions mark R1 complete (accessor parity evidence) before entering R2 cutover?
2. Which compatibility helpers are temporary, and what objective signal allows each helper to be removed in R4?
3. What rollback path exists if R2 cutover reveals parity regressions late?
4. What artifacts must be present to sign off the final retirement (diff summary, parity tests, command logs)?

### 12.7 Test and Evidence Requirements

1. Which existing runtime-focused tests are mandatory gates per slice (R0-R4)?
2. Which new tests are required specifically for accessor precedence conflicts and fallback rules?
3. Which fixtures cover container transfer and composite edge cases sufficiently, and where are gaps?
4. What command outputs and evidence logs must be captured for lock-off reviews?

### 12.8 Suggested Walkthrough Order

1. Contract boundaries.
2. Payload family shape.
3. Accessor precedence and defaults.
4. Execution parity guarantees.
5. Scope discipline and anti-overengineering checks.
6. Migration controls and test evidence requirements.

## 13. Locked Decision Log (Q20-Q28)

Status: Locked

### Q20. Anti-Bleed Runtime Boundary Guardrail

1. No compile-time dependency from Storyboard.Shared to StoryboardDesigner.App.
2. Any violation is a blocker.
3. Architecture separation guardrail tests are mandatory in runtime gates.

### Q21. R1 Completion Criteria Before R2

1. Typed payload contracts exist for all in-scope action families.
2. Accessors have locked precedence and defaults.
3. Accessor parity tests are green (typed, legacy, default cases).
4. Focused runtime regression gates are green.
5. Exception list is empty or explicitly deferred with signoff.

### Q22. Temporary Compatibility Helpers and Retirement

1. Temporary helpers include legacy field fallback readers, typed-vs-legacy conflict probes, and transition bridge shims.
2. Retire when all in-scope executor paths consume typed accessors, parity gates are green, and no unresolved compatibility exceptions remain.

### Q23. Compatibility Helpers Surviving Past R2

1. No helper survives by default.
2. Exception requires explicit reclassification as a supported runtime contract feature with dedicated tests and documentation.

### Q24. Mandatory Test Gate for Each New Typed Action Family

1. Contract mapping tests (raw payload to typed contract).
2. Accessor behavior tests (precedence, defaults, invalid data handling).
3. Executor parity tests (legacy vs typed outcomes).
4. Focused runtime regression gate.
5. Session replay parity pack, required throughout R0-R4 and after refactor-impacting PRs:
6. GameSimulatorPlaybackRegressionTests
7. GameCommandProcessorFixtureTests
8. SharedManagerHostFixtureTests

### Q25. R2 Cutover Rollback Policy

1. Immediately roll back affected family to compatibility accessor behavior on parity regression.
2. No partial forward merge unless replay/session parity pack is green.
3. Root-cause analysis and corrective regression tests are required before reattempt.

### Q26. Required Slice Sign-Off Artifacts

1. Timestamped command log of validation gates with pass/fail outcomes.
2. Replay/session parity pack results.
3. Scoped diff summary for runtime contract/accessor/executor changes.
4. Updated decision log entries for exceptions and defers.

### Q27. Required Reviewer Set

1. Runtime owner (author).
2. One Shared/runtime peer reviewer.
3. One boundary reviewer accountable for Shared vs Designer/Simulator separation guardrails.

### Q28. Final Go/No-Go Rule

1. Go only when all slice exit criteria are met, all mandatory gates are green (including replay/session parity pack), temporary compatibility helpers are retired or explicitly reclassified, and architecture guardrails are green.
2. Otherwise no-go, with unresolved items logged and owners assigned.

## 14. Implementation Summary (Completed)

1. Typed runtime payload families were introduced and used across action execution paths.
2. Composite runtime payloads were split into one payload type per action family (BuildCompositeByTarget, BuildCompositeByParts, BreakCompositeItem).
3. Container transfer payloads were split into one payload type per action family (PutObjectInContainer, RemoveObjectFromContainer).
4. Runtime executor paths were updated to consume typed accessors.
5. BreakCompositeItem behavior was aligned to child-sourced restoration semantics with quantifiable provenance support.
6. Runtime accessor legacy fallback reads were retired; accessors now use typed payload values or typed defaults.

## 15. Validation Evidence (Latest)

1. `dotnet build .\StoryboardDesigner.slnx` -> Passed.
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "RuntimeActionPayloadAccessorsTests|CompositeBuildActionTests|GameCommandProcessorFixtureTests|GameSimulatorPlaybackRegressionTests|SharedManagerHostFixtureTests|GameManagerTests|ArchitectureSeparationGuardrailsTests"` -> Passed (81 passed, 0 failed).

## 16. Current Follow-On Cleanup

1. Continue reducing legacy envelope-field surface in runtime contracts/mappers where behavior no longer depends on those fields.
2. Keep external/runtime contract safety and architecture guardrails unchanged during this cleanup.
