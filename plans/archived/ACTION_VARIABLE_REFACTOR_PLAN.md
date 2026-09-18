# Action Variable Refactor Plan

Status: Closed (Current Scope, 2026-07-08)
Owner: Storyboard.Shared runtime action execution + StoryboardDesigner.App action authoring UX
Last updated: 2026-07-08

## 1. Purpose

Consolidate action variable (action.*) support into a single runtime-owned source of truth in Storyboard.Shared, with designer consuming that metadata for design-time token discovery, validation, and authoring UX.

## 2. Why This Plan Exists

Current action variable support is spread across multiple layers and duplicate token lists can drift.

Symptoms:

1. Designer can permit a token that runtime does not populate for all outcome paths.
2. Action-specific token lists exist in multiple places (payload model hints, token providers, runtime resolver keys).
3. Token discoverability and runtime population behavior are not guaranteed to remain aligned.
4. Regression fixes are often reactive (found by playback/runtime tests) rather than prevented by a single contract.

## 3. Goals

1. Make runtime the canonical authority for supported action.* variables per action type.
2. Ensure designer token suggestions/validation are consumed from Shared runtime metadata.
3. Remove duplicate token lists from designer-side payload/provider definitions where possible.
4. Add guardrails to detect token declaration vs runtime-population drift.
5. Publish and approve a durable developer cookbook for adding/extending action variables end-to-end.

## 4. Non-Goals (Initial Refactor)

1. No large rewrite of action execution architecture.
2. No breaking changes to existing outcome message token syntax (`action.*`) unless explicitly approved.
3. No host boundary violation (Shared must remain independent from designer/simulator).
4. No forced expansion of all action types in one PR; phased migration is preferred.

## 5. Core Design Direction

## 5.1 Canonical Runtime Contract

Introduce (or adapt to) a runtime metadata contract in Shared for action variables, keyed by action type.

Required shape per action type:

1. Supported action variable tokens (full `action.*` names).
2. Token categories (list/scalar) and index behavior notes where applicable.
3. Optional outcome-path applicability metadata:
   1. FailureOnly
   2. SuccessOnly
   3. Both
4. Optional token description text for designer UX hints.

## 5.2 Designer Consumption Model

Designer retrieves supported action variable metadata from Shared APIs (no duplicated hardcoded token lists).

Designer usage points:

1. Echo editor token lists.
2. Quick token filters.
3. Script unknown reference validation token set assembly.
4. Optional metadata display (description/example).

## 5.3 Drift Guardrails

1. Runtime tests assert each emitted action variable token is declared by canonical metadata.
2. Designer tests assert designer-visible token lists exactly match Shared metadata for each action type.
3. Script validation tests assert accepted/rejected token behavior is driven by canonical source.
4. Architecture guardrails prevent introducing new designer-only token registries disconnected from Shared.

## 6. Success Criteria

Plan is complete when all are true:

1. Shared runtime is the single place to maintain action variable support metadata per action type.
2. Designer token providers/registries no longer maintain independent source-of-truth token lists.
3. Existing token-dependent behavior remains green (build, focused runtime, replay, full suite).
4. Guardrail tests exist for runtime/designer token parity and runtime-population alignment.
5. Cookbook is updated and approved:
   1. `ACTION_VARIABLE_COOKBOOK.md` updated with final architecture and workflow.
   2. Cookbook reviewed and marked approved in this plan execution log.

## 7. Scope

In scope:

1. Shared runtime action variable metadata contracts and registration.
2. Runtime action executables/resolvers alignment to canonical metadata.
3. Designer token discovery and script validation consumption from Shared metadata.
4. Tests and guardrails for parity and emission correctness.
5. Cookbook update and approval tracking.

Out of scope:

1. Broad script language redesign.
2. Non-action variable systems unrelated to `action.*` token flow.
3. Contract versioning changes not required by this refactor.

## 8. Architecture Boundaries

1. Shared owns canonical action variable metadata and runtime behavior.
2. Designer consumes Shared metadata; no reverse dependency from Shared to Designer.
3. Simulator remains a consumer of Shared runtime only.

## 9. Implementation Slices

### Slice R0: Lockoff and Inventory Baseline

Goal: inventory current token declarations and runtime population paths before code movement.

Tasks:

1. Produce an inventory map for each action type:
   1. designer-declared tokens,
   2. runtime-populated tokens,
   3. mismatch notes.
