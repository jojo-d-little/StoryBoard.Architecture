# Room Display Name And Transition Presentation - Stage 2 Designer Authoring UX Handoff

Status: Complete
Stage: 2 of 7
Date: 2026-09-11
Owner Session: GitHub Copilot (GPT-5.3-Codex)

## Opening Prompt (Use To Start This Stage)

Start Stage 2 of plans/active/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_PLAN.md.
Read Stage 1 handoff first and complete only Stage 2 (Designer Authoring UX).
Do not begin Stage 3.
Honor Stage 2 boundary allowlists from the main plan; do not edit outside Stage 2 edit scope.
Update this handoff with files changed, validation results, behavioral notes, and explicit next-stage checklist.

## Stage Boundary Allowlist Snapshot (From Main Plan)

Default deny rule:
1. Any path not explicitly listed in allowed read/edit scope is out of scope for this stage.

Edit-implies-read rule:
1. Any path in allowed edit scope is automatically readable.
2. Allowed read scope lists only extra read-only dependencies.

1. Allowed read scope:
- plans/**
- Storyboard.Shared.Contracts/**
2. Allowed edit scope:
- StoryboardDesigner.App/**
- StoryboardDesigner.App.Tests/**
- StoryboardDesigner.App.SmokeTests/**
- plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_02_DESIGNER_HANDOFF.md

## Compatibility Mode Declaration

1. Stage 2 compatibility mode: N/A (consumer adaptation stage).

## Scope Completed

1. Added Room Settings authoring support for room `nameInGame` as an optional producer-entered value.
2. Wired room `nameInGame` through designer room workflows:
- edit room settings for placed rooms,
- edit room settings for room templates,
- add-new-room bootstrap (including template-seeded defaults).
3. Added traversal authoring support for optional `presentationEffectKey` in the existing non-wizard traversal editor dialog using a dropdown of catalog-backed `RoomTransition` cues (Stage 2 requirement: keep traversal wizard stable).
4. Propagated traversal `presentationEffectKey` through create/update traversal ViewModel paths so authored values persist and export via existing Stage 1 contract mapping.
5. Added traversal review visibility for transition effect key in the room traversal review grid.
6. Added focused tests for new room alias authoring behavior and traversal presentation effect persistence/normalization.

## Files Changed

1. StoryboardDesigner.App/Services/RoomSettingsEditRequest.cs
2. StoryboardDesigner.App/Services/PresentationEffectsCatalogService.cs
3. StoryboardDesigner.App/Views/RoomSettingsDialog.xaml
4. StoryboardDesigner.App/Views/RoomSettingsDialog.xaml.cs
5. StoryboardDesigner.App/Views/TraversalEditorDialog.xaml
6. StoryboardDesigner.App/Views/TraversalEditorDialog.xaml.cs
7. StoryboardDesigner.App/ViewModels/MainWindowViewModel.cs
8. StoryboardDesigner.App/ViewModels/MainWindowViewModel.ProjectExplorer.cs
9. StoryboardDesigner.App/ViewModels/MainWindowViewModel.FileCommands.cs
10. StoryboardDesigner.App/Views/Controls/AreaMapCanvas.xaml.cs
11. StoryboardDesigner.App/Views/Controls/AreaRoomPlacementCard.xaml.cs
12. StoryboardDesigner.App/Views/RoomTraversalsDialog.xaml
13. StoryboardDesigner.App/Views/RoomTraversalsDialog.xaml.cs
14. StoryboardDesigner.App.Tests/PromoteToRoomTemplateWorkflowTests.cs
15. StoryboardDesigner.App.Tests/MainWindowViewModelAreaMapTraversalSyncTests.cs
16. StoryboardDesigner.App.Tests/MainWindowViewModelRoomSettingsTests.cs
17. plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_02_DESIGNER_HANDOFF.md

## Contract/Interface Impact

1. No shared schema or shared runtime contract files were edited in Stage 2.
2. Designer now consumes Stage 1 additive contract fields in authoring UX:
- room alias via `nameInGame`,
- traversal transition override via `presentationEffectKey`.
3. Internal designer API changes only:
- `RoomSettingsEditRequest` now carries `NameInGame` (defaulted for compatibility),
- `MainWindowViewModel.TryUpdateSelectedAreaTraversal(...)` now accepts `presentationEffectKey`,
- `MainWindowViewModel.TryCreateSelectedAreaTraversal(...)` now accepts `presentationEffectKey`.

## Validation Commands Executed

1. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "ArchitectureSeparationGuardrailsTests|GameSimulatorPlaybackRegressionTests|GameManagerTests"
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~MainWindowViewModelAreaMapTraversalSyncTests|FullyQualifiedName~PromoteToRoomTemplateWorkflowTests|FullyQualifiedName~MainWindowViewModelRoomSettingsTests"

## Test Results

1. Architecture/regression filtered suite: PASS (59/59).
2. Stage 2 focused authoring suite: PASS (28/28).
3. Final Stage 2 closeout rerun after the RoomTransition dropdown update: PASS (59/59 and 28/28).
4. During implementation, one intermediate failing run occurred due to temporary malformed `TraversalEditorDialog.xaml`; issue was corrected and rerun passed.

## Behavioral Notes

1. Room settings dialog now has a dedicated "Name In Game" field; blank keeps existing fallback behavior to room `Name`.
2. Room edits and room-template edits now persist `NameInGame` changes on the corresponding `Room` model.
3. Add-new-room flow now seeds the room settings request with template `NameInGame` when bootstrapping from a room template.
4. Traversal editor now exposes optional transition effect key selection as a dropdown sourced from configured `RoomTransition` cues, with a `(Default)` option and preservation of previously-authored unlisted keys.
5. Room traversal review dialog now displays a "Transition Effect" column, showing `(Default)` when no override key is authored.

## Known Issues/Risks

1. Traversal transition effect key selection is constrained to catalog-listed `RoomTransition` entries in normal authoring flow; previously-authored legacy/unlisted keys are preserved as selectable values to avoid destructive edits.
2. Runtime host/web consumption and fallback resolution remain downstream work (Stages 3-5), so authored overrides are not fully end-to-end visible until those stages complete.

## Boundary Compliance Report

1. Out-of-scope reads performed:
- ENHANCEMENT_GUIDELINES.md (required repository entry-point policy read).
- .github/instructions/storyboard-designer-mvvm.instructions.md (applies to StoryboardDesigner.App edits).
- .github/instructions/storyboard-tests.instructions.md (applies to StoryboardDesigner.App.Tests edits).
2. Out-of-scope edits performed:
- None.
3. Stage-boundary exceptions approved:
- None.
4. Session context scope notes:
- Stage 2 implementation remained inside allowed edit scope; no contract/shared runtime edits were made.

## Explicit Next-Stage Start Checklist

1. Read this Stage 2 handoff and Stage 1 handoff before starting Stage 3.
2. Treat designer-authored traversal `presentationEffectKey` as optional input that may be null/blank and already normalized by Stage 2.
3. Preserve room display-name fallback semantics: use `nameInGame` when set, otherwise canonical room `name`.
4. In Stage 3 runtime integration, carry traversal effect selection forward without introducing host/designer boundary coupling.
5. Keep Stage 3 edits constrained to that stage allowlist before touching host/web portal work.
