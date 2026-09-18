# Action Payload Refactor Plan

Status: Completed (Execution and Closure Recorded)
Owner: StoryboardDesigner.App authoring workflows
Last updated: 2026-07-03

Historical context note: Sections describing initial slice plans and open questions are retained for auditability; completion state and final decisions are recorded in Sections 10-13.

## 1. Purpose

Refactor action authoring data so each action type has a clear, isolated set of data points, while preserving existing project file compatibility and clean export/runtime contract behavior.

## 2. Problem Statement

Current CommandAction is a broad mutable bag containing fields for many unrelated action behaviors.

Primary consequences:

1. Weak field ownership boundaries per ActionType.
2. Repeated ActionType switching across editor, validation, and export mapping.
3. Higher regression risk when adding/changing action kinds.
4. Harder testability because applicability rules are distributed.

## 3. Goals

1. Define explicit field ownership for each supported ActionType.
2. Introduce envelope plus payload shape without breaking existing saved projects.
3. Centralize ActionType applicability and normalization rules.
4. Keep validation and runtime behavior equivalent during transition.
5. Migrate incrementally in small test-backed slices.
6. Standardize outcome messaging so every action can author both success and failure echoes.

## 4. Non-Goals (Initial Slices)

1. No immediate runtime contract schema rewrite.
2. No all-at-once replacement of every editor binding.
3. No broad UI redesign.
4. No removal of legacy fields until payload parity is proven.

## 4.1 Prerequisite: Remove Legacy Linked Commands UI Surface

Before payload migration begins, remove the legacy Linked Commands authoring surface that duplicates action interaction paths.

Scope of prerequisite cleanup:

1. Remove Linked Commands tree node and related tree actions.
2. Remove linked-commands dialog workflow entry points and menu/context actions that open it.
3. Remove view/viewmodel glue used only by the Linked Commands UI feature.

Out of scope for prerequisite cleanup:

1. Do not remove runtime command processing and command model contracts.
2. Do not remove Game Actions authoring flows.
3. Do not change action validation semantics.

Exit criteria:

1. Only one action editing path remains in designer tree: Game Actions.
2. No dead references to removed Linked Commands UI types remain.
3. Build and full test suite are green.

## 5. Target Model

## 5.1 Envelope (stable identity and routing)

CommandAction keeps:

1. Id
2. Name
3. ActionType
4. Trigger fields: NoVerbLinkage, Verbs, DirectionQualifierText
5. Cross-cutting dispatch fields: ChildCommandForwardingMode, SimilarChildDispatchMode
6. Cross-cutting outcome fields: SuccessEchoMessage, FailureEchoMessage
7. Payload (new typed payload container)

## 5.2 Payload Families (new)

1. LinkedFlowPayload
2. SynonymPayload
3. EchoPayload
4. CheckGamePropertyPayload
5. SetGamePropertyPayload
6. ContainerTransferPayload
7. NavigatePayload
8. CompositeByTargetPayload
9. CompositeByPartsPayload
10. BreakCompositePayload

Resolved: OptionalEchoMessage was removed. Canonical outcome storage is OutcomeMessageMap keyed by result-code token.

## 6. ActionType Ownership Matrix (v1)

Envelope-only fields are omitted from per-type rows unless needed for clarity.

| ActionType | Owned payload fields |
| --- | --- |
| LinkedActions | LinkedActions |
| Synonym | SynonymTargetActionId |
| EchoMessage | OutcomeMessageMap |
| CheckGameProperty | FlagName, FlagValue |
| SetGameProperty | GamePropertyName, GamePropertyValue |
| PutObjectInContainer | TargetContainerId, OutcomeMessageMap |
| RemoveObjectFromContainer | TargetContainerId, OutcomeMessageMap |
| NavigateDirection | OutcomeMessageMap |
| BuildCompositeByTarget | CompositeTargetObjectId, CompositeRecipeId, CompositeRequiredPartObjectIds, CompositeStrictPartCountEnforcement, CompositeMinimumRequiredPartCount, CompositePartConsumptionMode, OutcomeMessageMap |
| BuildCompositeByParts | CompositeTargetObjectId, CompositeRecipeId, CompositeRequiredPartObjectIds, CompositeStrictPartCountEnforcement, CompositeMinimumRequiredPartCount, CompositeMatchMode, CompositeAmbiguityPolicy, CompositePartConsumptionMode, CompositeResolvedTargetOutputTemplate, OutcomeMessageMap |
| BreakCompositeItem | CompositeTargetObjectId, CompositeRecipeId, CompositeRequiredPartObjectIds, CompositeStrictPartCountEnforcement, CompositeMinimumRequiredPartCount, CompositePartConsumptionMode, OutcomeMessageMap |

## 7. Migration Strategy

## 7.0 Prerequisite Slice: Linked Commands UI Cleanup

Execution order note: Slice 7.0 runs first and must complete before payload implementation slices begin.

1. Remove Linked Commands tree node types and navigation wiring that exist only to support the old dialog flow.
2. Remove Linked Commands dialog workflow service method and command wiring.
3. Remove Linked Commands dialog artifacts and feature-exclusive VM helpers.
4. Keep RoomCommand model/runtime paths intact until explicitly planned otherwise.