2. Lock canonical metadata model shape in this plan.
3. Capture migration strategy for BuildCompositeByParts as pilot.

Exit criteria:

1. Inventory baseline checked in (plan notes and/or companion markdown).
2. Canonical contract shape approved.
3. Full validation cycle completed and recorded for this slice.

### Slice R1: Shared Canonical Metadata Foundation

Goal: introduce canonical action variable metadata in Storyboard.Shared.

Tasks:

1. Add Shared registry/descriptor for action variable tokens keyed by `CommandActionType`.
2. Execute early pilot first: BuildCompositeByParts as the first shared-backed source-of-truth migration.
   1. Define canonical BuildCompositeByParts action-variable metadata in Shared.
   2. Keep designer-side interface shape unchanged during pilot.
   3. Use a thin designer facade adapter backed by Shared metadata (no local token literals for migrated pilot action).
3. Register baseline metadata for next pilot action type(s) after BuildCompositeByParts:
   1. BreakCompositeItem,
   2. container transfer actions (if low-risk).
4. Add Shared API accessor(s) for designer/runtime consumers.

Exit criteria:

1. Canonical metadata available in Shared for pilot actions.
2. BuildCompositeByParts designer token surface is served by Shared-backed adapter with no duplicate local token list.
3. Tests verify metadata shape and token stability.
4. Full validation cycle completed and recorded for this slice.

### Slice R2: Designer Consumption Migration

Goal: switch designer token sourcing to Shared metadata.

Tasks:

1. Replace/bridge `ActionEchoReferenceTokenProviderRegistry` consumption to Shared metadata-backed source.
   1. Land BuildCompositeByParts bridge first as the reference implementation.
2. Update `RoomActionEditorDialog` token retrieval path to canonical source.
3. Update script validation token assembly (`ScriptUnknownReferenceRule`) to canonical source.
4. Remove redundant token declarations for migrated actions where safe.

Exit criteria:

1. Designer token suggestion and validation for migrated actions driven by Shared metadata.
2. Parity tests prove no drift.
3. Full validation cycle completed and recorded for this slice.

### Slice R3: Runtime Population Alignment and Coverage

Goal: ensure runtime population behavior matches declared metadata (success/failure path awareness included).

Tasks:

1. Add/extend runtime tests for declared token population per result path.
2. Add guardrail test: emitted tokens must be declared.
3. Ensure BuildCompositeByParts success/failure context parity remains covered.

Exit criteria:

1. Runtime declared vs emitted token alignment tests pass.
2. No unknown-reference regressions in simulator/playback flows.
3. Full validation cycle completed and recorded for this slice.

### Slice R4: Cleanup and Consolidation

Goal: remove legacy duplicate token declarations and finalize architecture docs.

Status: Complete (2026-07-08).

Tasks:

1. Remove deprecated duplicate token lists from designer-side payload/provider classes (for migrated actions).
2. Keep compatibility adapters only where intentionally required.
3. Update cookbook to final model and include migration steps/checklists.
4. Record cookbook approval in plan log.

Exit criteria:

1. Single-source maintenance achieved for migrated action variables.
2. Cookbook updated and approved.
3. Full validation cycle completed and recorded for this slice.

### Slice R5 (Later Phase): action.all Debug Dump

Goal: add a runtime-owned debug variable `action.all` as an optional per-action token pattern that emits a human-readable dump of supported action variables and current values for the executing action.

Tasks:

1. Define canonical Shared metadata semantics for `action.all`:
   1. Optional per action type (pattern/suggestion), not mandatory for every current or future action.
   2. Includes only variables supported by that action type.
   3. Emits deterministic key ordering for stable debugging output.
   4. Output contract: line-delimited `key=value` entries, one variable per line.
2. Implement runtime generation in Shared so `action.all` is produced from canonical metadata + current invocation values.
3. Ensure designer token discovery and script validation include `action.all` only for action types that declare support.
4. Add safety guidance in cookbook:
   1. Debug-focused use intent.
   2. Formatting contract expectations.
   3. Example scripts.

Exit criteria:

1. `action.all` resolves in simulator/runtime outcome scripts without unknown reference errors for action types that declare support.
2. Output shows supported variable name/value pairs for the current action context.
3. Deterministic output format is covered by tests:
   1. line-delimited `key=value` entries,
   2. one variable per line,
   3. stable ordering + representative value formatting.
4. Cookbook includes approved guidance for game producers and developers.
5. Full validation cycle completed and recorded for this slice.

