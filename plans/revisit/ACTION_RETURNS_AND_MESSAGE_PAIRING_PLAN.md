# Action Returns and Message Pairing Plan

Status: Active (A0-A3 complete; A4 signoff pending UI automation coverage; A5 convergence pending)
Owner: Storyboard.Shared runtime actions + StoryboardDesigner.App action authoring UX
Last updated: 2026-07-08

## 1. Purpose

Standardize action outcomes so every action type exposes a deterministic result-code set and supports outcome echo scripts keyed by result code.

## 2. Why This Plan Exists

Current action outcome messaging is inconsistent across action types.

Symptoms:

1. Some action types rely on SuccessEchoMessage/FailureEchoMessage only.
2. Some action types embed additional outcome-specific echo fields.
3. Runtime-emitted result-code semantics are not centrally discoverable for designer UX.
4. Drift risk exists between executor behavior and any external registry/docs.

## 3. Goals

1. Make result codes a first-class, discoverable runtime contract per action type.
2. Ensure all actions support baseline outcome messaging for Success and Failure.
3. Enable optional action-specific result-code variants (additional success/failure flavors).
4. Let designer dialogs dynamically render editable echo scripts per supported result code.
5. Eliminate registry drift through compile-time and regression guardrails.

## 4. Non-Goals (Phase 1)

1. No broad executor architecture rewrite.
2. No immediate removal of all legacy message fields in one change.
3. No unversioned contract break in clean export/runtime DTOs.
4. No designer-runtime coupling that bypasses Shared contracts.

## 5. Core Design Direction

## 5.1 Result Code Model

1. Each action type has a strongly-typed internal enum for valid result codes.
2. All action-type enums explicitly include baseline result codes as the first two members:
   1. Success
   2. Failure
3. Action types may define additional action-specific result codes.

## 5.2 Token Stability Contract

1. Runtime/storage-facing result codes remain stable string tokens.
2. Enum member names are internal safety; token strings are external compatibility surface.
3. A mapping layer controls enum -> token to prevent accidental breaking changes from enum renames.

## 5.3 Registry Shape

Canonical registry key/value:

1. Key: CommandActionType.
2. Value:
   1. Enum type (or typed descriptor set) for supported result codes.
   2. Per-code metadata:
      1. Stable token
      2. Outcome kind (Success/Failure)
      3. UX label/description (optional)
      4. Built-in fallback echo (optional)

Placement:

1. Source of truth in Storyboard.Shared, executor-adjacent (actions layer).
2. RuntimeCommandActionExecutor may expose a passthrough API for discoverability.

## 5.4 Message Resolution Contract

For an action execution outcome with ResultCode R:

1. If OutcomeMessageMap contains R, use that script.
2. Else if outcome is success, fallback to Success script.
3. Else fallback to Failure script.
4. Else optional built-in default echo.

Persistence visibility rule:

1. For each action, all supported result codes are persisted in project JSON for OutcomeMessageMap.
2. If a supported code has no authored script, persist it with an explicit empty/blank value.
3. This is intentional to make unconfigured messaging opportunities visible to producers and reviewers.

## 6. Drift Prevention Guardrails

1. Executors emit result codes via enum-backed helper only (no ad-hoc string literals).
2. Registry generation/registration for each action type is required by tests.
3. Tests assert emitted result codes are a subset of declared supported codes.
4. Tests assert supported codes include baseline Success/Failure.
5. Designer validation blocks unsupported result-code message entries.

## 7. Architecture Boundaries

1. Shared owns action type, result code sets, token mapping, and runtime resolution rules.
2. Designer consumes shared metadata to drive dynamic per-code message editing UX.
3. Simulator continues to consume runtime contracts from Shared only.
4. No compile-time dependency from Shared to Designer.

## 8. Migration Strategy

## Phase A0: Decision Lock

1. Lock baseline universal result codes and naming conventions.
2. Lock token stability policy and versioning rules.
3. Lock where ActionType -> ResultCode metadata lives.

Exit criteria:

1. Decision record entries approved.