Exit criteria:

1. Designer tree exposes only Game Actions for action authoring.
2. No compile references to removed Linked Commands UI-only types remain.
3. Validation/build/test baseline remains green.

## 7.1 Slice 0: Discovery and Lock

1. Freeze ownership matrix and unresolved decisions.
2. Add focused regression tests for current behavior before data-shape changes.
3. Document invariants for save-load-save and clean export parity.
4. Run OptionalEchoMessage review and decide one of:
   a. Keep with clearer semantics and naming.
   b. Move to a better-scoped outcome field model.
   c. Deprecate and remove as dead/legacy behavior.

OptionalEchoMessage audit checklist (required):

1. Code usage inventory:
   a. Find all model, editor, validator, export/import, runtime mapper, and runtime executor references.
   b. Classify each reference as authoring UX, persistence contract, runtime behavior, or validation-only.
2. Behavior inventory:
   a. List every ActionType where OptionalEchoMessage is currently editable.
   b. List every ActionType where OptionalEchoMessage affects runtime output.
   c. Confirm whether OptionalEchoMessage can conflict with new envelope SuccessEchoMessage/FailureEchoMessage semantics.
3. Data inventory:
   a. Scan representative sample projects for OptionalEchoMessage presence and non-empty usage.
   b. Identify whether values are meaningful author-authored content or mostly empty/default residue.
4. Naming and semantics review:
   a. Document intended semantics in plain language (when it should run, why it exists).
   b. Compare with SuccessEchoMessage/FailureEchoMessage semantics and identify overlap.
5. Decision rubric:
   a. Keep/Rename only if it has a distinct, non-overlapping semantic and measurable active use.
   b. Rehome if semantic is valid but field placement/name is misleading.
   c. Remove if semantic overlaps outcome echoes or active usage is negligible/accidental.
6. Migration impact note (if rehome/remove):
   a. Define automatic migration behavior for existing project files.
   b. Define UI messaging for changed behavior (if any).
   c. Add regression tests proving no unintended runtime output loss.

Exit criteria:

1. Matrix approved.
2. Baseline tests pass and are linked in this plan.
3. OptionalEchoMessage disposition is decided and recorded.
4. OptionalEchoMessage audit evidence is linked in this plan (files/tests/results).

Mandatory Slice 0 baseline commands:

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"

Mandatory Slice 0 evidence capture in this plan:

1. Command run list with pass/fail outcome and date.
2. Save-load-save parity verification result for representative fixtures.
3. Clean export parity verification result for representative fixtures (including Birmingham baseline coverage).
4. Any approved snapshot baseline update notes.

Gate: Slice 1 implementation does not begin until Slice 0 evidence is recorded.

## 7.2 Slice 1: Introduce Payload Contracts and Bridge

1. Add payload interfaces/classes in StoryboardDesigner.App model layer.
2. Add payload property on CommandAction.
3. Add bridge methods:
   a. Legacy fields to payload.
   b. Payload to legacy fields.
4. Add centralized ActionType schema map (owned fields, required fields, normalizers).

Exit criteria:

1. No behavior change visible to users.
2. Build and test green.

## 7.3 Slice 2: Editor Integration (first families)

1. Move Echo and GameProperty editor paths to payload-backed accessors.
2. Replace duplicated ActionType conditionals with schema helper.
3. Add tests for payload-backed validation in editor save flow.

Exit criteria:

1. Echo and property actions are payload-first.
2. Legacy fields still mirrored for compatibility.

## 7.4 Slice 3: Export and Import Mapping Cutover

1. Update JsonExportService mapping to read payload first.
2. Keep fallback to legacy fields for old data.
3. Preserve exported contract shape in v1.

Exit criteria:

1. Snapshot tests unchanged unless intentionally updated.
2. Load/export round-trip parity verified.

## 7.5 Slice 4: Validation and Summary Cleanup

1. Update validators to consume payload fields through schema helpers.
2. Move summary formatting out of CommandAction into presenter/service.

Exit criteria:

1. No summary regressions.
2. Validation suite green.

## 7.6 Slice 5: Legacy Field Removal

1. Remove deprecated flat fields from CommandAction.
2. Remove bridge code.
3. Tighten tests to enforce payload-only model.
4. Run one-time migration for maintained sample projects and checked-in fixtures.
5. Remove remaining migration/shim helpers after migration is complete.

Exit criteria:

1. No legacy field references remain in app code.
2. Build and tests green.
3. Sample projects/fixtures are migrated and pass parity validation.
4. No migration/shim code paths remain in production authoring/runtime mapping flows.

## 7.7 Post-Cutover Bridge Retirement Gate

Purpose: ensure transitional compatibility code does not linger.

1. Confirm one-time migration completed for maintained sample projects and fixtures.
2. Remove all temporary migration/bridge/shim utilities introduced for payload cutover.
3. Remove transitional tests that only protect bridge behavior.
4. Keep only explicitly approved permanent backward-read compatibility behavior.