## 10. Validation Gates

Full validation cycle (mandatory at each slice exit):

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
4. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj

Default:

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj

Focused runtime gate:

1. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"

Replay smoke gate:

1. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"

Additional required tests (minimum):

1. Shared metadata registry coverage by action type.
2. Designer token parity tests against Shared metadata.
3. Runtime emitted-token subset tests against Shared metadata.
4. Action-specific unknown-reference regression tests for success and failure outcome scripts.

## 11. Risks and Mitigations

1. Risk: hidden designer dependencies on old token providers.
   Mitigation: phased adapter approach; parity tests before provider removal.
2. Risk: runtime only populates tokens on subset paths.
   Mitigation: per-result-path population tests; metadata path applicability field.
3. Risk: accidental token contract changes affect authored scripts.
   Mitigation: preserve token strings; add stability assertions in tests.
4. Risk: over-scoping to all actions at once.
   Mitigation: pilot-first rollout and incremental slice expansion.

## 12. Design Questions To Lock Off

These questions must be explicitly resolved (Locked) during execution to avoid hidden drift or implicit contract changes.

| Question ID | Question | Target Slice | Status |
| --- | --- | --- | --- |
| DQ-01 | Canonical metadata location: should action-variable declarations live in a dedicated Shared registry, action-local descriptor classes, or an executor-adjacent map? | R1 | Locked (2026-07-08) |
| DQ-02 | Token scope for metadata: should canonical lists include only base/list/count/index tokens, or also optional diagnostics tokens such as `action.all` on action types that opt in? | R1/R5 | Locked (2026-07-08) |
| DQ-03 | Path applicability model: is `FailureOnly/SuccessOnly/Both` sufficient, or do we need finer-grained applicability dimensions? | R1/R3 | Locked (2026-07-08) |
| DQ-04 | Ordering contract: what deterministic ordering rule is canonical (declaration order, ordinal token sort, or category then ordinal)? | R1 | Locked (2026-07-08) |
| DQ-05 | Designer bridge behavior: for actions not yet migrated, should the registry use fallback legacy providers, hard fail, or warning diagnostics? | R2 | Locked (2026-07-08) |
| DQ-06 | Pilot completion bar: what exact parity checks are required before expanding beyond BuildCompositeByParts? | R1/R2 | Locked (2026-07-08) |
| DQ-07 | Runtime guardrail strictness: should emitted-token checks fail on undeclared tokens only, or also fail when declared tokens are not populated on required paths? | R3 | Locked (2026-07-08) |
| DQ-08 | Versioning policy for token-set evolution: when a token is added or removed, what approval/update steps are mandatory (tests, cookbook, plan log)? | R3/R4 | Locked (2026-07-08) |

DQ-06 pilot completion checklist draft:

1. BuildCompositeByParts token parity is green: Shared canonical declarations match designer-visible token list.
2. Runtime legality is green: emitted token names are declared (no undeclared emissions).
3. Unknown-reference regressions are green for both success and failure scripts using BuildCompositeByParts tokens.
4. Shared-first bridge proof is green: BuildCompositeByParts token resolution is served via Shared-backed provider path, not local literal token list.
5. Validation evidence is recorded: build, focused runtime gate, replay smoke gate, and full test project run.
6. New cookbook draft is published for review based on pilot implementation (explicitly marked draft, subject to later refinement in R4).

DQ-07 pass/fail rules draft (for R3 lockoff):

1. Fail if any runtime-emitted token is not declared in canonical Shared metadata for that action type.
2. Current-phase behavior: declared token names are legal regardless of path; if no concrete value exists for the current path/context, runtime should return empty/no-value (not unknown-reference error).
3. Do not fail current-phase tests solely because a declared token has no value on a given path.
4. Path-specific applicability enforcement (`FailureOnly`/`SuccessOnly`) is deferred to late phase after non-`Both` availability values are enabled.
5. Keep this guardrail test-focused; runtime execution should continue to prefer diagnostics/output behavior over hard runtime exceptions for metadata mismatch.

DQ-03 lock notes:

1. Prepare now for per-token availability metadata, but only support `Both` as an active value in current implementation slices.
2. Defer path-specific availability behavior (`FailureOnly`/`SuccessOnly`) to a late phase to avoid immediate designer complexity spillover.
3. Until path-specific availability is enabled, consumers treat declared token names as legal and receive empty/no-value when unavailable in current context.