## Phase A1: Registry and Typed Result Code Foundation

1. Introduce action-result-code registry contract in Shared actions layer.
2. Add first action-type enum(s) and stable token mapping.
3. Add RuntimeCommandActionExecutor passthrough for metadata discovery.

Exit criteria:

1. Registry available for all in-scope action types.

## Phase A2: Executor Emission Alignment

1. Replace executor raw literals with typed result-code emission helpers.
2. Preserve existing behavior and token outputs.

Exit criteria:

1. Runtime parity tests pass with no output regressions.

## Phase A3: OutcomeMessageMap Introduction

1. Add per-action OutcomeMessageMap keyed by stable result-code token.
2. Keep legacy SuccessEchoMessage/FailureEchoMessage compatibility fallback.
3. Add resolver implementing the fallback chain.

Exit criteria:

1. Result-code-specific scripts resolve deterministically.

## Phase A4: Designer Dynamic Authoring UX

1. Action editor loads supported result codes from Shared registry.
2. Dialog renders script entry rows per supported result code.
3. Baseline Success/Failure rows always present.

Exit criteria:

1. UX supports dynamic per-action-type message configuration without hardcoded field lists.

## Phase A5: Cleanup and Convergence

1. Retire legacy per-action ad-hoc echo fields where superseded.
2. Keep explicit compatibility adapter where needed.
3. Update docs and snapshots.

Exit criteria:

1. Single consistent outcome-message authoring and runtime path.

## 9. Validation Gates

Default:

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj

Focused runtime gate:

1. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"

New required tests (minimum):

1. Registry coverage test: every supported CommandActionType declares result codes.
2. Executor drift test: emitted result codes are declared for that action type.
3. Resolver precedence test: exact-code script > Success/Failure fallback > built-in fallback.
4. Designer metadata test: per-action code list drives editor rows correctly.

## 10. Risks and Mitigations

1. Risk: token churn breaks existing data/scripts.
   Mitigation: explicit enum-to-token mapping and compatibility policy.
2. Risk: duplicate legacy and new message surfaces during migration.
   Mitigation: deterministic fallback precedence and phased retirement.
3. Risk: runtime/designer drift in supported codes.
   Mitigation: single Shared registry + guardrail tests.

## 11. Locked Decisions (A0)

Status key:

1. Locked: approved for implementation in this plan.

| Decision ID | Decision | Status |
| --- | --- | --- |
| ARM-01 | Every action-type result-code enum must explicitly define `Success` and `Failure` as its first two members; registry does not inject baseline codes. | Locked (2026-07-05) |
| ARM-02 | OutcomeMessageMap is introduced on RuntimeCommandActionDescriptor as the canonical runtime mapping keyed by stable result-code token; payload-specific legacy echo fields remain as compatibility fallbacks during transition. Designer UX lock: detailed per-result-code script editing is handled in one reusable Action Echo Editor dialog, while main action dialogs show compact per-code status (defined/undefined) plus optional single-line script hint. | Locked (2026-07-05) |
| ARM-03 | Stable result-code tokens use PascalCase for this phase to align with existing runtime result tokens and minimize migration churn. Runtime and designer matching must be case-insensitive (case-neutral reads/lookups), while persistence/export normalizes to canonical PascalCase tokens. | Locked (2026-07-05) |
| ARM-04 | Imported unsupported result-code entries are preserved as inert entries and surfaced as deterministic warnings; they are never executed at runtime unless supported by the action-type registry. For supported result codes, OutcomeMessageMap persists explicit entries even when script is blank, so unconfigured messages are discoverable in project JSON. | Locked (2026-07-05) |

### 11.1 Lock Rationale Notes

1. ARM-01 removes hidden special-case logic by requiring every action enum to carry the same explicit baseline contract (`Success`, `Failure`) before action-specific codes.
2. ARM-02 establishes one canonical target shape now, while allowing safe phased migration from existing payload-specific fields and keeping action dialogs compact through a shared echo-editor workflow.
3. ARM-03 avoids immediate token migration blast radius across tests, fixtures, and diagnostics while keeping implementations forgiving through case-insensitive matching and canonical write normalization.
4. ARM-04 prevents silent data loss, supports forward compatibility when newer producers/hosts introduce additional codes, and improves discoverability by persisting blank entries for unconfigured supported result codes.

