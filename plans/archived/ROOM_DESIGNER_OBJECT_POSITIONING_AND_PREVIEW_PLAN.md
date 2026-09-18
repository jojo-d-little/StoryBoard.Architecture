# Room Designer Object Positioning And Preview Plan

Status: Closed (Archived)
Owner: StoryboardDesigner.App room authoring workflow + rendering surface UX
Last updated: 2026-07-11

## Closeout Summary (2026-07-11)

1. Implementation slices P0 through P5 are complete.
2. Full validation is green: build succeeds and automated tests pass.
3. Deep UI smoke remains opt-in by environment variable and no longer reports as non-runnable in default suite runs.
4. Plan is closed and moved to archived.

## 1. Purpose

Introduce authoring-time visual placement for room-contained game objects directly in the room designer surface.

1. Room child objects can be rendered as positioned visual items in room designer preview.
2. Producers can drag-and-drop child objects to set X/Y position.
3. Position is instance-owned data on the room-contained object (not template-only metadata).
4. Producers can include/exclude specific room child objects from room preview rendering.

## 2. Why This Plan Exists

Object image visibility now exists conceptually, but room designer still lacks an intuitive placement workflow.

Symptoms:

1. Position editing by manual numeric X/Y is possible but not primary-friendly.
2. Producers cannot directly stage room object layout visually.
3. No dedicated room-level UX exists to include/exclude object visuals from preview.
4. Existing room designer interaction parity is incomplete versus map/ring exclusion-style affordances.

## 3. Goals

1. Make room child object position authorable via direct manipulation (drag-and-drop) in room designer.
2. Keep numeric X/Y editing available as secondary/manual precision path.
3. Add per-object preview inclusion toggle usable from room designer UI.
4. Ensure object visibility/position rendering is constrained to immediate room children only.
5. Preserve architecture boundaries: shared runtime contracts in Shared, host UX in Designer.

## 4. Non-Goals (Phase 1)

1. No positioning for grandchildren/nested descendants.
2. No runtime physics, collision, snapping, or z-depth simulation.
3. No auto-layout algorithms.
4. No rework of simulator host interaction model in this phase.
5. No template inheritance semantics added to position data in this phase.

## 5. Core Design Direction

### 5.1 Position Ownership Model

1. Use flat object-instance fields `PositionX` and `PositionY` on room-contained object instances.
2. Position is persisted with authored room/object data and round-trips deterministically.
3. Position is meaningful only when object parent scope is Room.
4. Legacy/default load behavior is `PositionX = 0`, `PositionY = 0`, `IncludeInPreview = true`.
5. Non-room scopes may hold default/unused values but must not be rendered by room designer.

### 5.2 Render Eligibility Rules

A room object visual is rendered in room designer only when all are true:

1. Object is an immediate child of the current room.
2. Object has image/visibility data sufficient for preview display.
3. Object is marked included in room preview.
4. Object is not filtered out by current editor mode/tool state.
5. Room object visuals render above room background imagery.
6. Among room objects, draw order is explicit and controlled by right-side list ordering (top row draws on top).

### 5.3 Primary Interaction Model

1. Click object icon on room designer surface to select.
2. Drag selected object to new position.
3. Only objects currently included in preview are selectable/draggable on canvas.
4. Drag interaction is live-preview only; persisted `PositionX`/`PositionY` commit on drag release.
5. Clamp full object thumbnail bounds so the entire object stays visible inside room content bounds.
6. While actively dragging, selected object is temporarily lifted to top visual priority, then returns to explicit list order on release.
7. Undo/redo uses one undo unit per completed drag gesture.
8. Keyboard nudging is supported: arrow keys move 1px, `Shift+Arrow` moves 10px.
9. Manual numeric editing remains a secondary precision path.

### 5.4 Preview Include/Exclude UX Direction (AUI)

Locked direction:

1. Room designer adds a right-side object panel next to the preview surface.
2. Panel lists immediate room children only (no grandchildren).
3. Each row shows object name, small thumbnail, and `Include In Preview` checkbox.
4. Each row provides an `Edit Image` action that opens the existing object image properties dialog.
5. Checkbox controls whether the object is rendered on the room preview surface.
6. Hidden objects remain in hierarchy/data; they are not deleted and still appear in non-preview editors.
7. Include/exclude is edited in the right-side room-designer panel only (no mirrored inspector toggle in phase 1).
8. Panel provides `Move Up` / `Move Down` actions; row order is the authoritative object draw order.

