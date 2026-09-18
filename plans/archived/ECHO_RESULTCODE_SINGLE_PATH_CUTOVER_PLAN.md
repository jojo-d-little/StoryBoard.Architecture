# Echo ResultCode Single-Path Cutover Plan

Status: Completed
Owner: StoryboardDesigner.App authoring + serialization
Last updated: 2026-07-15

## Plan Maintenance (2026-07-15)
1. Completion status re-verified against latest full-solution gate results.
2. This plan is archive-ready with no remaining implementation tasks.

## Completion Summary
1. CommandAction echo projections were removed from the designer model surface.
2. Action editor flows were cut over to OutcomeMessageMap token reads/writes only.
3. Designer persistence/export DTOs were cut over to OutcomeMessageMap only.
4. Shared clean runtime action DTO contract was cut over to OutcomeMessageMap only.
5. Regression/build gates passed after cutover.

## Objective
Replace all dual-path echo-script handling with one canonical path:
1. Store echo scripts only in OutcomeMessageMap.
2. Key map entries only by result-code token.
3. Remove all per-result echo projection properties and legacy per-field echo DTO fields.
4. Remove all map<->projection synchronization logic.
5. Keep one editor surface in RoomActionEditorDialog: the ResultCode-driven list.

This is an intentional hard cutover with no legacy compatibility.

## Scope
1. StoryboardDesigner.App model surface for CommandAction echo scripts.
2. Designer action editing UX in RoomActionEditorDialog.
3. DTO contracts used by app persistence/export for action echo scripts.
4. Serialization load/save pipeline.
5. Tests covering model behavior, editor behavior, and serialization behavior.

## Out Of Scope
1. Runtime result-code registry semantics in Storyboard.Shared.
2. Non-echo action payload fields (flags, variables, composite ids, etc.).
3. Migration tooling for historical files.

## Current-State Problems To Eliminate
1. Duplicate UI editors for the same data:
- Top generic ResultCode list and lower per-action echo text fields.
2. Duplicate model API for same data.
3. Synchronization complexity and drift risk:
- Manual merge/sync logic in dialog save/render flow.
4. User confusion from compatibility warning text about unsupported preserved entries.

## Target End State
1. CommandAction exposes only OutcomeMessageMap for echoes.
2. All reads/writes of echo scripts go through OutcomeMessageMap.
3. RoomActionEditorDialog uses only the top ResultCode list editor for echo scripts.
4. No per-action echo textboxes in lower panels.
5. Unsupported tokens are preserved in data, flagged by validation, and shown as unsupported in the result-code editor.
6. DTOs/persistence include only OutcomeMessageMap for echo scripts.

## Hard Policy Decisions
1. No compatibility bridge:
- Remove legacy echo fields from DTOs and readers.
- Older files that only contain legacy echo fields will not be upgraded automatically.
2. Strict token policy:
- Persist supported tokens normally.
- Unsupported/unknown tokens are preserved in data, loaded into the editor map, and surfaced via validation warnings.
3. Single editor policy:
- ResultCode list editor is the only echo editing UI in RoomActionEditorDialog.
4. Default script source policy:
- Remove hardcoded default echo constants from dialog/model code.
- Add producer-managed defaults config file keyed by action type and result-code token.
- No dedicated designer UI editor for this config; provide Tools menu command that opens the JSON file in a plain text editor window.

