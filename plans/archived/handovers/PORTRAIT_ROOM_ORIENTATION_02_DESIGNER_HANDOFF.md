# Portrait Room Orientation - Stage 2 Designer Handoff

Status: Complete
Stage: 2 of 5
Date: 2026-09-09
Owner Session: GPT-5.3-Codex

## Scope Completed
- Implemented room-level width/height as Designer room source-of-truth and removed Designer persistence/runtime mapping use of room-level cell-size override.
- Reworked room settings UX for orientation presets plus explicit width/height editing.
- Updated room and template room settings workflows, new-room creation flow, and room editor tab initialization/refresh to use room-effective dimensions.
- Updated authoring/load/save and runtime-export mappings so rooms persist explicit width/height and legacy rooms default deterministically from project defaults then write back explicit values on save.
- Updated room-object preview bounds validation to evaluate against room-effective bounds.
- Added a project-level room dimension validation rule with locked limits and divisibility constraints.
- Preserved save policy as non-blocking and ensured runtime export/game run remain blocked by validation errors.
- Added and updated focused regression tests for Stage 2 behavior.
- Fixed Designer preview rotation regression where selected directional image previews (including Down/floor images) were not honoring configured slot rotation in all preview surfaces.

## Files Changed
1. StoryboardDesigner.App/Models/Rooms/Room.cs
- Added room-level `RoomImageCanvasWidth` and `RoomImageCanvasHeight` model properties with safe defaults/normalization.
- Removed `RoomGridCellSizeOverride` from Designer room model.

2. StoryboardDesigner.App/Services/RoomSettingsEditRequest.cs
- Replaced room-grid-override payload with explicit room width/height fields.

3. StoryboardDesigner.App/Views/RoomSettingsDialog.xaml
- Replaced room grid override controls with orientation preset selector and width/height editors.

4. StoryboardDesigner.App/Views/RoomSettingsDialog.xaml.cs
- Implemented preset-driven and custom width/height editing UX.
- Added immediate validation guidance for positive values, divisibility, and 4..200 derived grid limits.

5. StoryboardDesigner.App/ViewModels/MainWindowViewModel.cs
- Updated room editor open/refresh to use room-effective width/height.
- Updated effective grid cell-size resolver to remain project-global.

6. StoryboardDesigner.App/ViewModels/MainWindowViewModel.ProjectExplorer.cs
- Updated room and template room settings workflows to read/write room width/height.
- Updated new room creation to seed explicit room width/height from project defaults.
- Updated traversal wizard door placement to consume source/destination room-effective dimensions.

7. StoryboardDesigner.App/ViewModels/MainWindowViewModel.FileCommands.cs
- Registered new room canvas validation rule.
- Kept save non-blocking and aligned export blocking policy to error-level issues.

8. StoryboardDesigner.App/Services/JsonExportService.cs
- Updated room authoring and room-template mapping to persist `roomImageCanvasWidth`/`roomImageCanvasHeight`.
- Removed room-level cell-size override mapping from Designer persistence paths.
- Added deterministic load-time fallback to project defaults when room-level dimensions are missing.
- Ensured explicit width/height are emitted on subsequent saves.

9. StoryboardDesigner.App/Validation/Rules/Objects/RoomObjectPreviewCoordinateBoundsRule.cs
- Switched bounds source to room-effective dimensions (fallback to project defaults only when room dimensions are absent).

10. StoryboardDesigner.App/Validation/Rules/Project/RoomCanvasDimensionsRule.cs
- Added blocking project validation for room dimensions:
	- positive width/height
	- divisible by project grid cell size
	- derived rows/columns constrained to 4..200

11. StoryboardDesigner.App.Tests/RoomObjectPreviewCoordinateBoundsRuleTests.cs
- Updated expectations and setup to room-bound coordinates.

12. StoryboardDesigner.App.Tests/RoomCanvasDimensionsRuleTests.cs
- Added focused rule tests for valid room sizes, divisibility failures, and range-limit failures.

13. StoryboardDesigner.App.Tests/JsonExportServiceObjectPlacementPersistenceTests.cs
- Added legacy-room-dimension fallback + write-back regression coverage.

14. StoryboardDesigner.App.Tests/MainWindowViewModelValidationSaveWorkflowTests.cs
- Added export gating regression test verifying warnings-only validation does not block runtime export.

15. StoryboardDesigner.App/Features/RoomDesigner/Views/RoomDesignerWorkspaceView.xaml
- Restored selected-direction thumbnail preview to native image orientation (no preview rotation transform).

16. StoryboardDesigner.App/Features/RoomDesigner/Controls/RoomDesignerIndependentPreviewControl.xaml
- Fixed independent preview mode to apply selected direction rotation transform.

17. StoryboardDesigner.App/Features/RoomDesigner/Controls/RoomDesignerOverlayPreviewControl.xaml
- Fixed the main room designer surface independent-mode image layer to apply selected direction rotation transform.
- Updated composite directional overlay rendering to use layout-time rotation for directional slot images.

## Directional Overlay Rendering Solution (Carry-Forward)
Problem signature observed during Stage 2:
1. Directional overlays looked offset/cropped after introducing portrait room dimensions and broader slot rotation usage.
2. Symptoms included missing/partially off-canvas walls/floor and apparent "oversized canvas" blanks that were actually placement drift.

Root cause in Designer WPF surface:
1. Composite directional slot images were rotated with `RenderTransform` while placement was anchored/aligned in pre-rotation layout coordinates.
2. With mixed source image dimensions and quarter-turn rotations, render-time rotation produced drift/clipping relative to slot anchors.