Exit criteria:

1. No temporary compatibility TODOs remain for payload migration.
2. Full build and test suite are green.
3. Grep for retired bridge markers returns zero production hits.

## 8. Testing Gates

For each slice, run:

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj

Add focused tests for:

1. ActionType field ownership and normalization.
2. Payload to legacy and legacy to payload bridge correctness.
3. Save-load-save parity.
4. Clean export parity for representative fixtures.

## 9. Risks and Mitigations

1. Risk: Hidden dependencies on legacy fields in views and services.
Mitigation: Keep bridge phase and use grep-based reference inventory before removals.

2. Risk: Export drift due to new mapping paths.
Mitigation: Snapshot and round-trip tests at each slice.

3. Risk: Partial migration leaves inconsistent behavior.
Mitigation: ActionType schema map as single source of truth from Slice 1.

## 10. Progress Tracker

- [x] Slice 7.0 prerequisite: Linked Commands UI cleanup
- [x] Slice 0 approved and locked
- [x] Slice 1 payload contracts and bridge
- [x] Slice 2 editor migration for first families
- [x] Slice 3 export/import payload-first cutover
- [x] Slice 4 validation and summary cleanup
- [x] Slice 5 legacy field removal
- [x] Slice 7.7 post-cutover bridge retirement gate

## 10.1 Baseline Evidence Log

Recorded: 2026-07-03

1. Command: dotnet build .\StoryboardDesigner.slnx
   Outcome: Pass
   Summary: Build succeeded for Storyboard.Shared, Storyboard.Simulator, StoryboardDesigner.App, and StoryboardDesigner.App.Tests.
2. Command: dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
   Outcome: Pass
   Summary: total 288, failed 0, succeeded 288, skipped 0.
3. Command: dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"
   Outcome: Pass
   Summary: total 64, failed 0, succeeded 64, skipped 0.
4. Save-load-save parity verification
   Outcome: Pass
   Evidence: JsonExportServiceProjectStateTests round-trip coverage (linked room instance metadata, contained individual objects, traversal scaffolding, directional traversal mappings).
5. Clean export parity verification
   Outcome: Pass
   Evidence: JsonExportServiceCleanExportTests, JsonExportServiceCleanExportSnapshotTests, JsonExportServiceCleanExportValidationTests, and JsonExportServiceRegressionProtectionTests.
   Birmingham coverage: confirmed via JsonExportServiceCleanExportValidationTests using Samples/Birmingham/Birmingham.sbe.json.

## 11. Decisions Log

2026-07-03:

1. Refactor direction set to envelope plus payload.
2. Keep ActionType discriminator during transition.
3. Maintain backward compatibility during early slices.
4. Slice 0 evidence captured and locked (build, full tests, runtime-focused regression, save-load-save parity, clean export parity with Birmingham coverage).
5. Slice 1 scaffolding completed: payload contracts, schema map, and legacy bridge entry points added with regression tests passing.
6. Slice 2 first-family editor integration completed for Echo and GameProperty actions via payload-backed accessors while preserving legacy-field mirroring.
7. Slice 2 validation passed: focused filter (`ActionPayloadBridgeTests|RoomActionEditorDialog|ActionIntegrityRulesTests|JsonExportServiceProjectStateTests`) and full test suite green.
8. Slice 3 started with first-family payload-first export/import mapping (Echo, CheckGameProperty, SetGameProperty) while preserving existing JSON DTO shape.
9. Added regression coverage proving save/load uses payload-backed values for first families (`SaveProjectModel_UsesPayloadBackedActionValues_ForEchoAndGamePropertyFamilies`).
10. Slice 3 incremental validation passed: focused export/payload filters and full test suite green.
11. Slice 3 payload-first export mapping now covers remaining families (Synonym, PutObjectInContainer, NavigateDirection, BuildCompositeByTarget, BuildCompositeByParts, BreakCompositeItem) with legacy fallback preserved.
12. Expanded regression coverage to cross-family payload-first save/load verification (`SaveProjectModel_UsesPayloadBackedActionValues_AcrossActionFamilies`).
13. Slice 3 completion validation passed: build, focused export/payload regression filter, and full test suite green.
14. Slice 4 started: validation rules now consume payload-backed helpers for synonym/link checks and script field analysis, including action-node script facet detection.
15. Added payload-oriented validation regressions (`SynonymMissingTargetRule_UsesPayloadTargetId_WhenPresent`, `Evaluate_ReportsWarning_WhenPayloadEchoScriptUsesUnknownReference`).
16. Slice 4 incremental validation passed: focused validation/scripting filters and full test suite green.
17. Summary formatting extracted from CommandAction model into dedicated presenter service (`CommandActionSummaryPresenter`) with model properties delegating to presenter.
18. Added summary presenter regression coverage (`CommandActionSummaryPresenterTests`) for payload-backed echo/check summary behavior and tooltip rendering.
19. Slice 4 completion validation passed: solution build, focused summary/payload/validation filters, and full test suite green.
20. Post-slice hardening: added typed payload accessors for container-transfer, navigate, and composite families; summary presenter now consumes these accessors instead of legacy flat fields.
21. Expanded summary regression coverage for payload-backed container/composite family summaries.
22. Hardening validation passed: focused summary/payload tests and full suite green.
23. Slice 5 started (non-destructive migration batch): ProjectModelRuntimeSnapshotMapper and AreaNavigationEditorTabViewModel action-cloning paths now consume payload accessors with legacy fallback preserved; validation passed (focused 87/87 and full suite 303/303).
24. Slice 5 continuation batch: RoomActionEditorDialog working-copy initialization and save-apply paths now hydrate/persist synonym/container/navigate/composite families via centralized payload accessors, with legacy fields still mirrored for compatibility.
25. JsonExportService payload fallback helpers now delegate to ActionPayloadAccessors for synonym/container/navigate/composite families, reducing duplicated legacy fallback logic.
26. Slice 5 continuation validation passed: focused editor/export/payload filter tests (38/38) and full suite green (303/303).
27. Slice 5 continuation batch: ScopedActionsDialog action cloning now hydrates payload-owned fields via ActionPayloadAccessors (echo/check/set/synonym/container/navigate/composite) and carries payload snapshots forward.
28. ScopedActionsDialog validation now reads payload-backed echo and set-property script fields through ActionPayloadAccessors.
29. Slice 5 continuation validation passed: focused payload/linked-actions/export-state filters (74/74) and full suite green (303/303).
30. Slice 5 boundary hardening: ScopedActionsDialog now enforces payload synchronization at authoring boundaries (new action creation and save persistence) via SyncPayloadFromLegacyFields.
31. Boundary hardening validation passed: focused payload/linked-actions/export-state filters (74/74) and full suite green (303/303).
32. Slice 5 continuation batch: RoomActionEditorDialog validation/defaulting/script-editor flows now read set-property/container/navigate/synonym values through payload accessors and helper methods, preserving existing UI behavior.
33. Added local helper adapters in RoomActionEditorDialog for synonym/navigate payload reads-writes to keep payload and legacy mirrors aligned during editing.
34. Slice 5 continuation validation passed: focused dialog/payload/export-state filters (30/30) and full suite green (303/303).
35. Residual-read migration batch: LinkedActionGraphValidator now resolves synonym targets through ActionPayloadAccessors; RoomActionEditorDialog constructor/save validation now hydrates and validates echo/check/set values via payload-backed accessors.
36. Residual-read migration validation passed: focused dialog/payload/linked-actions filters (60/60) and full suite green (303/303).
37. Residual-read migration batch: MainWindowViewModel.ProjectExplorer CloneActions path now clones action-family fields through ActionPayloadAccessors (echo/check/set/container/navigate/synonym/composite) with payload sync after link remapping.
38. Project explorer clone-path validation passed: focused main-window/payload/export-state filters (37/37) and full suite green (303/303).
39. Residual-read cleanup batch: MainWindowViewModel.GameSimulator token extraction and traversal look-action provisioning now use payload accessors for echo/set-property fields; CommandActionSummaryPresenter SetFlag summary now uses accessor-backed check fields.
40. Residual cleanup validation passed: focused main-window/summary/payload/linked-actions filters (63/63) and full suite green (303/303).
41. Legacy-hydration reduction batch: JsonExportService ToCommandActionModel no longer broadly hydrates Echo/Flag/Set legacy fields during model creation; action-type payload setter paths now authoritatively populate those families (including SetFlag via check-property setter).
42. Hydration reduction validation passed: focused export/payload/integrity/linked-actions filters (82/82) and full suite green (303/303).
43. First destructive storage pass (echo/check/set): CommandAction removed primitive backing storage for EchoMessage/FlagName/FlagValue/GamePropertyName/GamePropertyValue and now stores these via typed facet payload caches (EchoPayload, CheckGamePropertyPayload, SetGamePropertyPayload), with payload assignments syncing facet caches.
44. Destructive pass regression note: implicit payload mutation from property setters was reverted to preserve existing payload lifecycle expectations (payload remains explicit via TryApplyPayload/SyncPayloadFromLegacyFields).
45. Destructive pass validation passed: focused bridge/editor/integrity/export/linked-actions filters (74/74) and full suite green (303/303).
46. Second destructive storage pass (container/navigate): CommandAction removed primitive backing storage for TargetContainerId and moved navigate/container result scripts to canonical OutcomeMessageMap, with typed facet payload caches retained for non-script payload data (ContainerTransferPayload, NavigatePayload).
47. Payload assignment sync now updates container and navigate facet caches in CommandAction alongside existing echo/check/set cache synchronization.
48. Second destructive pass validation passed: focused bridge/editor/integrity/export/linked-actions/summary filters (79/79) and full suite green (303/303).
49. Third destructive storage pass (synonym/composite): CommandAction removed primitive backing storage for SynonymTargetActionId and all composite fields, replacing them with typed facet payload caches (SynonymPayload, CompositeByTargetPayload, CompositeByPartsPayload, BreakCompositePayload).
50. Composite property accessors now route through action-type-aware facet selection and synchronized facet updates to preserve current editing and export semantics while eliminating legacy backing fields.
51. Third destructive pass validation passed: focused bridge/editor/integrity/export/linked-actions/summary/composite filters (112/112) and full suite green (303/303).
52. Bridge-retirement cleanup: JsonExportService no longer uses local Resolve* payload shim methods; export/DTO/clean-export mappings now call ActionPayloadAccessors directly at each mapping site.
53. Removed redundant ResolveSynonymTargetActionId/ResolveContainerTransferPayload/ResolveNavigatePayload/ResolveCompositeByTargetPayload/ResolveCompositeByPartsPayload/ResolveBreakCompositePayload helper methods from JsonExportService.
54. Bridge-retirement cleanup validation passed: focused export/bridge/integrity/linked-actions/summary filters (87/87) and full suite green (303/303).
55. Bridge-retirement cleanup: RoomActionEditorDialog now inlines direct ActionPayloadAccessors reads for working synonym target and navigate payload access, removing redundant local wrapper methods.
56. Removed obsolete dialog shim methods GetWorkingSynonymTargetActionId and GetWorkingNavigatePayload; remaining navigate/synonym helpers now only perform write/sync behavior.
57. Dialog shim-retirement validation passed: focused bridge/editor/export/integrity/linked-actions filters (90/90) and full suite green (303/303).
58. Bridge-retirement cleanup: removed redundant SyncPayloadFromLegacyFields calls for newly created EchoMessage actions in ProjectExplorer traversal look-action creation and ScopedActionsDialog add-action flow.
59. Rationale: ActionPayloadAccessors.SetEchoMessage already assigns EchoPayload when ActionType is EchoMessage, so immediate sync was duplicate work with no behavior change.
60. Redundant-sync retirement validation passed: focused bridge/editor/export/integrity/linked-actions/project-explorer filters (90/90) and full suite green (303/303).
61. Bridge-marker retirement batch: replaced all remaining SyncPayloadFromLegacyFields call sites with equivalent explicit payload snapshot assignment (`Payload = CreatePayloadSnapshot()`) in production and bridge tests.
62. Removed obsolete `CommandAction.SyncPayloadFromLegacyFields` helper after confirming zero remaining references across app and tests.
63. Bridge-marker retirement validation passed: focused bridge/editor/export/integrity/linked-actions/project-explorer filters (90/90), full suite green (303/303), and grep for `SyncPayloadFromLegacyFields(` returned zero hits.
64. Bridge-retirement cleanup: inlined payload snapshot creation and payload application logic directly into `CommandAction.CreatePayloadSnapshot` and `CommandAction.TryApplyPayload`, preserving compatibility checks and field-mapping behavior.
65. Removed obsolete `ActionPayloadBridge` utility file after inlining; grep now shows no production references to `ActionPayloadBridge` symbols.
66. ActionPayloadBridge retirement validation passed: focused bridge/editor/export/integrity/linked-actions/project-explorer filters (90/90) and full suite green (303/303).
67. Transitional test cleanup: retired bridge-era test artifact naming by replacing `ActionPayloadBridgeTests` with payload-lifecycle focused coverage in `CommandActionPayloadLifecycleTests` while preserving the same behavioral assertions.
68. Updated linked-actions snapshot test naming to reflect explicit payload snapshot semantics (`CreatePayloadSnapshot_CapturesLinkedActionsSnapshot`) rather than removed bridge helper semantics.
69. Test-artifact retirement validation passed: focused payload-lifecycle/editor/export/integrity/linked-actions filters (90/90), full suite green (303/303), and grep for `ActionPayloadBridge` in test sources returned zero hits.
70. Post-cutover terminology cleanup: updated remaining payload-lifecycle test names/messages to remove obsolete "legacy fields" wording now that payload bridge code paths are retired.
71. Terminology-cleanup validation passed: focused payload-lifecycle/editor/export/integrity/linked-actions filters (90/90) and full suite green (303/303).
72. Legacy-terminology inventory pass: retained intentional compatibility fixture naming in JsonExportService project-state tests (`LegacyState`, `LegacyShape`, `LegacyGlobals`) while removing non-semantic legacy placeholder strings from summary/payload lifecycle tests.
73. Terminology follow-up validation passed: focused summary/payload-lifecycle/editor/export/integrity/linked-actions filters (95/95) and full suite green (303/303).
74. Production terminology cleanup: renamed private file-command validation parser helper `ParseLegacyValidationError` to neutral `ParseValidationErrorLine` while keeping compatibility parsing behavior and issue codes unchanged.
75. Production terminology cleanup validation passed: focused main-window/payload/export/integrity/linked-actions/summary filters (98/98) and full suite green (303/303).
76. Production terminology cleanup: JsonExportService renamed legacy-labeled traversal passable migration helper/local symbols to neutral compatibility wording (`ApplyTraversalPassableVariableCompatibilityToLegs`, `traversalPassableVariable`) with unchanged migration behavior.
77. Compatibility-helper cleanup validation passed: focused export/payload/integrity/linked-actions/summary/main-window filters (91/91) and full suite green (303/303).
78. Production terminology cleanup: renamed `TraversalConnectionNormalizationUtility.NormalizeLegacyLinks` to neutral `NormalizeLinks` with unchanged normalization behavior.
79. Terminology cleanup validation passed: focused traversal/export/payload/integrity/linked-actions/summary filters (92/92) and full suite green (303/303).
80. Q1 implementation started (slice 1): added envelope-level `SuccessEchoMessage` and `FailureEchoMessage` fields to designer/runtime action models and clean/runtime DTO contracts while retaining `OptionalEchoMessage` for backward-read compatibility.
81. Added deterministic Optional->Success compatibility mapping on load/bootstrap paths (`ToCommandActionModel`, runtime snapshot mapper, clean runtime bootstrap mapper) when success echo is empty.
82. Q1 slice-1 validation passed: focused runtime/export/payload/integrity/summary/guardrail filters (93/93) and full suite green (303/303).
83. Q1 runtime semantics slice: RuntimeCommandActionExecutor now evaluates outcome echoes by core outcome (`SuccessEchoMessage` on logical success, `FailureEchoMessage` on logical failure) and keeps non-Echo OptionalEcho compatibility fallback for success paths while transition is in progress.
84. Added runtime regression coverage in GameCommandProcessorLinkedActionsTests for success outcome echo emission and failure outcome echo emission when core action fails.
85. Q1 runtime semantics validation passed: focused linked-actions/runtime/export/guardrail filters (81/81) and full suite green (303/303).
86. Q1 export-write transition slice: JsonExportService stopped writing `OptionalEchoMessage` for project/export/clean action DTO outputs; write paths now emit `SuccessEchoMessage`/`FailureEchoMessage` and apply Optional->Success fallback at write-time when success echo is empty.
87. Added regression coverage (`SaveProjectModel_DoesNotWriteOptionalEchoMessage_AndFallsBackToSuccessEchoMessage`) proving `optionalEchoMessage` is omitted from saved action JSON while authored Optional content is preserved via `successEchoMessage`.
88. Q1 export-write transition validation passed: focused export/runtime/guardrail filters (81/81) and full suite green (305/305).
89. Q1 runtime contract retirement slice: removed `OptionalEchoMessage` from `IRuntimeCommandAction` and `RuntimeCommandActionDescriptor`; runtime execution now uses only outcome channels (`SuccessEchoMessage`/`FailureEchoMessage`) with no executor-level Optional fallback.
90. Preserved backward-read compatibility at mapping boundaries by retaining deterministic Optional->Success fallback in project/runtime snapshot mapping and clean-runtime bootstrap mapping, while stopping Optional field projection into runtime descriptors.
91. Q1 runtime contract retirement validation passed: focused runtime/export/guardrail filters (85/85) and full suite green (306/306).
92. Q1 designer-surface retirement slice: RoomActionEditorDialog replaced the single Optional echo editor with explicit Success/Failure outcome echo editors and validates both outcome channels during save; Optional echo save/edit/status paths were removed from the dialog.
93. Scoped action cloning/defaulting/validation and summary/token helpers now treat outcome echoes as the canonical secondary script channels (`SuccessEchoMessage`/`FailureEchoMessage`), removing Optional echo validation/token-summary/script-field participation from these authoring surfaces.
94. Q1 designer-surface retirement validation passed: focused editor/validation/runtime/export/guardrail filters (91/91) and full suite green (306/306).
95. Q1 model-retirement slice: removed `OptionalEchoMessage` from `CommandAction` and eliminated remaining Optional clone-path usage in AreaNavigation and ProjectExplorer action cloning.
96. Updated project/runtime mapping to stop depending on model Optional state (`ProjectModelRuntimeSnapshotMapper` now maps `SuccessEchoMessage` directly) and kept backward-read migration anchored at JSON DTO import (`ToCommandActionModel` local Optional->Success migration).
97. Regression coverage update: project-state Optional compatibility test now injects legacy `optionalEchoMessage` through persisted room JSON before reload/save, proving migration behavior without model-level Optional storage.
98. Q1 model-retirement validation passed: focused runtime/export/project-state/guardrail filters (83/83) and full suite green (306/306).
99. Applied migration-first policy for zero-user rollout: one-time updated maintained Birmingham sample artifacts to canonical `successEchoMessage` and removed remaining OptionalEcho bridge usage from designer/shared DTO contracts and clean runtime bootstrap mapping.
100. Removed backward-read Optional migration path from JsonExportService model import (`ToCommandActionModel`) and retired OptionalEcho members from RoomExport/Clean DTO contracts to reduce long-term contract/scaffold complexity.
101. Updated project-state regression to assert canonical behavior (`successEchoMessage` persisted, `optionalEchoMessage` omitted) without legacy Optional fixture injection.
102. Post-migration validation passed: focused export/runtime/guardrail filters (81/81), full suite green (306/306), and workspace search confirms no production OptionalEcho references remain.
103. Residual terminology cleanup: updated remaining test identifiers to outcome-channel wording while preserving explicit assertion that legacy `optionalEchoMessage` is omitted from persisted JSON.
104. Terminology cleanup validation passed: focused export/runtime/guardrail filters (81/81) and full suite green (306/306).
105. Plan closure pass synchronized document state to implementation reality: status moved to Completed and progress tracker now marks Slice 5 and Slice 7.7 done.
106. Final focused closure validation passed with Birmingham/export/runtime guardrail filters (`JsonExportServiceCleanExportValidationTests|JsonExportServiceCleanExportTests|JsonExportServiceProjectStateTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests`) at 35/35.
107. Final full-suite closure validation passed: 306/306 green.
108. Final repository sweep confirms no remaining production bridge markers (`ActionPayloadBridge`, `SyncPayloadFromLegacyFields`) and no production OptionalEcho references; remaining mentions are intentional historical narrative and one write-omission regression assertion.
109. Final drift check completed: payload plan shows no unresolved checklist items or active-status markers, and code search confirms zero production bridge/OptionalEcho references in `Storyboard.Shared` and `StoryboardDesigner.App` sources.