## Locked Decisions (2026-07-15)
1. Backward compatibility for legacy per-field echo storage: removed.
2. Unsupported tokens: preserved (with validation warning), not dropped.
3. Default script source: external producer-managed JSON config (no hardcoded defaults in code).
4. Defaults file location and precedence:
- Master copy lives beside the application executable.
- File name: default echo messages.json
- Optional project override file uses the same filename in the project folder.
- If project override exists, it is primary and missing keys fall back to master.
5. Tools menu edit behavior:
- Tools menu provides two explicit entries:
- Edit Application Default Echo Messages (master file)
- Edit Project Default Echo Messages (project file)
6. File creation behavior:
- If effective defaults file does not exist, auto-create it with a populated skeleton covering all action types and supported result codes.
7. Project command availability:
- Edit Project Default Echo Messages is disabled when no project is open.
8. Skeleton completeness policy:
- Include every action type section in the generated skeleton, even when an action type currently has zero result codes.
- For zero-result-code action types (for example LinkedActions at present), generate an empty object section to avoid special-case omissions and support future expansion.
9. Unsupported token validation severity:
- Unsupported/unknown result-code entries are reported as warnings (not errors).
10. Defaults application timing:
- Apply defaults only when creating new actions or creating missing token entries.
- Do not retroactively fill blank scripts on existing actions when loading/opening.
11. Action type change behavior:
- On ActionType change, immediately normalize OutcomeMessageMap to the new action type's supported tokens.
- Tokens not supported by the new type are removed at change time.
12. Action type normalization UX:
- Do not prompt for confirmation before normalization in the action editor dialog.
- During normalization, populate the new type token set from effective defaults config.
13. Shared-token preservation on ActionType change:
- If a result-code token exists in both old and new action types, keep the existing script value.
- Apply defaults only for tokens missing after normalization.
14. Skeleton JSON shape:
- Keep defaults JSON as pure key/value data only.
- Do not include comments, metadata hints, or example-only fields.
15. Malformed defaults handling:
- If defaults JSON is malformed, show a warning and continue with empty defaults.
- Do not block opening or saving action editors.
16. Malformed defaults warning UX:
- Warning identifies which defaults file failed (application or project) and includes full path.
- Warning includes an immediate Open for Review/Edit action.
- Open action launches the same text editor flow used by Tools menu edit commands.
17. Multiple parse failures UX:
- If both defaults files are malformed, show warnings one at a time (sequential), not a combined dialog.
18. Warning trigger timing:
- Do not show malformed-default warnings when opening Action Editor dialog.
- Show warnings only when defaults are actually needed (new action/new token seeding or ActionType normalization seeding).
19. Startup validation:
- Validate both defaults files (application and project when project is open) at designer startup.
- Surface startup validation findings through the existing validation surface as standard validation rules.
- Do not use startup popup dialogs.
20. Normalization fallback behavior:
- If ActionType normalization needs defaults and effective defaults are malformed, proceed with normalization using empty-string defaults for newly seeded tokens.
21. Validation scope mapping:
- Application-defaults validation issues surface as global/project-level warnings.
- Project-defaults validation issues surface as project-scoped warnings.
22. Missing project sections policy:
- Missing action-type sections (or missing tokens) in project defaults are acceptable.
- Do not emit warnings for missing project sections/tokens because master fallback intentionally covers them.
23. Missing master sections policy:
- Missing action-type sections or missing supported tokens in master defaults produce warnings.
- Master is the baseline fallback source; gaps in master are treated as incomplete configuration.
24. Missing-file creation vs refresh distinction:
- Missing-file creation is first-time bootstrap only (pure skeleton creation when file does not exist).
- Refresh is a separate intentional operation on an existing file.
25. Existing-file refresh reconciliation policy:
- Refresh preserves existing valid entries and their values.
- Refresh adds missing action-type sections and missing supported token keys.
- Refresh removes extra/invalid action-type keys and invalid token keys.
- Refresh is non-destructive for valid entries; it is corrective for invalid/out-of-schema entries.
26. Refresh result feedback:
- After refresh, show a concise summary only.
- Summary reports count of entries added and count of entries removed.
- No verbose per-entry listing in the default UX.
27. Refresh summary presentation:
- Present refresh summary as a non-modal toast/status notification.
- Do not use a modal completion dialog.
28. Refresh command shape:
- Provide one refresh command under Tools.
- Command prompts user to choose refresh target (application defaults or project defaults).
 - Prompt has no preselected default; user must choose explicitly.
29. Refresh target missing-file behavior:
- If selected refresh target file does not exist, create it first (skeleton bootstrap), then run reconcile.
30. Refresh execution confirmation:
- Refresh runs immediately after target selection.
- No extra confirmation dialog is shown.

## Implementation Plan

### Phase 1 - Model API Simplification
1. Remove projection properties in CommandAction used for echo scripts.
2. Remove helper methods tied to projection wrappers (for example token-specific getter/setter wrappers that only back these properties).
3. Introduce small explicit helpers for dictionary-only access where needed:
- GetOutcomeScript(token)
- SetOutcomeScript(token, script)
- NormalizeOutcomeMapForActionType(actionType, map)
4. Keep script-field provider behavior by enumerating OutcomeMessageMap entries only.