Final fix pattern used in Designer (recommended for Simulator parity work):
1. Keep room canvas sizing bound to effective room dimensions.
2. Keep slot alignment and offsets directional/explicit.
3. Apply directional image rotation in the composite overlay with `LayoutTransform` (not `RenderTransform`) so rotated bounds participate in layout before clipping/anchor resolution.
4. Preserve independent/selected preview rotation bindings, but keep thumbnail preview native for authoring clarity.

Why this matters for later Simulator/WebPortal work:
1. Any host that composes directional room layers with WPF-like or layout-separated transforms can reproduce the same drift when using render-time rotation.
2. Parity checks should verify each slot under 0/90/180/270 rotations across both 800x600 and 600x800 rooms.
3. If similar drift appears, first validate whether rotation is applied at layout-time vs render-time in that host pipeline.

## Contract/Interface Impact
1. No schema or generated contract files were hand-edited in Stage 2.
2. Designer implementation now aligns to Stage 1 contract shape:
	- room-level `roomImageCanvasWidth`
	- room-level `roomImageCanvasHeight`
	- no Designer-side mapping dependency on room-level `roomGridCellSizeOverride`.
3. Project-level `roomImageCanvasWidth` and `roomImageCanvasHeight` remain defaults for new rooms, not active-room truth once room values are explicit.

## Validation Commands Executed
1. `dotnet build .\\StoryboardDesigner.slnx`
	- PASS
2. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj`
	- FAIL (pre-existing/unrelated failures outside Stage 2 scope; Stage 2-focused tests listed below passed)
3. `dotnet test .\\Storyboard.TransportCodegen.Tests\\Storyboard.TransportCodegen.Tests.csproj`
	- PASS
4. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`
	- PASS
5. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "SchemaCodegenHardcodedDtoGuardrailsTests|SchemaEmittedContractDriftGuardrailsTests|ArchitectureSeparationGuardrailsTests|TransportArtifactGuardrailsTests"`
	- PASS

Additional focused Stage 2 verification:
6. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~TraversalWizardApplyPipelineTests.RunTraversalWizardForRoom_UsesSelectedDoorTemplate_WhenProvided|FullyQualifiedName~TreeScopedValidationContextActionsTests.ExecuteTreeContextAction_ValidateNodeOnly_FiltersToNodeScope|FullyQualifiedName~RoomCanvasDimensionsRuleTests"`
	- PASS
7. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~MainWindowViewModelValidationSaveWorkflowTests.TryExportCleanProject_WithValidationErrors_BlocksRuntimeExport|FullyQualifiedName~MainWindowViewModelValidationSaveWorkflowTests.TryExportCleanProject_WithWarningsOnly_AllowsRuntimeExport|FullyQualifiedName~RoomObjectPreviewCoordinateBoundsRuleTests|FullyQualifiedName~RoomCanvasDimensionsRuleTests|FullyQualifiedName~JsonExportServiceObjectPlacementPersistenceTests.TryLoadProjectModel_DefaultsMissingRoomCanvasDimensions_FromProjectDefaults_AndWritesBackOnSave"`
	- PASS

## Test Results
1. Stage 2-focused regression tests for room settings, mapping/defaulting/write-back, room bounds validation source, and export/run gating behavior pass.
2. Full `StoryboardDesigner.App.Tests` suite currently includes pre-existing failures outside Stage 2 scope (examples observed: quantifiable rename propagation expectations, starter template parity duplication, and unrelated import/global tests).
3. Required contract/interface and transport guardrail gates pass.

## Behavioral Notes (Locked Expectations)
1. Orientation remains derived from room width/height and is not separately persisted in runtime contracts.
2. Save is allowed with dimension violations, but run/export must be blocked until validation passes.
3. Legacy room data without explicit room width/height must default deterministically from project defaults and write back explicit values on next save.
4. Project-level room width/height remain creation defaults, not active-room truth once room-level values are explicit.
5. Room object preview coordinate validation now evaluates against room-effective bounds.
6. Traversal wizard door placement uses source/destination room-effective dimensions, avoiding accidental project-default leakage.
7. Directional room-image rotation now renders consistently in Designer independent preview and the main room designer surface image layer; composite overlay slots use layout-time rotation to avoid anchor drift under quarter-turn rotations. This is relevant parity context for Stage 4 (WebPortal) and Stage 5 (Simulator) rendering review.

## Known Issues/Risks
1. Full designer test suite has unrelated pre-existing failures; Stage 2 downstream confidence relies on focused pass set plus guardrail gate until those baseline failures are resolved.
2. Runtime aliases for `roomGridCellSizeOverride` remain in GameEngine/Simulator compatibility paths and are expected to be handled in Stage 3/5 cleanup decisions.
3. Any future room-dimension rule strictness changes must preserve the save-non-blocking contract and avoid accidentally blocking authoring save.
4. Cross-host verification risk: directional rotation behavior should be explicitly validated in Stage 4/5 to ensure Simulator/WebPortal match Designer expectations for floor/ceiling (Down/Up) image rotations and composite-slot anchor stability under 0/90/180/270 rotations.

## Next-Stage Start Checklist (GameEngine)
1. Confirm Designer now emits explicit room-level width/height for rooms and room templates.
2. Confirm legacy room defaulting + save write-back behavior is in place and tested.
3. Confirm room-level cell-size override usage is removed from Designer persistence/runtime mapping paths.
4. Confirm validation policy behavior (save allowed, run/export blocked) is implemented and covered by tests.
5. Confirm room editor/preview bounds now reflect effective room dimensions.
6. In GameEngine stage, replace remaining runtime compatibility reads of `roomGridCellSizeOverride` only via explicit Stage 3 decisions, not opportunistic edits.