## 12. Resolved Questions and Defaults

Historical snapshot of design decisions resolved during implementation.

| ID | Question | Proposed Default | Status | Final Decision |
| --- | --- | --- | --- | --- |
| Q1 | OptionalEchoMessage disposition: keep/rename/rehome/remove | Perform a targeted behavior audit in Slice 0; prefer deprecate/remove if no clear distinct semantics remain | Decided | Remove OptionalEchoMessage entirely. Use envelope SuccessEchoMessage and FailureEchoMessage (both optional authoring fields) everywhere OptionalEchoMessage was previously used. |
| Q2 | LinkedActions ownership: envelope vs LinkedFlowPayload | Keep LinkedActions on envelope during transition; expose through LinkedFlowPayload adapter where needed | Decided | Keep LinkedActions on envelope for slices 1-5. Payload abstractions may access via adapters; revisit physical relocation only after payload migration if a measurable benefit exists. |
| Q3 | Check action naming: keep FlagName/FlagValue or normalize now | Keep legacy names during bridge; normalize naming in payload abstractions only | Decided | Keep legacy FlagName/FlagValue for bridge compatibility through migration slices. Payload APIs use normalized names (PropertyName, ExpectedValue) from Slice 1 onward; legacy names retire with bridge removal. |
| Q4 | Composite payload shape: one shared payload vs split payloads | Use split payloads as currently planned (ByTarget, ByParts, BreakComposite) for clearer ownership | Decided | Use split payloads as canonical storage model with one-to-one ActionType separation. Shared helper logic is allowed in services/mappers only. |
| Q5 | Payload container shape on CommandAction | Use single payload interface property with ActionType discriminator enforcement via schema map | Decided | Use a single payload property on CommandAction (IActionPayload Payload). ActionType remains discriminator/source of truth; schema/normalizer enforces payload compatibility at edit/load/save/export boundaries. No union wrapper type in slices 1-5. |
| Q6 | Ownership of normalization/defaulting rules | Centralize in action schema/normalizer service; keep model setters minimal | Decided | Centralize normalization/defaulting in action schema/normalizer service. Keep model setters minimal and side-effect free. Apply normalization at edit/load/save/export boundaries. |
| Q7 | Behavior when ActionType changes | Clear non-owned payload fields immediately in working copy/editor model to enforce strict ownership | Decided | On ActionType change, clear non-owned type-specific data immediately in working model; retain envelope cross-cutting fields. Show confirmation when non-empty type-specific data would be lost, then run schema normalizer immediately after switch. |
| Q8 | Slice 7.0 deletion boundary | Remove Linked Commands UI-only tree/dialog/workflow code only; keep RoomCommand model and runtime mapping intact | Decided | Slice 7.0 removes Linked Commands UI-only tree/dialog/workflow code first. RoomCommand model, runtime processing behavior, and contracts remain unchanged in this slice. |
| Q9 | Legacy bridge retirement timing | Keep bridge through Slice 4; remove in Slice 5 with payload-only enforcement | Decided | Bridge/shim logic is allowed only through slices 1-4 for controlled migration. Slice 5 removes bridge/shim code from production paths. Slice 7.7 hard gate verifies one-time sample migration and zero temporary compatibility leftovers. |
| Q10 | Slice 0 mandatory baseline tests | Require full build + full app tests + focused save-load-save and clean export parity checks | Decided | Slice 0 requires solution build, full app tests, focused runtime-boundary regression filter, and recorded save-load-save plus clean export parity evidence before Slice 1 starts. |
| Q11 | Universal action outcome messaging | Add SuccessEchoMessage and FailureEchoMessage as envelope-level fields; all action editors expose both fields for authoring | Decided | Envelope owns SuccessEchoMessage and FailureEchoMessage for all action types |

