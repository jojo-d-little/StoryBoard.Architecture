# Action Variable Chooser Event Anchor Unification - Stage 04 Designer Anchor And Picker Unification Handoff

Status: Completed (implemented and validated)
Stage: 4 of 7
Date: 2026-09-12
Owner Session: GitHub Copilot (GPT-5.3-Codex)

## Opening Prompt (Use To Start This Stage)

Start Stage 04 of [plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md](plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md).
Complete only designer anchor/picker unification and three-lane discovery UX in this stage.
Do not begin Stage 05 broad action-wave rollout or Stage 07 retirement in this stage.
Honor stage boundaries from the main plan and record validation evidence.
Treat `self` as first-class anchor vocabulary (Stage 03 extension complete), not a bespoke designer special-case.
For action output authoring, require `currentAction` anchor as entry and complete from `actionOutputVariableCatalog` for the current action type.
Prefer intrinsic leaf guidance (`nameInGame`, `name`) immediately after anchor selection in quick UX paths.
Enforce explicit validation severity policy for anchor references:
1. Hard-error only when invalidity is fully knowable at design time.
2. Soft findings for dynamic tails that cannot be proven invalid at design time.
3. Soft findings must be written to validation report output on disk and must not trigger save-time popup reporting.

## Stage Boundary Allowlist Snapshot (From Main Plan)

Default deny rule:
1. Any path not explicitly listed in allowed read/edit scope is out of scope for this stage.

Edit-implies-read rule:
1. Any path in allowed edit scope is automatically readable.
2. Allowed read scope should list only extra read-only dependencies.