## 12. Immediate Next Step

1. Complete A4 signoff by adding direct dialog-level UI regression coverage for result-code row rendering/edit interactions and inline status/hint updates.
2. Close A4 checklist items by marking implemented behavior as Closed where already delivered and keeping only truly pending items open.
3. Start A5 convergence pass to retire superseded legacy echo-field pathways and document any compatibility adapters that remain intentional.

## 14. A1 Pilot Delta (2026-07-05)

Completed:

1. Added Shared action-result-code registry contracts and executor passthrough lookup APIs.
2. Added explicit baseline result-code enums (`Success`, `Failure`) for all currently executor-supported action types and registered them in the Shared registry.
3. Added NavigateDirection pilot result-code enum with explicit baseline `Success`/`Failure` and stable PascalCase token mapping.
4. Replaced NavigateDirection executor string literals with enum-backed result-code emission.
5. Added guardrail tests for:
   1. baseline enum order (`Success`, `Failure` first),
   2. case-insensitive token lookup,
   3. emitted-token subset declaration coverage,
   4. registry coverage for all currently executor-supported action types.

Validation evidence:

1. `dotnet build .\\StoryboardDesigner.slnx` passed.
2. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "RuntimeActionResultCodeRegistryTests|GameCommandProcessorFixtureTests"` passed (6/6).
3. `GameSimulatorPlaybackRegressionTests` currently shows existing fixture drift unrelated to this pilot slice and remains tracked separately.

## 15. A3 Starter Delta (2026-07-05)

Completed:

1. Added additive Shared contract property `OutcomeMessageMap` on runtime action descriptors (and interface) keyed by stable result-code token.
2. Updated runtime outcome script resolution to support deterministic precedence:
   1. exact `OutcomeMessageMap[resultCodeToken]` match (case-insensitive),
   2. baseline `OutcomeMessageMap[Success|Failure]` fallback,
   3. typed payload success/failure fields,
   4. legacy `SuccessEchoMessage` / `FailureEchoMessage`.
3. Preserved ARM-04 behavior for explicit blank entries: if a map key exists with blank script, no deeper fallback is applied.
4. Added/updated regression tests to verify exact-token precedence, case-insensitive lookup, baseline fallback, and blank-entry suppression.
5. Added Shared `RuntimeOutcomeMessageMapBuilder` and wired runtime descriptor mapping paths to populate deterministic maps for:
   1. designer in-memory runtime mapping (`ProjectModelRuntimeSnapshotMapper`),
   2. clean-export bootstrap mapping (`CleanRuntimeBootstrapSnapshotMapper`).
6. Mapper-populated maps now seed explicit blank entries for all supported tokens and set effective baseline success/failure scripts plus NavigateDirection-specific token scripts.
7. Added persistence wiring for action `OutcomeMessageMap` across project and clean-export action DTOs, with canonical supported-token normalization and explicit blank supported entries on write.
8. Added project save/load regression coverage proving canonical supported token persistence (`Success`/`Failure`), blank-entry retention, and unsupported token preservation.
9. Added clean-export contract coverage proving room clean JSON emits `outcomeMessageMap` with canonical supported tokens, explicit blank supported entries, and preserved unsupported entries.
10. Added clean-bootstrap runtime mapper coverage proving imported supported tokens are canonicalized, supported blanks are retained, and unsupported entries remain present as inert data in runtime descriptor maps.
11. Added runtime command-processing fixture coverage proving unsupported `OutcomeMessageMap` tokens remain inert during execution and are not selected unless a supported emitted token resolves.
12. Added runtime command-processing fixture coverage proving a supported emitted token (`Success`) from `OutcomeMessageMap` overrides legacy success echo fallback during execution.
13. Added runtime command-processing fixture coverage proving a supported emitted token (`Success`) with explicit blank script in `OutcomeMessageMap` suppresses legacy success echo fallback output.
14. Added A4 starter: registry-driven per-action defined/undefined status metadata in `RoomActionEditorDialog` using Shared supported result-code sets, with compact status text + tooltip breakdown.
15. Added `ActionOutcomeMessageStatusFormatter` service and tests so status evaluation is deterministic, case-normalized, and honors explicit blank entries.
16. Added reusable `ActionEchoEditorDialog` with dynamic per-result-code rows sourced from Shared registry metadata, including supported/unsupported visibility and per-code script editing via `TextOutputScriptEditorDialog`.
17. Wired main action dialog outcome buttons to launch `ActionEchoEditorDialog` and persist edited `OutcomeMessageMap` entries back to the working/source action models.
18. Added `ActionEchoEditorEntryBuilder` + tests for canonical supported-token normalization, unsupported entry preservation, and blank-entry round-trip behavior.
19. Added reusable dialog UX polish: unsupported-entry warning banner (ARM-04 visibility) and per-row support metadata (`Supported` vs `Unsupported (inert at runtime)`) with explanatory tooltips.
20. Added deterministic unsupported-entry warning summary in `ActionOutcomeMessageStatusFormatter.BuildTooltipText`, so compact-status tooltip surfaces preserved inert entries from `OutcomeMessageMap`.
21. Added `ActionEchoEditorDialogStatePresenter` as a testable state layer for dialog selection-dependent button enablement and unsupported-warning visibility/text, then refactored dialog code-behind to consume presenter output.
22. Added presenter regression tests covering no-selection baseline, selected-entry button state, and pluralized unsupported-warning text.
23. Centralized unsupported-entry warning phrasing in `UnsupportedOutcomeEntryWarningTextFormatter` and switched both dialog banner and status tooltip summary generation to consume this shared formatter.
24. Added dedicated formatter tests to lock singular/plural wording and empty-string behavior for non-positive counts.
25. Added `ActionOutcomeMessageTextFormatter` to centralize compact status text wording and no-result-codes wording, and refactored `ActionOutcomeMessageStatusFormatter` to consume it.
26. Added dedicated compact-status formatter tests for baseline output, clamped counts, and non-positive total fallback behavior.
27. Completed targeted rereview/fix for `RuntimeBuildCompositeByPartsResultCode`: expanded typed enum members (`IncompleteRecipe`, `SpecifyTarget`, `TargetMismatch`, `NoMatchingRecipe`), switched registry metadata to typed code set mapping, and aligned BuildCompositeByParts executor failure branches to emit matching result-code tokens.
28. Added registry coverage proving BuildCompositeByParts typed tokens are declared and descriptor lookup remains case-insensitive.
29. Applied readability refactor: moved action result-code enums out of `RuntimeActionStandardResultCodes.cs` into `Storyboard.Shared/GameServices/Actions/GameActions/RuntimeActionResultCodeEnums/` with one enum per file, and moved `RuntimeBuildCompositeByPartsResultCodes` mapping into that folder.
30. Completed typed result-code review for `BuildCompositeByTarget` and `BreakCompositeItem`: added typed enum members + token mappers, updated registry to use typed code sets, and aligned executor branch outcomes to emit specific result-code tokens.
31. Added registry guardrail tests proving `BuildCompositeByTarget` and `BreakCompositeItem` typed tokens are declared and case-insensitive lookup resolves expected descriptors.
32. Applied readability consolidation for typed result-code pairs: merged each enum + token mapper back into a single file for `BuildCompositeByParts`, `BuildCompositeByTarget`, and `BreakCompositeItem` under `RuntimeActionResultCodeEnums/`.
33. Completed typed result-code review for `PutObjectInContainer` and `RemoveObjectFromContainer`: added typed enum members + token mappers, updated registry to use typed code sets, and aligned executor outcomes to emit specific result-code tokens for resolve-failed vs operation-failed branches.
34. Added registry guardrail tests proving container action typed tokens are declared and case-insensitive descriptor lookup resolves expected tokens.
35. Completed follow-up typed result-code review for `SetFlag`: added `SetFailed` typed token metadata, switched registry to typed code set, and aligned runtime SetFlag failure/success outcomes to emit explicit result-code tokens.
36. Added registry guardrail tests proving SetFlag typed token declaration and case-insensitive descriptor lookup.
37. Improved `RoomActionEditorDialog` inline outcome authoring affordance by adding an explicit `Edit Mapped Echoes...` action that opens the reusable per-result-code editor without requiring Success/Failure preselection.
38. Enhanced compact status presentation in `RoomActionEditorDialog` with clearer mapped-echo phrasing, richer tooltip guidance, and synchronized button hints that reflect current defined/undefined counts.
39. Expanded `ActionOutcomeMessageStatusFormatterTests` coverage for typed tokens, including `SetFlag` compact-status counts and `CheckGameProperty` typed-token tooltip entries.
40. Extracted mapped-echo inline wording into `ActionOutcomeMessageInlineHintFormatter` and wired `RoomActionEditorDialog` status/button tooltip updates through this formatter for deterministic UI text behavior.
41. Added `ActionOutcomeMessageInlineHintFormatterTests` to lock inline status text, tooltip guidance, button tooltip wording, and null-input tolerance.
42. Refined reusable `ActionEchoEditorDialog` UX to a consistent two-column result-code editing surface for all action types: column one shows result-code identity and outcome metadata, column two shows a read-only single-line script preview (first line).
43. Added click-to-edit behavior directly on the read-only script preview cell so selecting typical result codes and launching script editor is one click, while preserving unsupported-entry warning visibility.
44. Simplified `RoomActionEditorDialog` entry affordance to a single `Edit Outcome Echoes...` action to emphasize the result-code-driven workflow over legacy success/failure-specific buttons.
45. Expanded `ActionEchoEditorEntryBuilderTests` with script-preview coverage to lock first-line extraction/trimming semantics used by the new list UI.
46. Moved the result-code echo editing surface directly into `RoomActionEditorDialog` and removed button-gated launch for this flow: the room-action dialog now renders an inline two-column result-code list with click-to-edit script previews.
47. Replaced compact outcome-status preview usage in the room-action header with inline result-code editing and synchronized unsupported-entry warning display in-place.
48. Applied explicit ResultCode-only outcome-message cutover (legacy fallback retirement): runtime outcome script resolution no longer falls back to legacy `SuccessEchoMessage`/`FailureEchoMessage` or payload-specific success/failure echo fields when `OutcomeMessageMap` entries are absent.
49. Updated runtime snapshot/bootstrap mappers to seed supported tokens with blank defaults and then merge persisted `OutcomeMessageMap` entries as the sole authored message source for both supported and unsupported tokens.
50. Updated designer status formatter and focused regression tests to reflect ResultCode-only semantics (no legacy-derived defined status/output inference).

Validation evidence:

1. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "RuntimeActionPayloadAccessorsTests|RuntimeActionResultCodeRegistryTests|GameCommandProcessorFixtureTests"` passed (41/41).
2. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "RuntimeOutcomeMessageMapBuilderTests|RuntimeActionPayloadAccessorsTests|RuntimeActionResultCodeRegistryTests|GameCommandProcessorFixtureTests"` passed (44/44).
3. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "JsonExportServiceProjectStateTests|RuntimeOutcomeMessageMapBuilderTests|RuntimeActionPayloadAccessorsTests|RuntimeActionResultCodeRegistryTests|GameCommandProcessorFixtureTests"` passed (64/64).
4. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "JsonExportServiceCleanExportTests|JsonExportServiceProjectStateTests|RuntimeOutcomeMessageMapBuilderTests|RuntimeActionPayloadAccessorsTests|RuntimeActionResultCodeRegistryTests|GameCommandProcessorFixtureTests"` passed (69/69).
5. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "QuantifiableRuntimeMaterializationTests|JsonExportServiceCleanExportTests|JsonExportServiceProjectStateTests|RuntimeOutcomeMessageMapBuilderTests|RuntimeActionPayloadAccessorsTests|RuntimeActionResultCodeRegistryTests|GameCommandProcessorFixtureTests"` passed (80/80).
6. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "GameCommandProcessorFixtureTests|QuantifiableRuntimeMaterializationTests|JsonExportServiceCleanExportTests|JsonExportServiceProjectStateTests|RuntimeOutcomeMessageMapBuilderTests|RuntimeActionPayloadAccessorsTests|RuntimeActionResultCodeRegistryTests"` passed (81/81).
7. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "GameCommandProcessorFixtureTests|QuantifiableRuntimeMaterializationTests|JsonExportServiceCleanExportTests|JsonExportServiceProjectStateTests|RuntimeOutcomeMessageMapBuilderTests|RuntimeActionPayloadAccessorsTests|RuntimeActionResultCodeRegistryTests"` passed (82/82).
8. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "GameCommandProcessorFixtureTests|QuantifiableRuntimeMaterializationTests|JsonExportServiceCleanExportTests|JsonExportServiceProjectStateTests|RuntimeOutcomeMessageMapBuilderTests|RuntimeActionPayloadAccessorsTests|RuntimeActionResultCodeRegistryTests"` passed (83/83).
9. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "ActionOutcomeMessageStatusFormatterTests|GameCommandProcessorFixtureTests|RuntimeOutcomeMessageMapBuilderTests|RuntimeActionPayloadAccessorsTests|RuntimeActionResultCodeRegistryTests"` passed (49/49).
10. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "ActionEchoEditorEntryBuilderTests|ActionOutcomeMessageStatusFormatterTests|GameCommandProcessorFixtureTests|RuntimeActionResultCodeRegistryTests"` passed (13/13).
11. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "ActionEchoEditorEntryBuilderTests|ActionOutcomeMessageStatusFormatterTests"` passed (4/4).
12. `dotnet build .\\StoryboardDesigner.slnx` passed.
13. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "ActionEchoEditorEntryBuilderTests"` passed (3/3).
14. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "ActionEchoEditorEntryBuilderTests|ActionOutcomeMessageStatusFormatterTests|GameCommandProcessorFixtureTests|RuntimeOutcomeMessageMapBuilderTests|RuntimeActionPayloadAccessorsTests|RuntimeActionResultCodeRegistryTests"` passed (52/52).
15. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "ActionOutcomeMessageStatusFormatterTests|ActionEchoEditorEntryBuilderTests"` passed (6/6).
16. `dotnet build .\\StoryboardDesigner.slnx` passed.
17. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "ActionEchoEditorDialogStatePresenterTests|ActionEchoEditorEntryBuilderTests|ActionOutcomeMessageStatusFormatterTests"` passed (9/9).
18. `dotnet build .\\StoryboardDesigner.slnx` passed.
19. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "UnsupportedOutcomeEntryWarningTextFormatterTests|ActionEchoEditorDialogStatePresenterTests|ActionOutcomeMessageStatusFormatterTests|ActionEchoEditorEntryBuilderTests"` passed (13/13).
20. `dotnet build .\\StoryboardDesigner.slnx` passed.
21. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "ActionOutcomeMessageTextFormatterTests|ActionOutcomeMessageStatusFormatterTests|UnsupportedOutcomeEntryWarningTextFormatterTests|ActionEchoEditorDialogStatePresenterTests|ActionEchoEditorEntryBuilderTests"` passed (17/17).
22. `dotnet build .\\StoryboardDesigner.slnx` passed.
23. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "RuntimeActionResultCodeRegistryTests|CompositeBuildActionTests|GameCommandProcessorFixtureTests"` passed (44/44).
24. `dotnet build .\\StoryboardDesigner.slnx` passed.
25. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "RuntimeActionResultCodeRegistryTests|GameCommandProcessorFixtureTests|CompositeBuildActionTests"` passed (44/44).
26. `dotnet build .\\StoryboardDesigner.slnx` passed.
27. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "RuntimeActionResultCodeRegistryTests|CompositeBuildActionTests|GameCommandProcessorFixtureTests"` passed (48/48).
28. `dotnet build .\\StoryboardDesigner.slnx` passed.
29. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "RuntimeActionResultCodeRegistryTests|CompositeBuildActionTests|GameCommandProcessorFixtureTests"` passed (48/48).
30. `dotnet build .\\StoryboardDesigner.slnx` passed.
31. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "RuntimeActionResultCodeRegistryTests|GameCommandProcessorFixtureTests|CompositeBuildActionTests"` passed (52/52).
32. `dotnet build .\\StoryboardDesigner.slnx` passed.
33. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "RuntimeActionResultCodeRegistryTests|GameCommandProcessorFixtureTests"` passed (25/25).
34. `dotnet build .\\StoryboardDesigner.slnx` passed.
35. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "ActionOutcomeMessageStatusFormatterTests|RuntimeActionResultCodeRegistryTests|GameCommandProcessorFixtureTests"` passed (30/30).
36. `dotnet build .\\StoryboardDesigner.slnx` passed.
37. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "ActionOutcomeMessageInlineHintFormatterTests|ActionOutcomeMessageStatusFormatterTests|RuntimeActionResultCodeRegistryTests|GameCommandProcessorFixtureTests"` passed (32/32).
38. `dotnet build .\\StoryboardDesigner.slnx` passed.
39. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "ActionEchoEditorEntryBuilderTests|ActionOutcomeMessageInlineHintFormatterTests|ActionOutcomeMessageStatusFormatterTests|RuntimeActionResultCodeRegistryTests|GameCommandProcessorFixtureTests"` passed (37/37).
40. `dotnet build .\\StoryboardDesigner.slnx` passed.
41. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --no-build --filter "ActionEchoEditorEntryBuilderTests|ActionOutcomeMessageStatusFormatterTests|ActionOutcomeMessageInlineHintFormatterTests|RuntimeActionResultCodeRegistryTests|GameCommandProcessorFixtureTests"` passed (37/37).
42. `dotnet build .\\StoryboardDesigner.slnx` passed.
43. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "ActionOutcomeMessageStatusFormatterTests|GameCommandProcessorFixtureTests|RuntimeActionResultCodeRegistryTests|GameManagerTests"` passed (33/33).
44. `dotnet build .\\StoryboardDesigner.slnx` passed.

## 13. A1 Implementation Checklist (File-by-File)

1. Shared registry contracts:
   1. Add action-result-code descriptor and registry types in `Storyboard.Shared/GameServices/Actions/`.
   2. Add NavigateDirection pilot enum + token mapping in `Storyboard.Shared/GameServices/Actions/`.
2. Executor discoverability:
   1. Add static passthrough methods on `RuntimeCommandActionExecutor` for supported result-code lookup.
3. Pilot executor alignment:
   1. Replace NavigateDirection string literals with enum-backed result-code usage in `NavigateDirectionExecutableAction`.
4. Guardrail tests:
   1. Add tests proving NavigateDirection registry declarations include explicit `Success`/`Failure` baseline and expected action-specific tokens.
   2. Add tests proving case-insensitive token lookup and emitted-token subset alignment for the pilot action type.
5. Validation:
   1. `dotnet build .\StoryboardDesigner.slnx`
   2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "RuntimeActionResultCodeRegistryTests|GameCommandProcessorFixtureTests|GameSimulatorPlaybackRegressionTests"`