## 12.1 Q1 Evidence Snapshot (OptionalEchoMessage)

Status: Decided

Code usage inventory summary:

1. Designer app references: 39 matches across model, dialog editors, validation, and export/import mapping.
2. Shared/runtime references: 8 matches across runtime action contracts, bootstrap mapping, runtime descriptor, and executor.
3. Test references: active runtime behavior tests exist in GameCommandProcessorLinkedActionsTests.

Authoring/editability findings:

1. RoomActionEditorDialog exposes a top-level action-echo editor for all action types.
2. For EchoMessage actions, the same button edits EchoMessage (not OptionalEchoMessage), and save flow clears OptionalEchoMessage.
3. ScopedActionsDialog validates OptionalEchoMessage whenever populated, regardless of action type.

Runtime semantics findings:

1. Runtime executor evaluates OptionalEchoMessage for all non-EchoMessage action types after core action execution.
2. OptionalEchoMessage result currently participates in overall logical success via logical AND with core outcome.
3. EchoMessage action type explicitly skips OptionalEchoMessage runtime execution.

Persistence and contract findings:

1. OptionalEchoMessage is persisted in project DTO mapping and clean export DTO mapping.
2. Runtime bootstrap maps OptionalEchoMessage into runtime descriptors.
3. Representative sample data includes non-empty OptionalEchoMessage usage (Birmingham fixture, SetGameProperty action).