## 6. Lockoff Questions (Must Resolve Before Full Implementation)

1. RDP-01: Data schema placement for position:
   1. Confirm existing object-instance DTO/model target fields and naming (`PositionX`, `PositionY` vs nested struct).
   2. Confirm default values and migration behavior for legacy projects.
2. RDP-02: Coordinate system definition:
   1. Confirm origin point (top-left expected).
   2. Confirm units (designer pixels vs logical units).
   3. Confirm whether coordinates are relative to room canvas content bounds.
3. RDP-03: Bounds policy:
   1. Allow drag outside room bounds or clamp?
   2. If clamped, define clamping to icon bounds vs icon anchor point.
4. RDP-04: Drag commit semantics:
   1. Live update while dragging vs commit-on-drop only.
   2. Undo stack granularity (single entry per drag session expected).
5. RDP-05: Layering/overlap policy:
   1. Confirm draw order for overlapping objects.
   2. Confirm selected-object visual priority behavior.
6. RDP-06: Include/exclude control location and precedence:
   1. Lock primary control location to right-side room object panel.
   2. Decide whether inspector retains mirrored toggle in phase 1 or phase 2.
   3. Confirm whether future room-level "hide all objects" view option should override object flags.
7. RDP-07: Scope enforcement:
   1. Confirm strict immediate-child-only rule for render and drag targets.
   2. Confirm grandchildren stay non-rendered even if they have image and position values.
8. RDP-08: Serialization/export implications:
   1. Confirm native authoring persistence for position + preview include flag.
   2. Confirm clean export contract impact and versioning decision (if emitted externally).
9. RDP-09: Validation behavior:
   1. Determine validation warnings/errors for missing image, NaN/out-of-range coordinates, or conflicting settings.
   2. Confirm when room child object position should be ignored vs flagged.
10. RDP-10: Backward compatibility + migration:
   1. Existing projects without position/include fields load with deterministic defaults.
   2. Save path should not churn unrelated JSON ordering/shape.

## 6.1 Lock Decisions (Finalized 2026-07-11)

1. DQ-01: Persist position as flat fields `PositionX` and `PositionY`.
2. DQ-02: Legacy/default load values are `PositionX = 0`, `PositionY = 0`, `IncludeInPreview = true`.
3. DQ-03: Coordinate system is top-left origin using pixel units.
4. DQ-04: Coordinates are relative to inner room content bounds.
5. DQ-05: Drag bounds clamp full object thumbnail bounds inside room content bounds.
6. DQ-06: Drag is live visual preview; persistence commit happens on drag release.
7. DQ-07: Undo granularity is one undo unit per completed drag gesture.
8. DQ-08: Draw order is explicit and user-controlled by right-side list order.
9. DQ-09: Active drag temporarily lifts selected object to top; fixed order is restored on release.
10. DQ-10: Include/exclude editing is room-designer panel only in phase 1.
11. DQ-11: Right-side panel rows include thumbnail, object name, include checkbox, move up/down controls, and image-edit entry point.
12. DQ-12: Excluded objects are hidden and non-interactive on canvas, but remain editable from the right-side list.
13. DQ-13: Keyboard nudging is enabled: arrow = 1px, `Shift+Arrow` = 10px.
14. DQ-14: Validation uses warnings, not save-blocking errors; normalize coordinates where possible; missing image means non-renderable.
15. DQ-15: Staged export strategy:
   1. Phase 1 persists native authoring fields only.
   2. Later phase adds clean-export fields for image + position + scale + rotation once churn stabilizes.
16. DQ-16: Required completion gate is build + targeted automated tests + playback smoke + manual testing signoff.
17. DQ-17: `Edit Image` from room-designer panel reuses the existing image-properties dialog/viewmodel path and refreshes thumbnail/preview immediately on close.

## 7. Implementation Slices

### 7.0 Phase Grouping Summary