## 16. A3/A4 Review Signoff (2026-07-05)

### 16.1 Phase A3 Signoff

Status: Closed (runtime + persistence + regression coverage in place)

Checklist:

1. `OutcomeMessageMap` exists on runtime action contract and descriptor: Closed.
2. Resolver precedence implemented (`exact token -> baseline Success/Failure -> typed payload -> legacy`): Closed.
3. Case-insensitive reads with canonical supported-token persistence normalization: Closed.
4. Supported blank entries persisted and honored at runtime (suppressing fallback): Closed.
5. Unsupported imported entries preserved as inert data and not executed unless supported token emitted: Closed.
6. Project JSON + clean-export JSON persistence coverage present: Closed.
7. Runtime bootstrap + project runtime mapper normalization/merge coverage present: Closed.
8. Runtime command-processing behavior coverage for unsupported inert and supported override paths: Closed.

Residual notes:

1. None for ARM-04 warning surfacing; deterministic unsupported-entry tooltip warning is now implemented and covered by formatter tests.

### 16.2 Phase A4 Signoff

Status: In progress for locked UX target

Checklist:

1. Action editor loads supported result codes from Shared registry: In progress.
   Update (2026-07-05): Dynamic supported code loading now implemented in reusable `ActionEchoEditorDialog`; remaining work is deeper UX integration (main-dialog inline affordances and richer status hints).