Observed confusion/risk:

1. Name "OptionalEchoMessage" under-specifies execution semantics (it is effectively a post-action script echo for non-EchoMessage actions).
2. Current behavior overlaps conceptually with planned universal SuccessEchoMessage/FailureEchoMessage and may create duplicate/ambiguous author intent.

Final decision and migration direction:

1. OptionalEchoMessage is deprecated and removed from model, editor, DTOs, runtime contracts, and validation references.
2. SuccessEchoMessage and FailureEchoMessage become envelope-level optional authoring fields for every action type.
3. Existing OptionalEchoMessage values are migrated deterministically:
   a. If both SuccessEchoMessage and FailureEchoMessage are empty, copy OptionalEchoMessage to SuccessEchoMessage.
   b. If SuccessEchoMessage is empty and FailureEchoMessage is non-empty, copy OptionalEchoMessage to SuccessEchoMessage.
   c. If SuccessEchoMessage is non-empty, keep it and do not overwrite; OptionalEchoMessage is dropped.
4. Runtime execution must evaluate outcome echoes by outcome semantics:
   a. Evaluate SuccessEchoMessage on logical success outcome.
   b. Evaluate FailureEchoMessage on logical failure outcome.
   c. Omit evaluation when the corresponding field is empty.
5. EchoMessage action type remains supported as primary action script content; optional outcome echoes are still allowed but not required.