1. Allowed read scope:
- plans/**
- StoryboardDesigner.App/**
- StoryboardDesigner.App.Tests/**
- Storyboard.GameEngine/Config/**

2. Allowed edit scope:
- StoryboardDesigner.App/**
- StoryboardDesigner.App.Tests/**
- plans/active/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_PLAN.md
- plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_04_DESIGNER_UNIFICATION_HANDOFF.md

## Scope Completed

1. Unified designer event-filter variable assist flow into explicit three-lane discovery UX.
2. Added lane 1 for known event payload keys/source mappings.
3. Added lane 2 for action output variables authored as `currentAction::action.*` and sourced from `actionOutputVariableCatalog` for the selected binding action type.
4. Preserved lane 3 for anchor/subproperty scope-guided variable discovery and made quick path faster.
5. Treated `self` and other anchor keys as shared anchor vocabulary through session-anchor reader usage.
6. Prioritized intrinsic leaf guidance by ranking `nameInGame` and `name` first in quick variable suggestions after anchor/subproperty selection.
7. Added `self.nameInGame` to designer variable choices and script known-token support.
8. Implemented hard/soft anchor validation severity policy by knowability:
9. Hard error for malformed anchor-root syntax.
10. Hard error for unknown anchor root.
11. Hard error for unsupported subproperty only when supported-subproperty metadata is explicitly available for that anchor.
12. Soft warning for dynamic/unverifiable tails after a valid anchor root (and known-or-not-provably-invalid subproperty chain).
13. Implemented soft-finding surfacing policy so `EVT-007` findings are written to the validation report on disk but excluded from save-time popup issue lists.

## Stage 04 Direction Lock (Added 2026-09-12)

1. Anchor-first authoring is the canonical model for event filters and echo scripting guidance surfaces.
2. `self` is now a first-class runtime anchor (Stage 03 reopened extension complete) and should be treated as anchor vocabulary, not a bespoke picker special-case.
3. For action output variables, users should author via `currentAction` anchor and then select a manifest-backed output key.

Designer completion model for action outputs:
1. Entry path: choose `currentAction` anchor.
2. Completion path: suggest keys from `actionOutputVariableCatalog` for the currently edited action type.
3. Canonical authored form remains anchor-rooted (for example `currentAction::action.resultCode`), not free-floating legacy `action.*` tokens.

MVP UX defaults to accelerate common scripting intent:
1. Prefer/promote intrinsic scope properties `nameInGame` and `name` as first suggestions after anchor selection.
2. Keep quick-insert affordances for `nameInGame` and `name` in chooser UI because these dominate echo-script usage.
3. Preserve generic anchor semantics; do not imply a concrete runtime object instance at authoring time.

Validation expectations updated for intrinsic anchor usage:
1. Anchor-rooted references ending in `name` or `nameInGame` on valid scope anchors should be treated as valid by default.
2. Do not emit ambiguity or unsupported-subproperty findings for those intrinsic leaves when the anchor root is known.
3. Continue hard/soft policy from this handoff for all non-intrinsic paths.

Future (non-MVP) designer aid, explicitly deferred:
1. Optional representative-scope browsing after anchor selection to help users discover likely values/properties.
2. This is authoring assistance only; it must not alter runtime resolution semantics or persist hidden bindings.
3. Runtime contract remains anchor-path based and generic.

Stage 04 required policy outcomes (must be evidenced before stage close):
1. Hard validation classification:
- Unknown anchor root => hard error.
- Unsupported supportedSubProperties member => hard error.
- Malformed anchor path syntax => hard error.
2. Soft validation classification:
- Valid anchor root + valid supportedSubProperties chain + dynamic/unverifiable tail => soft finding.
3. Soft surfacing policy:
- Include soft findings in persisted validation report output file.
- Exclude soft findings from save-time popup triggers.

## Files Changed

1. StoryboardDesigner.App/Views/EventBindingVariableAssistDialog.xaml
2. StoryboardDesigner.App/Views/EventBindingVariableAssistDialog.xaml.cs
3. StoryboardDesigner.App/Views/EventSubscriptionEditorDialog.xaml.cs
4. StoryboardDesigner.App/Views/EventSubscriptionLibraryDialog.xaml.cs
5. StoryboardDesigner.App/ViewModels/MainWindowViewModel.ProjectExplorer.cs
6. StoryboardDesigner.App/ViewModels/MainWindowViewModel.cs
7. StoryboardDesigner.App/ViewModels/MainWindowViewModel.FileCommands.cs
8. StoryboardDesigner.App/Validation/Rules/Project/EventSubscriptionAnchorSubPropertyReferenceRule.cs
9. StoryboardDesigner.App/Validation/Rules/Project/EventSubscriptionFilterVariableSyntaxRule.cs
10. StoryboardDesigner.App/Validation/Rules/Project/EventSubscriptionAnchorDynamicTailReferenceRule.cs (new)
11. StoryboardDesigner.App/Validation/Rules/Scripting/ScriptRuleSupport.cs
12. StoryboardDesigner.App.Tests/EventSubscriptionManifestValidationRulesTests.cs
13. StoryboardDesigner.App.Tests/MainWindowViewModelVariableChoicesTests.cs
14. StoryboardDesigner.App.Tests/MainWindowViewModelValidationSaveWorkflowTests.cs
15. plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_04_DESIGNER_UNIFICATION_HANDOFF.md

## Contract/Interface Impact

1. No shared runtime contract/interface DTO changes.
2. No Stage 05 rollout or Stage 07 retirement edits were performed.

## Validation Commands Executed

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~EventSubscriptionManifestValidationRulesTests|FullyQualifiedName~MainWindowViewModelVariableChoicesTests|FullyQualifiedName~MainWindowViewModelValidationSaveWorkflowTests"`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`

## Test Results

1. Focused Stage 04 test set passed: 23 passed, 0 failed.
2. Replay smoke gate passed: 9 passed, 0 failed.
3. Build succeeded.

## Behavioral Notes

1. Event filter variable chooser now exposes three discovery lanes and supports `currentAction` output authoring from manifest-backed keys.
2. Quick guidance now prefers intrinsic leaves (`nameInGame`, `name`) immediately after anchor/subproperty selection.
3. Save-time popup reporting excludes soft `EVT-007` findings while still writing them to disk reports.
4. Validation severity now follows hard/soft classification by design-time knowability for anchor references.

## Known Issues/Risks

1. Some anchors currently have sparse subproperty metadata in manifest sources; unsupported-subproperty hard errors are emitted only when subproperty declarations are explicitly present.
2. Action-output lane currently reflects whichever action types map to the selected binding action name in nearest-scope lookup; duplicate action names at same scope distance can intentionally show combined output keys.

## Boundary Compliance Report

1. Out-of-scope reads performed:
- `ENHANCEMENT_GUIDELINES.md` (required repository policy pre-read).
- `.github/instructions/storyboard-designer-mvvm.instructions.md` (required applyTo instruction).
- `.github/instructions/storyboard-tests.instructions.md` (required applyTo instruction).
- `/memories/repo/build-notes.md` (repository memory check for prior pitfalls).
2. Out-of-scope edits performed:
- None.
3. Stage-boundary exceptions approved:
- None requested.
4. Session context scope notes:
- All code edits remained within Stage 04 allowed edit scope.

## Explicit Next-Stage Start Checklist

1. Confirm three-lane discovery parity and script validation before Stage 05.
2. Confirm hard/soft validation-policy tests are present and green.
3. Confirm save-time popup suppression for soft findings is verified.
4. Confirm `currentAction` -> `actionOutputVariableCatalog` completion is implemented for the edited action type(s).
5. Confirm intrinsic `nameInGame`/`name` guidance is prioritized in quick suggestions.
6. Confirm deferred representative-scope browsing is tracked as post-MVP and not coupled to runtime behavior.