2. Reusable Action Echo Editor dialog renders per-code rows dynamically: In progress.
   Update (2026-07-05): Implemented with per-code rows, edit/clear interactions, unsupported-entry warning banner, and per-row support hints/tooltips; remaining follow-up is dialog-level UI interaction regression coverage.
3. Main action dialog shows compact per-code defined/undefined status + hint: In progress.
   Update (2026-07-05): Compact status + tooltip breakdown wired to Shared metadata, synchronized with dialog edits, includes unsupported preserved-entry warning summary, and now surfaces stronger inline affordances (`Edit Mapped Echoes...`) plus explicit mapped-echo guidance/hints.
4. Designer metadata test proving registry-driven row generation: In progress.
   Update (2026-07-05): Service-level tests added for row generation/normalization, dialog-state presenter behavior, and inline mapped-echo status/hint wording contract; direct WPF dialog UI automation tests remain pending.

Recommended next implementation slice:

1. Introduce a Shared-driven view model contract for per-action supported codes and wire a reusable Action Echo Editor dialog in Designer, then add UI-level regression tests for defined/undefined status rendering.

## 17. What Is Left and How To Resume

This plan remains active by design. Use this section as the source of truth for remaining work and restart steps.

### 17.1 What Is Left (Open)

1. Complete A4 signoff by adding direct dialog-level UI regression tests for inline result-code editing behavior.
2. Add/confirm UI-level coverage for these interaction paths:
   1. row selection and click-to-edit script preview,
   2. defined/undefined status refresh after edit and clear operations,
   3. unsupported-entry warning visibility and wording,
   4. outcome-hint text refresh after map changes.