Implementation notes for upcoming slices:

1. Slice 1: add envelope SuccessEchoMessage/FailureEchoMessage and bridge migration from OptionalEchoMessage on load.
2. Slice 2: expose Success/Failure fields in action editors for all action types and remove OptionalEcho UI.
3. Slice 3: stop writing OptionalEchoMessage in exports/contracts while preserving backward read compatibility during transition.
4. Slice 5: remove all remaining OptionalEchoMessage bridge code and tests that assert old semantics.

Evidence pointers:

1. Storyboard.Shared/GameServices/Actions/RuntimeCommandActionExecutor.cs
2. Storyboard.Shared/GameServices/Actions/IRuntimeCommandAction.cs
3. StoryboardDesigner.App/Views/RoomActionEditorDialog.xaml.cs
4. StoryboardDesigner.App/Views/RoomActionEditorDialog.xaml
5. StoryboardDesigner.App/Views/ScopedActionsDialog.xaml.cs
6. StoryboardDesigner.App/Services/JsonExportService.cs
7. StoryboardDesigner.App.Tests/GameCommandProcessorLinkedActionsTests.cs
8. Samples/Birmingham/Birmingham.sbe.rooms/0fc05c99f4fe4304b650f232b9eca7c5.room.json

## 12.2 Design Considerations (Future, Out of Current Scope)

1. Payload-level inheritance may be considered later if clear commonality emerges between payloads.
2. Current migration still prefers one-to-one payload storage per ActionType for strict field ownership.
3. If inheritance is introduced later, it should remain an implementation convenience and must not blur ActionType ownership boundaries.

## 12.3 Reviewer Callout Requirement (Normalizer Implementation)

When Q6 normalizer/schema logic is implemented, explicitly call this out in the implementation summary and include direct file+line hyperlinks for review.

Minimum review links to provide:

1. Normalizer entry point(s).
2. ActionType-to-payload schema map.
3. Boundary invocation points (load/import, ActionType change in editor working copy, pre-save/export, runtime snapshot mapping).
4. Idempotency test(s) and any change-report/audit test(s).

## 13. Final Closure Summary

Completion status:

1. Payload-first model/storage migration is complete across authoring/runtime/export paths.
2. Temporary bridge/shim scaffolding introduced for migration has been retired from production code.
3. One-time migration of maintained in-repo sample artifacts was completed (Birmingham now uses canonical `successEchoMessage` outcome field).
4. Final focused and full regression gates are green (entries 106-107).

Residual intentional artifacts:

1. Historical plan documentation retains prior migration narrative for auditability.
2. Regression tests intentionally assert omission of legacy `optionalEchoMessage` write output to prevent contract regression.