### Phase 2 - Dialog UI Single Surface
1. In RoomActionEditorDialog.xaml:
- Remove all echo TextBox rows from lower action panels.
- Keep only top Outcome Echoes (ResultCode-driven) section.
2. In RoomActionEditorDialog.xaml.cs:
- Remove BuildOutcomeEchoMapForEditor merge from projection fields.
- Remove SyncWorkingCopyScriptFieldsFromOutcomeMap projection write-back.
- Remove EnsureNavigateDirectionEchoDefaults and all hardcoded per-field default injectors.
- Load defaults from defaults config service when creating new/empty map entries for supported result codes.
- On ActionType changes in dialog working copy, normalize immediately with defaults seeding for the new type.
- Do not show a normalization confirmation prompt.
- Preserve scripts for shared tokens across old/new action types; seed defaults only for newly introduced missing tokens.
- Remove EditEchoScriptField_OnClick branches that target per-field echo controls.
- Validate echo scripts by iterating current ResultCode entries only.
3. Preserve action-specific non-echo panel fields (ids, modes, booleans, etc.).

### Phase 2b - Producer Defaults Config
1. Add defaults JSON file locations:
- Master: <app-executable-folder>/default echo messages.json
- Optional project override: <project-folder>/default echo messages.json
- Top-level object keyed by action type name.
- Each action type contains object keyed by supported result-code token.
- Action types with no registered result codes still appear with an empty object.
- No comment or helper/example properties are generated.
2. Add precedence/merge behavior for effective defaults:
- If only master exists: use master.
- If only project file exists: use project.
- If both exist: use project values first, then fallback to master for missing action type/result-code keys.
3. Add a defaults loader service in StoryboardDesigner.App:
- Validates action type keys.
- Validates result-code token keys against RuntimeActionResultCodeRegistry.
- Logs/ignores invalid keys without crashing editor workflows.
- On malformed JSON parse, surfaces warning and returns empty defaults set.
- Parse failure payload includes source kind (application/project) and full path for warning UX.
- Treat missing project sections/tokens as normal fallback cases, not validation failures.
4. Add startup defaults validation workflow:
- At designer startup, validate application defaults file and project defaults file (when available).
- Cache parse/validation results for later defaults-resolution calls.
- Do not block startup on validation failures.
- Emit validation rule entries (warning severity) for malformed JSON, invalid action-type keys, and invalid result-code token keys.
- Scope emitted validation entries by source: application defaults -> global/project level; project defaults -> project-scoped.
- Emit warning entries for missing master action-type sections/tokens required by current runtime result-code registry.
5. Apply defaults only during new-action/new-entry creation paths.
6. Do not apply defaults retroactively when loading/opening existing actions.
7. During ActionType normalization seeding, if defaults are malformed/unavailable, seed missing tokens with empty strings and continue.
8. Add two Tools menu commands:
- Edit Application Default Echo Messages.
- Edit Project Default Echo Messages.
9. Command behavior:
- Application command opens master file directly.
- Project command opens project file directly.
- Project command is disabled when no project is open.
- If target file does not exist, auto-create populated skeleton JSON before opening.
- Each command opens its target JSON in text editor window.
- No form-based editor, no schema builder UI.
- Malformed-default warning Open action must reuse this same file-open editor path.
10. Optional refresh command behavior:
- "Refresh Default Echo Messages Skeleton" is a single Tools command.
- Command asks which target file to refresh (application or project).
- Project target option is unavailable when no project is open.
- After selection, command ensures target file exists (create if missing), then reconciles target defaults file against current action/result-code registry.
- Command executes immediately after target selection (no additional confirmation prompt).
- Preserve existing valid values.
- Add missing sections/tokens.
- Remove invalid/extra sections/tokens.
- Treat this as an intentional command, separate from missing-file auto-creation.
- Show brief completion summary: X entries added, Y entries removed.

### Phase 3 - Serialization Contract Cutover
1. Remove legacy per-echo fields from DTOs:
- CommandActionExportDto in Serialization/RoomExportDto.cs
- JsonExportService.CommandActionDto internal class
2. Keep only OutcomeMessageMap for echo serialization.
3. Remove all reads/writes that hydrate model echo projections from legacy fields.
4. On load:
- Read OutcomeMessageMap only.
- Normalize canonical token casing for known supported tokens.
- Preserve unsupported/unknown tokens in map.
- Emit validation warning entries for unsupported tokens.
5. On save:
- Persist normalized OutcomeMessageMap only.