1. Total phases: 3
2. Total slices: 6
3. Phase 1 Foundations: P0, P1
4. Phase 2 UX And Interaction: P2, P3, P4
5. Phase 3 Hardening And Closeout: P5

### Slice P0: Lockoff + Contract Decisions

Tasks:

1. Record and publish lock decisions from DQ-01..DQ-17.
2. Finalize data contract notes for phase-1 native persistence and phase-2 clean-export expansion.
3. Define implementation-ready UX acceptance criteria for drag, ordering, include toggles, and image-edit entry point.

Exit criteria:

1. All lock decisions recorded.
2. No unresolved schema or interaction ambiguity remains.

### Slice P1: Model + Serialization Foundation

Tasks:

1. Add position + preview include fields to object-instance model/DTOs.
2. Add deterministic serialization round-trip coverage.
3. Add migration/defaulting for legacy project content.

Exit criteria:

1. Projects load/save position and include state safely.
2. Legacy projects continue to load without errors.

### Slice P2: Room Designer Rendering Surface Integration

Tasks:

1. Render immediate room child object visuals on room canvas layer.
2. Apply render eligibility rules (image presence + include flag + scope).
3. Add selected-state visual treatment and hit-test targeting.

Exit criteria:

1. Producers can see eligible child objects in room designer.
2. Grandchildren are never rendered in room designer object layer.

### Slice P3: Drag-and-Drop Authoring Workflow

Tasks:

1. Implement pointer drag interaction for selected room child object visuals.
2. Update X/Y during or at drag-complete per lock decision.
3. Integrate undo/redo transaction behavior.
4. Keep manual X/Y edit controls as secondary path.

Exit criteria:

1. Dragging object on canvas updates persisted position data.
2. Undo/redo restores prior position accurately.

### Slice P4: Include/Exclude UX + Layer Management

Tasks:

1. Implement right-side room-object panel with per-row include checkbox, thumbnail, object name, and `Edit Image` action.
2. Implement `Move Up` / `Move Down` in panel and bind order directly to object draw ordering (top row draws on top).
3. Add quick filtering for hidden/visible entries.
4. Ensure toggling visibility does not mutate position/state except render inclusion.

Exit criteria:

1. Producers can quickly include/exclude room child objects from preview.
2. Producers can reorder room objects in panel and observe deterministic draw-order updates.
3. Producers can edit object image settings from room designer panel without navigating away.
4. Hidden objects remain editable through hierarchy/right-side panel paths.

### Slice P5: Validation + Regression Hardening

Tasks:

1. Add validation rules for illegal coordinate values and invalid scope use.
2. Add tests for scope restrictions, serialization defaults, drag behavior, and toggle behavior.
3. Add architecture guardrails where shared/runtime boundaries are touched.

Exit criteria:

1. Feature is covered by deterministic tests.
2. No boundary regressions introduced.

## 8. Test And Validation Gates

Between-slice gate (required before moving to next slice):

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`

Per-slice minimum gate:

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "Room|Object|Validation|Serialization"`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`

Manual signoff gate (required before feature complete):

1. Verify right-side list interactions: include checkbox, `Edit Image`, move up/down.
2. Verify drag/drop behavior, clamping, and temporary drag-top visual.
3. Verify keyboard nudging and undo granularity.
4. Verify excluded-object hidden/non-interactive canvas behavior.

Runtime-boundary gate (when Shared contracts/managers are touched):

1. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`

## 9. Initial Acceptance Checklist

1. In room designer, immediate room child objects with enabled preview and valid image render visibly.
2. Dragging a rendered object updates its persisted position.
3. Manual X/Y edit remains available and syncs with canvas placement.
4. Per-object include/exclude toggle hides/shows object in room preview without deleting object.
5. Grandchildren never render as draggable room-layer visuals.
6. Load/save round-trip preserves position and include/exclude state.
7. Existing project files without new fields open safely with defaults.
8. Each room-designer object row can open image properties dialog and persist image setting changes.
9. Room object ordering controls set deterministic draw order among room objects while keeping all room objects above room imagery.

## 10. Notes For Follow-Up

1. Optional future slice: snap-to-grid and alignment guides.
2. Optional future slice: multi-select drag for room child objects.
3. Optional future slice: z-order controls for overlapping visuals.
