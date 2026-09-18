# Portrait Room Orientation - Stage 5 Simulator Handoff

Status: Complete
Stage: 5 of 5
Date: 2026-09-10
Owner Session: GPT-5.3-Codex

## Scope Completed
1. Implemented simulator active-room bounds adoption from room-change payloads:
- Uses `newRoom.roomImageCanvasWidth` and `newRoom.roomImageCanvasHeight` as active-room truth when present.
- Falls back to authored session dimensions and then default 800x600 when room-level dimensions are unavailable.
2. Separated simulator viewport size from room logical size:
- `RenderSurfaceWidth`/`RenderSurfaceHeight` remain viewport dimensions.
- New active-room dimensions drive logical room scene sizing.
3. Implemented contain/center presentation for room scene inside simulator viewport:
- Scene now scales uniformly to fit while preserving aspect ratio.
- Mixed-aspect letterbox/pillarbox behavior is accepted MVP behavior.
4. Implemented click and waypoint coordinate mapping from viewport-space to room-space:
- Uses inverse contain transform before command/waypoint coordinate emission.
- Letterbox-side clicks clamp to nearest room bounds edge.
5. Added focused simulator regression tests for Stage 5 bounds correctness and mixed-aspect transition behavior.
6. Added startup fallback hardening so active-room logical bounds adopt authored viewport dimensions when no initial room payload is present.
7. Applied Stage 2 parity fix for directional room-image composition by using layout-time rotation (`LayoutTransform`) instead of render-time rotation (`RenderTransform`) in simulator room-image layer templates to prevent portrait drift/cropping.

## Files Changed
1. Storyboard.Simulator/ViewModels/SimulatorViewModel.cs
- Added active-room logical dimensions (`ActiveRoomRenderWidth`, `ActiveRoomRenderHeight`).
- Added room-dimension hydration from `HostSessionDataEnvelope.RoomChange.NewRoom`.
- Added viewport->room inverse contain mapping for click and waypoint interactions.
2. Storyboard.Simulator/MainWindow.xaml
- Updated render preview composition to draw room scene in a centered `Viewbox` (`Stretch=Uniform`) using active-room logical dimensions.
- Kept viewport border dimensions unchanged and clipping behavior intact.
3. Storyboard.Simulator.Tests/SimulatorReplaySpeedSemanticsTests.cs
- Added tests for room-level bounds precedence over authored defaults.
- Added tests for mixed-aspect transition rebinding.
- Added tests for contain-mapped click coordinate clamping in portrait-in-landscape scenarios.
4. Storyboard.Simulator.Tests/SimulatorRoomImageTemplateTransformParityTests.cs
- Updated directional room-image transform parity guard to assert `LayoutTransform` usage for portrait-safe slot composition.

## Contract/Interface Impact
1. No schema edits.
2. No generated contract DTO edits.
3. Simulator consumes existing Stage 1 fields on host room-change payload:
- `roomImageCanvasWidth`
- `roomImageCanvasHeight`
4. Shared contracts remain unchanged in this stage.

## Validation Commands Executed
1. `dotnet test .\Storyboard.Simulator.Tests\Storyboard.Simulator.Tests.csproj --nologo --verbosity minimal --filter "FullyQualifiedName~SimulatorReplaySpeedSemanticsTests|FullyQualifiedName~SimulatorRoomImageTemplateTransformParityTests"`
- PASS (27 passed, 0 failed)
2. `dotnet test .\Storyboard.Simulator.Tests\Storyboard.Simulator.Tests.csproj --nologo --verbosity minimal`
- PASS (92 passed, 0 failed)
3. `dotnet build .\StoryboardDesigner.slnx --nologo --verbosity minimal`
- PASS (solution build succeeded)
4. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests|SchemaCodegenHardcodedDtoGuardrailsTests|SchemaEmittedContractDriftGuardrailsTests|TransportArtifactGuardrailsTests"`
- PASS (67 passed, 0 failed)
5. `dotnet test .\Storyboard.TransportCodegen.Tests\Storyboard.TransportCodegen.Tests.csproj`
- PASS (9 passed, 0 failed)
6. `dotnet test .\Storyboard.Simulator.Tests\Storyboard.Simulator.Tests.csproj --nologo --verbosity minimal --filter "FullyQualifiedName~SimulatorRoomImageTemplateTransformParityTests|FullyQualifiedName~SimulatorReplaySpeedSemanticsTests|FullyQualifiedName~SimulatorDirectionalImageAnchoringTests"`
- PASS (36 passed, 0 failed)

## Test Results
1. New room-bounds precedence test confirms room-level dimensions (600x800) override authored/session dimensions (800x600) for active-room logical bounds.
2. New mixed-aspect transition test confirms active-room bounds rebind on sequential transitions (800x600 -> 600x800 -> 800x600) without stale carryover.
3. New click-mapping test confirms viewport clicks in side letterbox regions map/clamp to room-space x bounds under contain scaling.
4. New startup fallback test confirms active-room logical bounds adopt authored dimensions before first room payload when room payload data is absent.
5. Existing simulator suites continue to pass with no Stage 5 regressions observed in focused or full simulator project runs.
6. Directional room-image rendering now mirrors Stage 2 Designer guidance for portrait-safe layout-time rotation and passes focused parity/anchoring coverage.

## Behavioral Notes
1. Active-room dimensions are now treated as renderer and interaction source-of-truth when provided by room-change payloads.
2. Viewport dimensions remain host-controlled and independent from room logical dimensions.
3. Mixed-aspect rooms render centered with contain behavior; visible empty side/top regions are expected MVP behavior.
4. Point-click and waypoint coordinates are emitted in room-space after inverse contain mapping, reducing mismatch risk between visual preview and runtime command intent.
5. Stage 4 WebPortal behavior parity for bounds-source and contain-style mapping is now reflected in simulator host behavior.

## Known Issues/Risks
1. Simulator interaction coverage now includes click/waypoint mapping under mixed aspect, but broader manual UX smoke for complex object layering in extreme non-standard dimensions remains advisable.
2. Transition visual polish beyond MVP (advanced mixed-aspect choreography aesthetics) remains deferred as planned.

## Final MVP Closure Notes
1. Stage 5 Simulator Support is complete.
2. With Stage 1-5 complete, Portrait Room Orientation MVP workstream is closed.
3. Residual items are non-blocking and deferred per plan scope (aesthetic transition polish).