## 13. Lock Decisions (To Fill)

| Decision ID | Decision | Status |
| --- | --- | --- |
| AVR-01 | Canonical action variable metadata lives in Storyboard.Shared and is keyed by CommandActionType. | Locked (2026-07-08) |
| AVR-02 | Designer token UX and script validation consume Shared metadata (no independent designer source of truth). | Locked (2026-07-08) |
| AVR-03 | Token strings (`action.*`) remain stable unless explicitly versioned and approved. | Locked (2026-07-08) |
| AVR-04 | Cookbook update + explicit approval is mandatory completion criterion. | Locked (2026-07-08) |
| AVR-05 | Later-phase debug feature `action.all` is runtime-owned, optional per action type, and emits line-delimited `key=value` pairs (one per line) for supported action variables in deterministic order when enabled for that action. | Locked (2026-07-08) |
| AVR-06 | Canonical action-variable declarations are defined in Shared action-local descriptors and registered by action type in a Shared registry; runtime executors must emit only declared tokens. | Locked (2026-07-08) |
| AVR-07 | Canonical action-variable metadata includes only currently resolvable runtime tokens per action type; diagnostics tokens (including `action.all`) remain disabled-capability declarations until implemented, then become enabled canonical tokens only for action types that opt in. | Locked (2026-07-08) |
| AVR-08 | Canonical metadata includes per-token availability shape now, but current slices support only `Both`; if a declared token has no value in current context/path, runtime returns empty/no-value without treating it as an unknown-reference error. | Locked (2026-07-08) |
| AVR-09 | Deterministic token ordering is declaration order from Shared metadata; designer token lists and optional `action.all` output must preserve this order unless an explicit contract change is approved and logged. | Locked (2026-07-08) |
| AVR-10 | During migration, designer token resolution is Shared-first with fallback to legacy providers for non-migrated actions; fallback usage is observable via diagnostics/tests and must be removed or disabled by R4 completion. | Locked (2026-07-08) |
| AVR-11 | Expansion beyond BuildCompositeByParts is blocked until pilot parity/runtime legality/unknown-reference/shared-bridge gates are green and a pilot cookbook draft review package is published. | Locked (2026-07-08) |
| AVR-12 | R3 guardrails fail on undeclared emitted token names; declared token names remain legal in current slices and may resolve to empty/no-value without error when unavailable in context, with path-specific strictness deferred to late phase. | Locked (2026-07-08) |
| AVR-13 | Token additions are expected to be common, fluid, and non-breaking: add to canonical Shared declarations and rely on normal validation/test gates. Token removals/renames are disruptive and require explicit lock decision plus migration notes; impacted producers should receive fast feedback via project validation. | Locked (2026-07-08) |

## 14. Execution Log

2026-07-08:

1. Plan created in active with runtime-owned canonical action-variable strategy.
2. Initial slices defined for lockoff, Shared foundation, designer migration, runtime alignment, cleanup.
3. Cookbook update and approval added as explicit success criterion.
4. Completed Slice R0 inventory baseline and added artifact `plans/active/ACTION_VARIABLE_REFACTOR_INVENTORY_BASELINE.md`.
5. Baseline identifies strongest near-term normalization target: NavigateDirection mixed token model (runtime emits `action.Success`/`action.ResultCode` while designer provider exposes room-scoped tokens only).
6. Added later-phase Slice R5 for runtime-owned `action.all` debug emission, including deterministic output and cookbook guidance gates.
7. Locked early pilot approach: BuildCompositeByParts is the first migration, keeping designer provider interface while serving tokens through a thin Shared-backed facade adapter.
8. Added explicit Design Questions To Lock Off list (DQ-01..DQ-08) so open architecture decisions are tracked and closed intentionally during slices.
9. Locked DQ-01 and recorded AVR-06 to codify Shared action-local descriptor + registry ownership for canonical token declarations.
10. Added DQ-07 pass/fail guardrail draft so R3 strictness can be locked without ambiguity.
11. Locked DQ-02 and recorded AVR-07 to keep canonical token scope runtime-resolvable first, with diagnostics tokens (including `action.all`) staged as disabled capability until implemented.
12. Locked DQ-03 as deferred path-specific applicability: prepare metadata shape now, support `Both` only for current slices, and treat declared-but-unavailable values as empty/no-value rather than errors.
13. Locked DQ-04 and recorded AVR-09 to make declaration order the canonical deterministic token ordering across designer consumption and `action.all` output.
14. Locked DQ-05 and recorded AVR-10 to use Shared-first with legacy fallback during migration, then remove or disable fallback by R4 completion.
15. Locked DQ-06 and recorded AVR-11 to block expansion beyond BuildCompositeByParts until pilot gates and cookbook draft review package are complete.
16. Locked DQ-07 and recorded AVR-12 to enforce undeclared-emission guardrails while preserving legal empty/no-value behavior for declared tokens in current slices.
17. Locked DQ-08 and recorded AVR-13 with a low-friction evolution policy: additive tokens are non-breaking and fluid, while removals/renames require explicit lockoff and migration handling with producer validation feedback.
18. Implemented R1 pilot foundation in Shared with canonical action-variable descriptor types, Shared descriptor registry, and declaration-order token accessor API.
19. Migrated BuildCompositeByParts designer token provider to Shared-backed descriptor consumption and removed local provider token literals.
20. Expanded Shared canonical descriptor coverage to BreakCompositeItem, PutObjectInContainer, and RemoveObjectFromContainer.
21. Migrated BreakCompositeItem and container transfer designer token providers to Shared-backed descriptor consumption and removed local provider token literals.
22. Added provider parity/stability tests asserting migrated designer token lists match Shared canonical declaration order for BuildCompositeByParts, BreakCompositeItem, PutObjectInContainer, and RemoveObjectFromContainer.
23. Ran full validation cycle after migration increment: solution build, focused runtime gate, replay smoke gate, and full test project run (all green).
24. Completed BuildCompositeByParts payload-side deduplication by replacing local `CompositeByPartsPayload.GetReferenceTokens` literals with Shared canonical descriptor lookup.
25. Declared BuildCompositeByParts pilot end-to-end checkpoint complete for implementation readiness (Shared descriptor + provider bridge + payload dedup + script validation compatibility + validation evidence).
26. Pause rule reaffirmed: no additional action-type expansion work proceeds until pilot cookbook review package is reviewed and approved.
27. Pilot approval recorded by owner: BuildCompositeByParts cookbook checkpoint package accepted; post-pilot implementation resumed.
28. Implemented runtime emitted-token legality guardrail tests for migrated actions (BuildCompositeByParts, BreakCompositeItem, PutObjectInContainer, RemoveObjectFromContainer) against Shared canonical metadata declarations.
29. Guardrail run surfaced canonical drift (`action.targetContainerNameCount` emitted by runtime but undeclared); fixed by updating Shared descriptor declaration and associated descriptor expectation tests.
30. Ran full validation cycle after guardrail implementation and drift fix: solution build, focused runtime gate, replay smoke gate, and full test project run (all green).
31. Completed NavigateDirection normalization by adding canonical Shared descriptor coverage for mixed action/room token set and registering it in Shared descriptor registry.
32. Migrated designer NavigateDirection token provider and `NavigatePayload.GetReferenceTokens` to Shared descriptor lookup (removed local literal token lists).
33. Extended parity and legality guardrails for NavigateDirection (registry parity tests, descriptor expectation tests, payload lifecycle token assertions, and emitted-token guardrail coverage).
34. Ran full validation cycle after NavigateDirection migration: solution build, focused runtime gate, replay smoke gate, and full test project run (all green).
35. Completed BreakComposite payload-side deduplication by replacing local `BreakCompositePayload.GetReferenceTokens` literals with Shared canonical descriptor lookup.
36. Added payload lifecycle regression coverage to assert BreakComposite payload token availability from canonical source.
37. Ran full validation cycle after BreakComposite payload deduplication: solution build, focused runtime gate, replay smoke gate, and full test project run (all green).
38. Added ScriptUnknownReferenceRule parity tests for NavigateDirection canonical mixed tokens: acceptance for `action.Success`/`action.ResultCode` + room tokens and rejection for unknown navigate aliases.
39. Ran full validation cycle after Navigate script-validation hardening: solution build, focused runtime gate, replay smoke gate, and full test project run (all green).
40. Added central Shared runtime guardrail utility to validate emitted action-variable keys against canonical metadata and append diagnostics for undeclared emissions.
41. Wired central guardrail checks into runtime executable merge points for BuildCompositeByParts, BreakCompositeItem, PutObjectInContainer, RemoveObjectFromContainer, and NavigateDirection.
42. Added dedicated guardrail unit tests for declared/undeclared/indexed/mixed Navigate token behavior at the central utility level.
43. Ran full validation cycle after central runtime guardrail integration: solution build, focused runtime gate, replay smoke gate, and full test project run (all green).
44. Aligned remaining room-token string literals to canonical RuntimeTokenCatalog constants across Navigate runtime merge values, shared scope-chain aliases, and designer token-seed helpers.
45. Ran full validation cycle after room-token constant alignment: solution build, focused runtime gate, replay smoke gate, and full test project run (all green).
46. Completed final R4 redundancy sweep for migrated paths and confirmed no remaining duplicate token declarations outside canonical Shared descriptor consumption.
47. Recorded cookbook closeout approval and confirmed migrated action-variable guidance is synchronized with central runtime guardrail architecture.
48. Ran full validation cycle for R4 closeout checkpoint: solution build, focused runtime gate, replay smoke gate, and full test project run (all green).
49. Clarified later-phase `action.all` semantics: treat as a normal optional per-action variable pattern (not universal or special); only action types that declare support participate in discovery, validation, and runtime output.
50. Implemented first R5 contract slice by adding canonical `action.all` token key support and opting in NavigateDirection descriptor declarations as a pattern proof (no forced universal enablement).
51. Added opt-in coverage tests: NavigateDirection accepts `action.all` in script unknown-reference validation while a non-opted-in action (BuildCompositeByParts) still reports `action.all` as unknown.
52. Ran full validation cycle after R5 opt-in declaration slice: solution build, focused runtime gate, replay smoke gate, and full test project run (all green).
53. Implemented runtime `action.all` value generation for opted-in actions via a Shared diagnostic value builder that emits declaration-order `key=value` lines and excludes recursive self-inclusion.
54. Wired NavigateDirection outcome emission to append `action.all` when declared for that action type and merge it into invocation-context values/tokens used by outcome script evaluation.
55. Added runtime execution regression coverage asserting NavigateDirection `action.all` output lines are deterministic and declaration-ordered for the moved-path scenario.
56. Re-ran validation for R5 runtime generation increment (build, targeted guardrail/parity suites, and full test suite; all green).
57. Expanded `action.all` opt-in to all composite-related actions: BuildCompositeByTarget, BuildCompositeByParts, and BreakCompositeItem descriptor declarations now include `action.all`.
58. Added BuildCompositeByTarget canonical descriptor/registry coverage with Shared-first designer provider and payload token retrieval aligned to Shared metadata.
59. Wired composite runtime executables to append `action.all` when declared before guardrail evaluation and outcome-script interpolation (BuildCompositeByTarget, BuildCompositeByParts, BreakCompositeItem).
60. Extended script unknown-reference and provider parity tests for composite `action.all` opt-in behavior; non-opted-in control remains enforced.
61. Ran full validation cycle for composite opt-in expansion: solution build, focused runtime gate, replay smoke gate, and full test project run (all green).
62. Expanded R5 opt-in to container transfer actions by declaring `action.all` for PutObjectInContainer and RemoveObjectFromContainer in Shared canonical descriptors.
63. Wired PutObjectInContainer and RemoveObjectFromContainer runtime executables to append `action.all` before emitted-token guardrail checks and invocation-context merge for outcome interpolation.
64. Extended verification for container transfer opt-in behavior: provider parity expectations, script unknown-reference acceptance, and runtime fixture coverage validating deterministic declaration-order `action.all` key output.
65. Ran full validation cycle for container transfer opt-in expansion: solution build (`StoryboardDesigner.slnx`), focused runtime gate, replay smoke gate, and full test project run (all green; 554 passed).
66. Closed R5 non-composite decision pass for the current Shared canonical descriptor scope: no remaining non-composite candidates are pending because NavigateDirection, PutObjectInContainer, and RemoveObjectFromContainer are all opted in to `action.all`; broader action-type expansion remains out of scope until those types are migrated into Shared canonical declarations.
67. Closeout accepted: plan marked Closed (Current Scope); any further findings should be tracked as new bug fixes or follow-on enhancements rather than reopening this refactor plan.

## 15. Immediate Next Step

1. Keep R4 and current-scope R5 closed.
2. Route additional findings through normal bug-fix workflow.
3. For future action-type migrations into Shared descriptors, evaluate optional `action.all` opt-in as part of that new work.
4. Continue using full validation cycle evidence on all follow-on changes.