### Phase 4 - Service/Formatter Cleanup
1. ActionEchoEditorEntryBuilder:
- Keep supported entries deterministic by descriptor order.
- Append unsupported preserved entries with explicit unsupported marker.
2. Keep unsupported warning UX plumbing, but update wording to point users to validation/issues and cleanup workflow.
3. ActionOutcomeMessageStatusFormatter:
- Continue reporting unsupported summary where present.
- Distinguish supported defaults vs explicit scripts when defaults config supplies fallback text.

### Phase 5 - Call-Site Refactor
1. Replace projection-property usages across viewmodels/services/tests with OutcomeMessageMap token access.
2. Remove action-specific echo copy blocks in cloning/mapping code.
3. Ensure all action constructors/factories initialize OutcomeMessageMap via normalized token set where required.

### Phase 6 - Test Rewrite And Guardrails
1. Update/remove tests that assert projection property behavior.
2. Add tests that assert dictionary-only invariants:
- No projection echo properties exist on CommandAction.
- RoomActionEditorDialog exposes one echo editing surface.
- Save path writes only OutcomeMessageMap.
- Loader ignores legacy echo fields and does not backfill from them.
- Unsupported tokens are preserved and surfaced as warnings.
3. Add regression tests for action-type token normalization during action type change.
4. Add tests asserting that action-type changes purge old-type-only tokens, preserve shared-token scripts, seed missing new-type tokens from effective defaults, and do not show confirmation prompts.

## File-Level Worklist (Primary)
1. StoryboardDesigner.App/Models/Actions/CommandAction.PayloadProjections.cs
2. StoryboardDesigner.App/Views/RoomActionEditorDialog.xaml
3. StoryboardDesigner.App/Views/RoomActionEditorDialog.xaml.cs
4. StoryboardDesigner.App/Services/ActionEchoEditorEntryBuilder.cs
5. StoryboardDesigner.App/Services/ActionEchoEditorDialogStatePresenter.cs
6. StoryboardDesigner.App/Services/UnsupportedOutcomeEntryWarningTextFormatter.cs
7. StoryboardDesigner.App/Services/ActionOutcomeMessageStatusFormatter.cs
8. StoryboardDesigner.App/Serialization/RoomExportDto.cs
9. StoryboardDesigner.App/Services/JsonExportService.cs
10. StoryboardDesigner.App/Config/** (new defaults JSON + loader/writer support)
11. StoryboardDesigner.App/MainWindow and Tools menu command wiring
12. StoryboardDesigner.App/**/*.cs call sites using removed projection echo properties
13. StoryboardDesigner.App.Tests/**/* affected tests

## Breaking Changes
1. Historical files that only use legacy per-field echo keys are unsupported after cutover.
2. Any external tool expecting legacy echo fields in serialized output will break.
3. Defaults behavior moves from hardcoded code constants to external config file content.

## Validation Plan
1. Between every phase boundary, run non-smoke validation:
- dotnet build .\StoryboardDesigner.slnx
- dotnet test .\Storyboard.Shared.Tests\Storyboard.Shared.Tests.csproj
- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
- dotnet test .\Storyboard.Simulator.Tests\Storyboard.Simulator.Tests.csproj
2. At final completion, run full test validation including smoke tests:
- dotnet test .\StoryboardDesigner.slnx

## Acceptance Criteria
1. There is exactly one echo editing UX in RoomActionEditorDialog: ResultCode list editor.
2. CommandAction has no echo projection properties; only OutcomeMessageMap is used.
3. No map<->projection sync methods remain.
4. Unsupported result-code entries are visible as unsupported and produce validation warnings.
5. DTOs and serializers contain no legacy per-echo fields.
6. All tests pass with updated expectations.

## Execution Order Recommendation
1. Phase 1 (model API) plus compile fixes.
2. Phase 2 (dialog UI/code-behind) plus focused UI tests.
3. Phase 3 (serialization) plus fixture updates.
4. Phase 4 and 5 (cleanup + call sites).
5. Phase 6 (test hardening + full validation).
6. Enforce phase gates: after each phase, execute the non-smoke validation set before proceeding.
7. Execute full solution test run (including smoke tests) once at the end.

## Risks
1. Large compile blast radius due to many call sites.
2. Test churn where assertions currently rely on projection properties.
3. Loss of backward file compatibility by design.

## Risk Mitigations
1. Implement in small commits per phase with full compile after each phase.
2. Add temporary compile-time TODO markers only within phase branches, not merged mainline.
3. Keep strict phase gates with non-smoke test suite between phases and full suite at end.
