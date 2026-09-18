# Room Designer + Appearance Scaling Checkpoint Plan

## Purpose
Capture current state of the room-preview and appearance-dialog scaling effort so work can resume quickly and safely.

## Current Status (2026-07-23)

### Completed
1. Unified preview sizing semantics around native image pixels:
- Scale `1.0` now means native image pixel size before local rotation/offset adjustments.

2. Room preview sizing/placement updates:
- Native image dimensions are used as the icon base size.
- Scale is applied through rendered width/height instead of transform-scale layering.
- Centering offsets now use rendered dimensions (post-scale) for placement math.

3. Appearance dialog sizing updates:
- Guide cell size is now tied to project room grid cell size (instead of fixed 220/7 assumptions).
- Image scale can be edited directly by typing and takes effect live.
- Nudge controls support finer granularity (`0.01` step) and lower floor (`0.01`).
- Scale is applied through width/height (matching room preview), with rotation/offset remaining as transforms.

4. Contract plumbing:
- Project room grid cell size is threaded through object edit request paths into appearance editing.

5. Regression coverage updates:
- Focused preview workflow tests updated for native-size semantics and rendered-size centering behavior.

### Verified
1. `dotnet build .\StoryboardDesigner.slnx` passes.
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~RoomDesignerTabViewModelPreviewWorkflowTests"` passes.

## Known User Validation Focus
User reports visual behavior is improved and room preview looks correct; additional hands-on review is pending and considered critical before expanding scope.

## Resume Checklist (Next Session)
1. Re-run targeted visual checks in appearance dialog and room preview using real assets:
- Brass key (~800x800) at scale `0.1` should approximate 2x2 cells at 40px cell size.
- Tall door asset (~228px dimension path under test) should appear near 5.7 cells at scale `1.0` when expected axis aligns.

2. Confirm no apparent clipping/partial render artifacts in appearance dialog at small scales.

3. Compare orientation cases (N/E/S/W) for door-like assets to ensure consistent alignment and perceived footprint fit.

4. If anomalies remain, capture exact repro tuple:
- object
- source image dimensions
- scale
- footprint WxH
- orientation
- expected vs actual visual

## Guardrails
1. Keep Designer-only logic in `StoryboardDesigner.App`; do not introduce runtime host coupling.
2. Preserve current native-pixel scale semantics unless explicitly redefined.
3. Prefer minimal deltas and keep focused tests green while iterating.