3. Reconcile A4 checklist wording so implemented items are marked Closed and only true residuals remain In progress.
4. Begin A5 convergence by documenting which legacy echo-field pathways are still intentionally retained versus ready for retirement.
5. Add focused regression coverage for any A5 cleanup changes before broad validation.

### 17.2 How To Resume (Next Session Checklist)

1. Read Section 16.2 and this Section 17 first; treat them as the active execution board.
2. Start with tests first for the missing A4 UI interactions, then implement minimal code changes needed to make them pass.
3. Keep Shared/runtime contracts as source of truth; avoid introducing Designer-only result-code logic that bypasses Shared metadata.
4. After A4 tests are green, update Section 16.2 checklist statuses in this plan in the same change.
5. Open a small A5 slice only after A4 is closed, and scope it to one legacy-path cleanup target at a time.

### 17.3 Resume Validation Runbook

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "ActionEchoEditorEntryBuilderTests|ActionOutcomeMessageInlineHintFormatterTests|ActionOutcomeMessageStatusFormatterTests|RuntimeActionResultCodeRegistryTests|GameCommandProcessorFixtureTests"`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`
4. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`

### 17.4 Ready-To-Archive Criteria (Not Yet)

1. Section 16.2 A4 checklist items are all Closed with date-stamped updates.
2. A5 convergence work is completed or explicitly deferred with rationale and no ambiguous "in progress" notes.
3. Validation runbook in Section 17.3 is green.
4. Plan status line is updated from Active to Completed.
