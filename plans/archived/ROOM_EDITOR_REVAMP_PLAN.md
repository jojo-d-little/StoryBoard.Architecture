# Room Editor Revamp Plan

Status: Active (Discovery + Planning)
Owner: StoryboardDesigner.App authoring UX
Last updated: 2026-07-10

## 0. Locked Product Decision (2026-07-10)

The current center-panel concept of "Default Room View" is replaced by "Room Display Preview".

Decision details:

1. Remove the concept of authoring or assigning a default room image.
2. Keep and strengthen the center panel as a preview surface only.
3. The preview surface must render directional room images, including overlay-mode behavior.
4. Image definition/assignment moves to directional view configuration, not the center preview panel.

Initial acceptance criteria:

1. No "default image" field or workflow remains in the room editor UX.
2. Center panel label, semantics, and documentation use "Room Display Preview" consistently.
3. Selecting directional view sources updates preview output deterministically.
4. Overlay-mode directional assets are previewable from the same center surface.
5. Existing preview value is preserved while authoring responsibility is separated.

The room display mode setting ("Independent" vs "Overlay") becomes room-global.

Decision details:

1. Remove per-direction mode toggles for Independent vs Overlay.
2. Add one room-level display mode setting that applies to the entire room.
3. All directional view behavior resolves from that single room-level mode.
4. Directional editors remain responsible for directional assets, but not for mode ownership.

Initial acceptance criteria:

1. A room has exactly one display mode value at any given time.
2. No directional pane can store or present a conflicting mode value.
3. Switching room display mode updates all directional preview behavior consistently.
4. Existing projects with mixed directional mode states are migrated deterministically to one room-level value.
5. UI text and documentation describe display mode as a room property, not a directional property.

The room editor layout is preview-first.

Decision details:

1. Maximize center-panel area dedicated to Room Display Preview.
2. Minimize the visual footprint of directional image definition controls.
3. Directional definition controls should use compact, collapsible, or progressive disclosure patterns.
4. Layout should prioritize preview area expansion across common desktop resolutions.

Initial acceptance criteria:

1. Default room-editor layout allocates the majority of central horizontal space to preview.
2. Directional definition UI is compact by default and does not dominate the viewport.
3. Users can access all directional definition functions without permanently expanded heavy panels.
4. Preview remains primary focal element during directional image selection/editing.
5. Default layout uses a compact top control band (~10% height target) and a dominant preview pane below (~90% height target).

Directional image definition UI is consolidated into one co-located configuration area.

Decision details:

1. Do not place directional image definition controls in spatially analogous positions around the preview.
2. Move all directional image definition/editing controls into a single co-located panel/region.
3. Use directional selectors (for example tabs/list/segmented buttons) inside that one region to switch active direction.
4. Preserve directional clarity with strong labels/icons/status summaries instead of physical placement around preview.

Initial acceptance criteria:

1. Directional editing can be completed from one consolidated location.
2. Preview area is not fragmented by direction-specific control blocks around it.
3. Active direction is always clear via explicit UI state.
4. Switching directions in the consolidated editor updates preview deterministically.

Room preview implementation is split by mode-specific controls.

Decision details:

1. Do not force Overlay and Independent preview behavior into one control.
2. Create/shape a dedicated Overlay-mode preview control from the current overlay-oriented implementation.
3. Create a separate Independent-mode preview control.
4. For current scope, Independent-mode preview control can be a lightweight placeholder that only renders the selected image.

Initial acceptance criteria:

1. Room display mode selects which preview control is active.
2. Overlay-mode preview path preserves current overlay capabilities and behavior parity.
3. Independent-mode preview path exists and can display the selected image even if advanced behavior is deferred.
4. Shared shell and viewmodel orchestration remain mode-agnostic where practical, with mode-specific rendering isolated to dedicated controls.

Directional image editing uses a modeless per-direction dialog workflow.

Decision details:

1. Consolidated directional editor includes a direction selector dropdown.
2. Next to the selector, show a thumbnail of the selected direction's primary image.
3. Provide an Edit button that opens a modeless directional image editor dialog for that direction.
4. Multiple directional editor dialogs may remain open at the same time.
5. Directional editor dialog includes all direction image settings, including x/y/rotate controls.
6. Changes in modeless directional editor dialogs update the room preview in real time.
7. Directional editor dialog includes a simple checkbox toggle controlling whether that direction image is shown in preview.
8. X, Y, and Rotation controls support both direct text entry and up/down arrow bump adjustments.
9. Rotation control includes quick-set actions for 0, 90, 180, and -90.

Initial acceptance criteria:

1. Selecting a direction updates thumbnail and active-direction context immediately.
2. Edit launches modeless dialog for selected direction without blocking the room editor.
3. More than one directional dialog can be open concurrently without state corruption.
4. X, Y, and rotation edits propagate to preview in real time.
5. Closing a dialog does not discard already-applied changes unless explicit cancel semantics are introduced.
6. Toggling the preview-visibility checkbox immediately shows/hides that direction image in the preview.
7. X, Y, and Rotation values can be typed directly or changed one step at a time via arrow bump controls.
8. Rotation quick-set actions for 0, 90, 180, and -90 apply immediately and update preview.

## 1. Purpose

Create a deliberate, easier-to-use Room Designer experience that improves map authoring speed, reduces authoring mistakes, and keeps room-level editing predictable as projects scale.

## 2. Why This Plan Exists

The current room-designer workflow needs a broad UX and workflow pass before implementation details can be confidently staged.

Current pain themes to validate and prioritize:

1. Cognitive load is high for common room-edit operations.
2. Important context is split across multiple editor surfaces.
3. Room-level actions, objects, and traversal setup are harder to reason about than they should be.
4. Discovery and recoverability for mistakes are weaker than desired.

## 3. Revamp Vision

The room editor should feel like a guided composition workspace:

1. Structure first: room identity, placement, and traversal intent are obvious.
2. Content second: room objects, interactions, and behavior are edited in focused, coherent zones.
3. Validation always-on: users get immediate, actionable feedback before save/export.
4. Safe iteration: high-impact changes support preview and rollback-friendly workflows.

## 4. Scope

In scope for this plan:

1. Room Designer tab information architecture and interaction model.
2. Core room-edit workflows (create, update, reorder, connect, validate).
3. UX for object placement/assignment and room action authoring entry points.
4. Usability guardrails (warnings, inline diagnostics, confirmation strategy).
5. Regression and smoke-test strategy for room-editor behavior.

Out of scope for initial revamp slice:

1. Runtime contract redesign unless a clear gap is found.
2. Cross-host simulator UI redesign.
3. Large export-schema changes without explicit follow-up planning.

## 5. Concepts To Capture (What Good Looks Like)

Capture and refine these concepts during discovery:

1. Room-first workflow model
- From selecting a room to completing all required room setup without context thrash.

2. Spatial clarity model
- Clear visual relationship between room identity, links, and actionable content.

3. Authoring confidence model
- Inline validation that explains both what failed and how to fix it.

4. Progressive disclosure model
- Advanced options available without overwhelming basic room setup.

5. Safe-change model
- Preview-driven handling for destructive or high-variance actions.

## 6. Specifics To Capture (Concrete Requirement Backlog)

Track specifics as short requirement statements with acceptance criteria.

### 6.1 Workflow Requirements

1. Creating a new room requires only essential fields up front.
2. Editing a room keeps primary metadata visible while changing nested content.
3. Moving between rooms preserves unsaved intent safely (prompt/autosave strategy decision).
4. Bulk room operations have explicit confirmation and undo/rollback path.

### 6.2 Navigation and Linking Requirements

1. Room connection authoring is visible from room context.
2. Directionality and traversal mode are understandable without reading implementation details.
3. Invalid connection states are blocked or clearly surfaced with one-click remediation where possible.

### 6.3 Room Content Requirements

1. Room object lists support fast add/remove/find/filter operations.
2. Object assignment conflicts are surfaced inline before save.
3. Room action entry points are discoverable and not buried in unrelated panels.

### 6.4 Validation and Feedback Requirements

1. Validation summaries are actionable and linked to specific editor sections.
2. Inline errors avoid generic wording and point to exact correction path.
3. Save/export readiness state is visible from the room editor surface.

### 6.5 Performance and Scale Requirements

1. Editor remains responsive with large room counts and dense room content.
2. Selection and tab-switch latency remains stable under realistic project sizes.

### 6.6 Room Display Preview Requirements

1. Center panel is a non-authoring preview-only surface.
2. Preview source is directional room view configuration (not a default-image slot).
3. Overlay-mode directional images can be previewed without switching to unrelated editor contexts.
4. Preview updates are deterministic when changing direction/overlay source selection.
5. Directional image authoring controls are clearly separated from preview rendering.
6. Current slice preview rendering scope is main image only; multi-layer composition beyond main image is deferred.

### 6.7 Room Display Mode Requirements

1. Display mode is a room-level global setting with exactly one effective value per room.
2. Independent vs Overlay is selected once at room scope, not per direction.
3. Directional panels consume effective room mode but cannot override it.
4. Preview and directional rendering semantics stay consistent with selected room mode.
5. Legacy per-direction mode data is migrated to room-level mode with deterministic rules and diagnostics.

### 6.8 Preview-First Layout Requirements

1. Room Display Preview is the primary visual region in the room-editor center area.
2. Directional image definition controls are minimized by default while remaining discoverable.
3. Directional definition sections support compact/collapsed states and efficient expand-on-demand editing.
4. Layout preserves preview prominence at typical desktop sizes and remains usable on smaller widths.
5. Phase-1 IA must define measurable layout targets (for example, target preview width share and minimum preview viewport size).
6. Directional image definition controls are co-located in one consolidated editor region, not distributed around preview edges.
7. Baseline layout target is vertical: compact control band near top (~10% height) and preview pane below (~90% height).

### 6.10 Consolidated Directional Definition Requirements

1. All directional image configuration is managed from one shared UI region.
2. Direction selection control (tabs/list/segmented control) drives which directional settings are shown.
3. Each direction exposes a compact status summary when not active.
4. Consolidated editor remains compact by default and supports quick per-direction switching.
5. Directional context must remain unambiguous without spatial placement mimicry.
6. Direction selector presents all supported directions so users can choose any direction to review.
7. If a direction has no assigned image, show an icon plus "No image" text as thumbnail fallback.
8. When a direction is preview-hidden and its modeless dialog is closed, show a subtle hidden-state indicator in the consolidated thumbnail/status area.
9. Room image display authoring always supports all 8 directions for image-building workflows.
10. Image display direction availability is intentionally independent from traversal direction restrictions/settings.

### 6.11 Modeless Directional Editor Requirements

1. Direction selector dropdown chooses the active direction in the consolidated panel.
2. Consolidated panel shows current-direction primary-image thumbnail.
3. Edit action opens a modeless dialog scoped to the chosen direction.
4. Multiple modeless direction dialogs may be opened and used concurrently.
5. Dialog contains full directional image controls, including x/y offsets and rotation.
6. Dialog edits update preview output in real time.
7. Concurrent dialog edits resolve deterministically and keep preview/state synchronized.
8. Dialog includes a checkbox to include/exclude that directional image from preview rendering.
9. Checkbox changes apply immediately to preview and are reversible by re-toggle.
10. X, Y, and Rotation fields support dual input modes: typed numeric entry and incremental arrow bump controls.
11. Rotation field provides quick presets: 0, 90, 180, and -90.
12. Edit remains enabled even when no image is currently assigned for the selected direction.
13. The modeless directional dialog is the only authoring path for defining/adding directional images.
14. When room selection changes, open modeless directional dialogs auto-close.
15. Dialog edits apply immediately (apply-as-you-type / apply-on-click) without requiring an explicit Save button.
16. A lightweight update delay/debounce is allowed only to prevent unnecessary intermediate renders while preserving realtime user feel.
17. Arrow bump behavior for X, Y, and Rotation is single-step per click in the initial slice.
18. Preview visibility checkbox state is transient editor state only (not authored data).
19. Preview visibility checkbox state is not persisted to project JSON and does not affect clean export/runtime data.
20. The preview visibility checkbox in the modeless dialog is the primary visibility indicator while the dialog is open.
21. Modeless directional dialogs do not include a Cancel action in the initial slice.
22. X, Y, and Rotation input is permissive with no hard numeric clamps in the initial slice.
23. If absolute X, Y, or Rotation exceeds 1000, show a soft, non-blocking warning when exiting/closing the dialog.

### 6.9 Mode-Specific Preview Control Requirements

1. Overlay mode uses a dedicated preview control designed for overlay composition behavior.
2. Independent mode uses a separate dedicated preview control.
3. Independent-mode control may initially ship as a placeholder with simple image-display behavior.
4. Mode switch chooses the corresponding preview control deterministically.
5. Mode-specific rendering logic is isolated to control-specific components to reduce branching complexity.

## 7. Proposed Execution Phases

### Phase 0: Discovery Lock

1. Capture current pain inventory from existing behavior and tests.
2. Define top-priority user journeys (new room, edit room, connect rooms, validate).
3. Lock measurable success criteria for the revamp.

Deliverables:

1. Problem inventory.
2. Journey map.
3. Prioritized requirement list.

### Phase 1: Information Architecture

1. Propose room-editor layout and section model.
2. Define interaction rules for navigation between sections and rooms.
3. Validate MVVM-friendly boundaries for view/viewmodel/service responsibilities.

Deliverables:

1. IA blueprint.
2. Interaction rules.
3. Boundary notes for implementation staging.

### Phase 2: UX Slice Implementation (Low Risk)

1. Implement shell/layout and section scaffolding.
2. Improve discoverability and baseline validation presentation.
3. Keep behavior parity where possible while reducing friction.
4. Deliver mode-specific preview host routing with Overlay control first and Independent placeholder control.

Deliverables:

1. First shipped room-editor shell improvements.
2. Targeted regression tests.
3. Overlay preview control extraction/adaptation and Independent placeholder preview control wired to mode selection.

### Phase 3: Workflow Deepening

1. Improve linking and room-content operations.
2. Add safe-change flows for destructive/high-variance operations.
3. Refine diagnostics and correction loops.

Deliverables:

1. High-value workflow improvements.
2. Extended tests and UX guardrails.

### Phase 4: Hardening and Closeout

1. Performance pass for larger projects.
2. Accessibility and keyboard-flow pass.
3. Final regression and smoke validation.

Deliverables:

1. Stabilized room-editor revamp.
2. Completion notes and follow-up backlog.

## 8. Acceptance Criteria For Plan Completion

1. Priority room-edit journeys are simpler and require fewer context switches.
2. Validation feedback quality is materially improved and actionable.
3. Core room operations are covered by targeted automated tests.
4. No architecture boundary regressions across Designer, Shared, and Simulator.

## 9. Risks and Mitigations

1. Risk: Scope grows too large.
- Mitigation: Enforce phased slices with explicit out-of-scope list per slice.

2. Risk: UX-only changes accidentally shift runtime/export behavior.
- Mitigation: Add parity checks and focused regression tests when touching shared seams.

3. Risk: Large UI refactors increase regression chance.
- Mitigation: Prefer additive migration and remove old paths only after parity validation.

## 10. Validation Strategy (Per Implementation Slice)

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"

For runtime-boundary-impacting slices:

1. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"

## 11. Immediate Next Capture Tasks

1. Build a pain-point inventory for the current Room Designer tab.
2. List top 5 user journeys and expected success outcomes.
3. Define first implementation slice boundaries (what to change now vs defer).
4. Add an initial requirement table with Priority, Owner, Status, and Notes.

## 12. Requirement Tracker Seed

| ID | Requirement | Priority | Owner | Status | Notes |
| --- | --- | --- | --- | --- | --- |
| RER-001 | Streamline create-room flow to essential inputs first | High | TBD | Planned | Define exact minimal field set |
| RER-002 | Keep room metadata visible while editing room content | High | TBD | Planned | Evaluate sticky summary panel |
| RER-003 | Improve room linking discoverability and correction UX | High | TBD | Planned | Include invalid-state remediation |
| RER-004 | Add clearer inline validation and section-level summaries | High | TBD | Planned | Tie errors to direct fix locations |
| RER-005 | Define safe preview/rollback flow for destructive edits | Medium | TBD | Planned | Match existing safe-change patterns |
| RER-006 | Replace Default Room View with Room Display Preview (preview-only; no default image authoring) | High | TBD | Planned | Preserve overlay preview value from directional sources |
| RER-007 | Make Independent vs Overlay a room-global display mode (not per-direction) | High | TBD | Planned | Include deterministic migration for legacy mixed directional mode states |
| RER-008 | Maximize Room Display Preview real estate and minimize directional definition UI footprint | High | TBD | Planned | Define measurable layout targets during IA phase |
| RER-009 | Split Room Display Preview into mode-specific controls (Overlay control + Independent control) | High | TBD | Planned | Avoid single-control branching for divergent mode behavior |
| RER-010 | Ship Independent-mode preview as placeholder image viewer in initial slice | Medium | TBD | Planned | Defer advanced independent-mode behavior until later slice |
| RER-011 | Replace spatially distributed directional editors with one co-located directional configuration area | High | TBD | Planned | Preserve directional clarity with explicit labels/selectors/status summaries |
| RER-012 | Use direction dropdown + thumbnail + Edit button workflow in consolidated directional editor | High | TBD | Planned | Edit opens modeless per-direction dialog |
| RER-013 | Support multiple concurrent modeless directional dialogs with realtime preview sync | High | TBD | Planned | Includes x/y/rotate live updates |
| RER-014 | Add modeless directional dialog checkbox to toggle image visibility in preview | Medium | TBD | Planned | Transient preview-only toggle; not saved to project JSON/export/runtime data |
| RER-015 | Support typed and arrow-bump numeric editing for X/Y/Rotation with rotation quick presets (0, 90, 180, -90) | Medium | TBD | Planned | Preset actions apply immediately to preview |
| RER-016 | Keep Edit enabled for empty directions; use modeless dialog as sole directional image authoring path | High | TBD | Planned | No alternative inline image-definition entry points in top strip |
| RER-017 | Always allow all 8 directions in room image display authoring, independent of traversal settings | High | TBD | Planned | Display-building intentionally less restrictive than traversal |
| RER-018 | Keep numeric transform entry permissive (no hard limits) with soft warning when abs value exceeds 1000 on dialog exit | Medium | TBD | Planned | Non-blocking caution only |

## 13. Initial Brain-Dump Synthesis (Organized)

This section consolidates the initial room-editor revamp direction into an implementation-ready baseline.

### 13.1 Locked Decisions

1. Replace "Default Room View" with "Room Display Preview" (preview-only surface).
2. Remove default image authoring concept from the center panel.
3. Make display mode (Independent vs Overlay) room-global, not per direction.
4. Prioritize preview-first layout: maximize preview area and minimize directional definition chrome.
5. Use mode-specific preview controls:
- Dedicated Overlay preview control (primary current focus).
- Separate Independent preview control (placeholder image viewer in first slice).
6. Consolidate all directional image-definition UI into one co-located region; do not spread controls around preview by direction.
7. Use direction dropdown + thumbnail + Edit button in consolidated area, with modeless per-direction dialogs and real-time preview updates.

### 13.2 Immediate Implementation Sequence

1. Establish room-level display mode as the source of truth.
2. Introduce preview host routing based on room mode.
3. Extract/adapt current overlay-centric preview into Overlay control.
4. Add Independent placeholder preview control with simple image rendering.
5. Remove remaining default-image authoring affordances from center area.
6. Replace directional edge-positioned editors with a consolidated direction-switching editor panel.
7. Add modeless per-direction editor dialog and live x/y/rotate preview synchronization.

### 13.3 UX Principles To Preserve During Refactor

1. Preserve current overlay preview value and behavior parity where expected.
2. Keep directional image definition available but visually secondary.
3. Maintain deterministic preview updates when users change directional inputs.
4. Avoid introducing mode-conflict states in UI or persisted data.

### 13.4 Risks To Watch Early

1. Legacy per-direction mode data can produce ambiguous migration cases.
2. Overlay behavior parity can regress during control extraction.
3. Aggressive compaction can hide directional definition discoverability.

### 13.5 Next Discussion Frame: Screen Real-Estate Balance

Use this frame to finalize layout targets before implementation:

1. Default width allocation target between preview and directional-definition regions.
2. Minimum preview viewport dimensions before layout adaptation triggers.
3. Which directional controls are always visible vs collapsed by default.
4. Expand/collapse behavior and persistence (per room, per session, or global preference).
5. Compact-summary design for directional definitions when collapsed.
6. Consolidated directional editor interaction model (dropdown + thumbnail + Edit button) and default footprint.

Proposed starting targets for discussion (to confirm, adjust, or reject):

1. Desktop default: vertical layout with ~10% top control band and ~90% preview pane.
2. Minimum preview viewport target: 900x500 logical pixels before responsive compaction escalates.
3. Directional controls default compact with one-line status summaries.
4. Single-click Edit opens modeless per-direction dialog; central layout remains compact.

## 14. Pre-Implementation Questions (Review One By One)

Purpose: finalize unresolved design and behavior decisions before implementation begins.

### 14.1 Layout and Real-Estate Decisions

1. Locked: default desktop layout target is ~10% top control band and ~90% preview pane.
2. Prototype-first: implement the compact top band with current expected control set, then validate real usage before adding overflow behavior.
3. Defer exact narrow-width top-band overflow rules until post-prototype review.
4. Defer preview-downscale vs control-wrapping precedence until post-prototype review.

### 14.2 Consolidated Directional Panel Behavior

1. Locked: room image display editing always exposes all 8 directions.
2. Locked: dropdown lists all supported directions with configured/not-configured status badges.
3. Locked: thumbnail fallback for no-image directions is icon + "No image" text.
4. Locked: Edit stays enabled for no-image directions and opens modeless dialog where image definition/assignment occurs.

### 14.3 Modeless Dialog Lifecycle and Concurrency

1. Locked: if Edit is clicked for a direction with an existing open dialog, focus/activate the existing dialog.
2. Locked: allow at most one open dialog per direction at a time.
3. Locked for initial slice: dialogs reopen with default position/size; persistence can be revisited later.
4. Locked: if room selection changes, all open modeless directional dialogs auto-close.

### 14.4 Live Update and Commit Semantics

1. Locked: changes are apply-as-you-type/apply-on-click and update preview immediately; no explicit Save in initial slice.
2. Locked: no Cancel action is required for modeless directional dialogs in this slice.
3. Locked: lightweight debounce is allowed to reduce unnecessary intermediate updates, but interaction must still feel realtime.
4. Locked: arrow-bump behavior is single-step per click in first slice.

### 14.5 Numeric Input Rules (X, Y, Rotation)

1. Locked: no hard numeric ranges/clamps for X, Y, and Rotation in initial slice.
2. Locked: keep raw entered rotation values (no normalization remap in initial slice).
3. Locked: arrow-bump increment remains 1 per click with no modifier-step behavior in initial slice.
4. Locked: quick rotation presets are 0, 90, 180, and -90.

### 14.6 Preview Visibility Toggle Semantics

1. Locked: show/hide checkbox is preview-only (moment-in-time producer convenience), not authored directional metadata.
2. Locked for current slice: overlay preview handles only the main image layer; additional layer ordering is deferred.
3. Locked: checkbox in the modeless dialog is the primary indicator; when dialog is closed, show hidden state in the thumbnail/status area.
4. Locked: toggling visibility affects editor preview only and is not persisted to JSON/export/runtime content.

### 14.7 Migration and Backward Compatibility

1. Locked for prototype slice: no automated migration is required.
2. Locked for prototype slice: no migration notice/banner is required.
3. Locked for prototype slice: no migration diagnostics are required.
4. Locked for prototype slice: legacy files can be corrected manually as needed.

### 14.8 Testing and Guardrails

1. Locked: use the approved top-5 must-have automated tests listed below for slice 1.
2. Locked: include dialog-concurrency coverage in slice 1 (covered by modeless lifecycle policy test).
3. Locked: prioritize smoke paths that validate preview-first layout and directional edit entry workflows.
4. Locked: set a lightweight responsiveness baseline for rapid transform edits during prototype validation and refine after first pass.

Proposed top-5 must-have automated tests for slice 1:

1. Room-level display mode routing test
- Verifies room-global display mode selects the correct preview control host (Overlay control vs Independent placeholder), independent of per-direction entry values.
 - Review status: Approved.

2. Direction selector and edit entry test
- Verifies dropdown shows all 8 directions, no-image directions show icon + "No image" thumbnail fallback, and Edit remains enabled for empty directions.
 - Review status: Approved.

3. Modeless dialog lifecycle policy test
- Verifies one dialog per direction (Edit re-focuses existing), and switching rooms auto-closes open directional dialogs.
 - Review status: Approved.

4. Live transform update and control semantics test
- Verifies apply-as-you-type behavior for X/Y/Rotation updates preview with realtime feel, arrow bump is single-step per click, and presets 0/90/180/-90 apply immediately.
 - Review status: Approved.

5. Preview-only visibility toggle boundary test
- Verifies checkbox show/hide immediately affects preview, hidden-state cue appears in thumbnail/status area when dialog is closed, and visibility toggle state is not persisted/authored/exported.
 - Review status: Approved.

### 14.9 Recommended Review Order

1. Minimum preview size validation after first prototype pass.
2. Directional strip behavior and dropdown content model.
3. Modeless dialog instance policy (one per direction vs many).
4. Live update/commit semantics.
5. Numeric ranges and stepping behavior.
6. Visibility toggle persistence and overlay layering semantics.
7. Migration rule for legacy mode data.
8. Slice-1 must-have test list.

### 14.10 Decision Log (Resolved)

1. Layout split: locked to preview-first vertical composition with approximately 10% top controls and 90% preview pane.
2. Top-band responsiveness: prototype first and tune after visual validation; no preemptive overflow complexity added now.
3. Direction selector content: show all supported directions and let users choose what to review.
4. Thumbnail fallback: use icon + "No image" text when a direction has no assigned image.
5. Edit behavior: always enabled; modeless directional dialog is the only path to define/add directional images.
6. Dialog reuse policy: one dialog per direction; Edit re-focuses existing dialog for that direction.
7. Dialog position/size: use default on reopen for now; revisit persistence only if needed.
8. Room switch behavior: auto-close all open modeless directional dialogs.
9. Commit semantics: apply-as-you-type/apply-on-click with realtime feel; optional light debounce permitted.
10. Arrow bump interaction: single-step per click for initial slice.
11. Preview visibility toggle: transient preview-only state, not authored/persisted and not part of export/runtime data.
12. Overlay layering scope: main image only in this slice; non-main layering behavior deferred to a later feature discussion.
13. Hidden-state indication: modeless checkbox is primary; thumbnail/status area carries a subtle hidden cue when dialog is closed.
14. Direction availability for image display: always all 8 directions, independent of traversal mode constraints.
15. Dialog command scope: no Cancel action for modeless directional dialogs in initial slice.
16. Numeric policy: permissive transform values with no hard clamps; show soft non-blocking warning on dialog exit when absolute value exceeds 1000.
17. Migration policy for prototype: skip automated migration and handle legacy file adjustments manually if needed.
18. Testing scope for slice 1: all five proposed automated tests are approved, including dialog-concurrency coverage.

## 15. First Prototype Implementation Task List (File-Level)

Purpose: execute a thin vertical slice that proves the new room display workflow with minimal risk.

### 15.1 Workspace Layout Refactor (Preview-First 10/90)

1. Replace directional edge-card layout with compact top control strip + dominant preview pane.
- Target files: [StoryboardDesigner.App/Views/Controls/RoomDesignerWorkspace.xaml](StoryboardDesigner.App/Views/Controls/RoomDesignerWorkspace.xaml), [StoryboardDesigner.App/ViewModels/RoomEditorTabViewModel.cs](StoryboardDesigner.App/ViewModels/RoomEditorTabViewModel.cs)

2. Rename center experience from default-view semantics to preview semantics.
- Target files: [StoryboardDesigner.App/Views/Controls/RoomDefaultViewPreview.xaml](StoryboardDesigner.App/Views/Controls/RoomDefaultViewPreview.xaml)

### 15.2 Room-Level Display Mode Source Of Truth

1. Introduce room-level image display mode field and bind top-strip mode selector to it.
- Target files: [StoryboardDesigner.App/Models/Rooms/Room.cs](StoryboardDesigner.App/Models/Rooms/Room.cs), [StoryboardDesigner.App/ViewModels/RoomEditorTabViewModel.cs](StoryboardDesigner.App/ViewModels/RoomEditorTabViewModel.cs)

2. Keep per-entry values only as transitional implementation detail in prototype (no migration work), but route preview selection from room-level mode.
- Target files: [StoryboardDesigner.App/Models/Rooms/RoomImageEntry.cs](StoryboardDesigner.App/Models/Rooms/RoomImageEntry.cs), [StoryboardDesigner.App/ViewModels/RoomImageSlotViewModel.cs](StoryboardDesigner.App/ViewModels/RoomImageSlotViewModel.cs), [StoryboardDesigner.App/Views/Controls/RoomDefaultViewPreview.xaml](StoryboardDesigner.App/Views/Controls/RoomDefaultViewPreview.xaml)

### 15.3 Split Preview Controls By Mode

1. Extract/adapt current overlay behavior into a dedicated overlay preview control.
- Target files: [StoryboardDesigner.App/Views/Controls/RoomDefaultViewPreview.xaml](StoryboardDesigner.App/Views/Controls/RoomDefaultViewPreview.xaml)

2. Add independent-mode placeholder preview control that only displays selected direction image.
- Target files: [StoryboardDesigner.App/Views/Controls/RoomDesignerWorkspace.xaml](StoryboardDesigner.App/Views/Controls/RoomDesignerWorkspace.xaml)

3. Add preview-host routing by room-level mode.
- Target files: [StoryboardDesigner.App/ViewModels/RoomEditorTabViewModel.cs](StoryboardDesigner.App/ViewModels/RoomEditorTabViewModel.cs), [StoryboardDesigner.App/Views/Controls/RoomDesignerWorkspace.xaml](StoryboardDesigner.App/Views/Controls/RoomDesignerWorkspace.xaml)

### 15.4 Consolidated Direction Selector Strip

1. Add dropdown with all 8 directions and configured/not-configured state.
- Target files: [StoryboardDesigner.App/ViewModels/RoomEditorTabViewModel.cs](StoryboardDesigner.App/ViewModels/RoomEditorTabViewModel.cs), [StoryboardDesigner.App/Views/Controls/RoomDesignerWorkspace.xaml](StoryboardDesigner.App/Views/Controls/RoomDesignerWorkspace.xaml)

2. Add selected-direction thumbnail with no-image fallback (icon + text).
- Target files: [StoryboardDesigner.App/ViewModels/RoomImageSlotViewModel.cs](StoryboardDesigner.App/ViewModels/RoomImageSlotViewModel.cs), [StoryboardDesigner.App/Views/Controls/RoomDesignerWorkspace.xaml](StoryboardDesigner.App/Views/Controls/RoomDesignerWorkspace.xaml)

3. Keep Edit enabled for all directions including empty-image directions.
- Target files: [StoryboardDesigner.App/ViewModels/MainWindowViewModel.RoomImageCommands.cs](StoryboardDesigner.App/ViewModels/MainWindowViewModel.RoomImageCommands.cs), [StoryboardDesigner.App/ViewModels/RoomEditorTabViewModel.cs](StoryboardDesigner.App/ViewModels/RoomEditorTabViewModel.cs)

### 15.5 Modeless Directional Dialog Pipeline

1. Convert image editor service from modal TryEdit pattern to modeless open/focus management.
- Target files: [StoryboardDesigner.App/Services/IRoomImageEditorService.cs](StoryboardDesigner.App/Services/IRoomImageEditorService.cs), [StoryboardDesigner.App/Services/RoomImageEditorService.cs](StoryboardDesigner.App/Services/RoomImageEditorService.cs), [StoryboardDesigner.App/ViewModels/MainWindowViewModel.RoomImageCommands.cs](StoryboardDesigner.App/ViewModels/MainWindowViewModel.RoomImageCommands.cs)

2. Enforce one dialog per direction and focus existing dialog on repeated Edit.
- Target files: [StoryboardDesigner.App/Services/RoomImageEditorService.cs](StoryboardDesigner.App/Services/RoomImageEditorService.cs)

3. Auto-close all open directional dialogs when selected room changes.
- Target files: [StoryboardDesigner.App/ViewModels/MainWindowViewModel.cs](StoryboardDesigner.App/ViewModels/MainWindowViewModel.cs), [StoryboardDesigner.App/Services/IRoomImageEditorService.cs](StoryboardDesigner.App/Services/IRoomImageEditorService.cs), [StoryboardDesigner.App/Services/RoomImageEditorService.cs](StoryboardDesigner.App/Services/RoomImageEditorService.cs)

### 15.6 Dialog UX And Live Editing Behavior

1. Update dialog to remove Save/Cancel semantics in prototype and support apply-as-you-type updates.
- Target files: [StoryboardDesigner.App/Views/RoomImageEditorDialog.xaml](StoryboardDesigner.App/Views/RoomImageEditorDialog.xaml), [StoryboardDesigner.App/Views/RoomImageEditorDialog.xaml.cs](StoryboardDesigner.App/Views/RoomImageEditorDialog.xaml.cs)

2. Add controls in dialog for:
- preview visibility checkbox (transient only)
- X/Y/Rotation text + arrow bump
- rotation quick presets (0, 90, 180, -90)
- soft warning on close when abs(X/Y/Rotation) > 1000
- Target files: [StoryboardDesigner.App/Views/RoomImageEditorDialog.xaml](StoryboardDesigner.App/Views/RoomImageEditorDialog.xaml), [StoryboardDesigner.App/Views/RoomImageEditorDialog.xaml.cs](StoryboardDesigner.App/Views/RoomImageEditorDialog.xaml.cs), [StoryboardDesigner.App/ViewModels/RoomImageSlotViewModel.cs](StoryboardDesigner.App/ViewModels/RoomImageSlotViewModel.cs)

3. Ensure hidden-state indicator appears in thumbnail/status area when dialog is closed.
- Target files: [StoryboardDesigner.App/Views/Controls/RoomDesignerWorkspace.xaml](StoryboardDesigner.App/Views/Controls/RoomDesignerWorkspace.xaml), [StoryboardDesigner.App/ViewModels/RoomImageSlotViewModel.cs](StoryboardDesigner.App/ViewModels/RoomImageSlotViewModel.cs)

### 15.7 Persistence Boundary And Export Safety

1. Keep preview visibility toggle out of authored/exported data.
- Target files: [StoryboardDesigner.App/Serialization/RoomExportDto.cs](StoryboardDesigner.App/Serialization/RoomExportDto.cs)

2. Confirm no runtime/shared contract coupling introduced by room-editor UI refactor.
- Target files: [StoryboardDesigner.App/Models/Rooms/Room.cs](StoryboardDesigner.App/Models/Rooms/Room.cs)

### 15.8 Approved Test Work For Slice 1

1. Add tests for the approved top-5 behaviors in the existing test project.
- Target location: [StoryboardDesigner.App.Tests/StoryboardDesigner.App.Tests.csproj](StoryboardDesigner.App.Tests/StoryboardDesigner.App.Tests.csproj)

2. Add/adjust smoke paths for preview-first layout and directional edit entry.
- Target location: [StoryboardDesigner.App.SmokeTests/StoryboardDesigner.App.SmokeTests.csproj](StoryboardDesigner.App.SmokeTests/StoryboardDesigner.App.SmokeTests.csproj)

### 15.9 Validation Gate For This Slice

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"

## 16. Room Designer Code Organization Convention (Proposed)

Purpose: treat Room Designer as a cohesive module so future changes are easier to find, reason about, and test.

### 16.1 Convention Choice

Use both:

1. Folder grouping by feature/module.
2. Consistent RoomDesigner prefix/suffix naming for module-specific types.

Rationale: folder-only still allows ambiguous class names, and naming-only still leaves files scattered.

### 16.2 Proposed Folder Grouping

Target feature root inside the app project:

1. [StoryboardDesigner.App/Features/RoomDesigner](StoryboardDesigner.App/Features/RoomDesigner)

Suggested subfolders:

1. [StoryboardDesigner.App/Features/RoomDesigner/Views](StoryboardDesigner.App/Features/RoomDesigner/Views)
2. [StoryboardDesigner.App/Features/RoomDesigner/ViewModels](StoryboardDesigner.App/Features/RoomDesigner/ViewModels)
3. [StoryboardDesigner.App/Features/RoomDesigner/Services](StoryboardDesigner.App/Features/RoomDesigner/Services)
4. [StoryboardDesigner.App/Features/RoomDesigner/Models](StoryboardDesigner.App/Features/RoomDesigner/Models)
5. [StoryboardDesigner.App/Features/RoomDesigner/Controls](StoryboardDesigner.App/Features/RoomDesigner/Controls)

### 16.3 Naming Convention

Use RoomDesigner-prefixed names for module-root surfaces and mode-specific preview components.

Examples:

1. RoomDesignerWorkspaceView
2. RoomDesignerPreviewHostControl
3. RoomDesignerOverlayPreviewControl
4. RoomDesignerIndependentPreviewControl
5. RoomDesignerDirectionStripControl
6. RoomDesignerDirectionImageDialog
7. RoomDesignerDirectionImageDialogService

Keep generic names only for truly cross-feature reusable primitives.

### 16.4 Adoption Strategy (Prototype-Friendly)

1. Do not run a large rename move first.
2. For this prototype slice, place all new room-designer artifacts in the feature folder and follow naming convention immediately.
3. Leave existing files in place unless they are directly touched by prototype work.
4. After prototype stabilization, run one cleanup pass to move/rename legacy room-designer files into the feature module.

### 16.5 Initial Mapping Targets (When Touched)

1. [StoryboardDesigner.App/Views/Controls/RoomDesignerWorkspace.xaml](StoryboardDesigner.App/Views/Controls/RoomDesignerWorkspace.xaml)
2. [StoryboardDesigner.App/Views/Controls/RoomDefaultViewPreview.xaml](StoryboardDesigner.App/Views/Controls/RoomDefaultViewPreview.xaml)
3. [StoryboardDesigner.App/ViewModels/RoomEditorTabViewModel.cs](StoryboardDesigner.App/ViewModels/RoomEditorTabViewModel.cs)
4. [StoryboardDesigner.App/ViewModels/RoomImageSlotViewModel.cs](StoryboardDesigner.App/ViewModels/RoomImageSlotViewModel.cs)
5. [StoryboardDesigner.App/Views/RoomImageEditorDialog.xaml](StoryboardDesigner.App/Views/RoomImageEditorDialog.xaml)
6. [StoryboardDesigner.App/Services/RoomImageEditorService.cs](StoryboardDesigner.App/Services/RoomImageEditorService.cs)

### 16.6 Guardrail

If a file is specific to Room Designer and not reused elsewhere, it should not live in a global folder by default.

## 17. Room Designer First-Wave Move Plan (Prototype Slice)

Purpose: apply the new module convention with minimal churn while implementing the prototype.

### 17.1 Move Strategy

1. Move only files that are directly modified in the prototype slice.
2. Keep namespace updates aligned with new paths in the same change.
3. Avoid mixed behavior refactor plus broad file relocation in one PR when possible.

### 17.2 Wave-1 File Mapping (When Touched)

1. [StoryboardDesigner.App/Views/Controls/RoomDesignerWorkspace.xaml](StoryboardDesigner.App/Views/Controls/RoomDesignerWorkspace.xaml) -> [StoryboardDesigner.App/Features/RoomDesigner/Views/RoomDesignerWorkspaceView.xaml](StoryboardDesigner.App/Features/RoomDesigner/Views/RoomDesignerWorkspaceView.xaml)
2. [StoryboardDesigner.App/Views/Controls/RoomDefaultViewPreview.xaml](StoryboardDesigner.App/Views/Controls/RoomDefaultViewPreview.xaml) -> [StoryboardDesigner.App/Features/RoomDesigner/Controls/RoomDesignerOverlayPreviewControl.xaml](StoryboardDesigner.App/Features/RoomDesigner/Controls/RoomDesignerOverlayPreviewControl.xaml)
3. [StoryboardDesigner.App/ViewModels/RoomEditorTabViewModel.cs](StoryboardDesigner.App/ViewModels/RoomEditorTabViewModel.cs) -> [StoryboardDesigner.App/Features/RoomDesigner/ViewModels/RoomDesignerTabViewModel.cs](StoryboardDesigner.App/Features/RoomDesigner/ViewModels/RoomDesignerTabViewModel.cs)
4. [StoryboardDesigner.App/ViewModels/RoomImageSlotViewModel.cs](StoryboardDesigner.App/ViewModels/RoomImageSlotViewModel.cs) -> [StoryboardDesigner.App/Features/RoomDesigner/ViewModels/RoomDesignerImageSlotViewModel.cs](StoryboardDesigner.App/Features/RoomDesigner/ViewModels/RoomDesignerImageSlotViewModel.cs)
5. [StoryboardDesigner.App/Views/RoomImageEditorDialog.xaml](StoryboardDesigner.App/Views/RoomImageEditorDialog.xaml) -> [StoryboardDesigner.App/Features/RoomDesigner/Views/RoomDesignerDirectionImageDialog.xaml](StoryboardDesigner.App/Features/RoomDesigner/Views/RoomDesignerDirectionImageDialog.xaml)
6. [StoryboardDesigner.App/Views/RoomImageEditorDialog.xaml.cs](StoryboardDesigner.App/Views/RoomImageEditorDialog.xaml.cs) -> [StoryboardDesigner.App/Features/RoomDesigner/Views/RoomDesignerDirectionImageDialog.xaml.cs](StoryboardDesigner.App/Features/RoomDesigner/Views/RoomDesignerDirectionImageDialog.xaml.cs)
7. [StoryboardDesigner.App/Services/IRoomImageEditorService.cs](StoryboardDesigner.App/Services/IRoomImageEditorService.cs) -> [StoryboardDesigner.App/Features/RoomDesigner/Services/IRoomDesignerDirectionImageDialogService.cs](StoryboardDesigner.App/Features/RoomDesigner/Services/IRoomDesignerDirectionImageDialogService.cs)
8. [StoryboardDesigner.App/Services/RoomImageEditorService.cs](StoryboardDesigner.App/Services/RoomImageEditorService.cs) -> [StoryboardDesigner.App/Features/RoomDesigner/Services/RoomDesignerDirectionImageDialogService.cs](StoryboardDesigner.App/Features/RoomDesigner/Services/RoomDesignerDirectionImageDialogService.cs)

### 17.3 New Files Added In Prototype (No Legacy Move Needed)

1. [StoryboardDesigner.App/Features/RoomDesigner/Controls/RoomDesignerIndependentPreviewControl.xaml](StoryboardDesigner.App/Features/RoomDesigner/Controls/RoomDesignerIndependentPreviewControl.xaml)
2. [StoryboardDesigner.App/Features/RoomDesigner/Controls/RoomDesignerPreviewHostControl.xaml](StoryboardDesigner.App/Features/RoomDesigner/Controls/RoomDesignerPreviewHostControl.xaml)
3. [StoryboardDesigner.App/Features/RoomDesigner/Controls/RoomDesignerDirectionStripControl.xaml](StoryboardDesigner.App/Features/RoomDesigner/Controls/RoomDesignerDirectionStripControl.xaml)

### 17.4 Execution Order

1. Create feature folders and add new files there first.
2. Move/rename dialog service and dialog view files.
3. Move/rename room-designer viewmodels.
4. Move/rename workspace/preview controls.
5. Update composition wiring and references.
6. Build and run targeted tests before additional behavior changes.

### 17.5 Deferred Moves (Post-Prototype Cleanup)

1. Any room-image related helpers still referenced broadly from other features.
2. Optional pass to rename remaining "RoomImage" type names to "RoomDesigner" naming where truly module-specific.

### 17.6 Current Status (2026-07-10)

Completed in workspace:

1. Wave-1 file mapping rename/moves in 17.2 are now applied under [StoryboardDesigner.App/Features/RoomDesigner](StoryboardDesigner.App/Features/RoomDesigner).
2. 17.3 placeholder controls now exist:
- [StoryboardDesigner.App/Features/RoomDesigner/Controls/RoomDesignerIndependentPreviewControl.xaml](StoryboardDesigner.App/Features/RoomDesigner/Controls/RoomDesignerIndependentPreviewControl.xaml)
- [StoryboardDesigner.App/Features/RoomDesigner/Controls/RoomDesignerPreviewHostControl.xaml](StoryboardDesigner.App/Features/RoomDesigner/Controls/RoomDesignerPreviewHostControl.xaml)
- [StoryboardDesigner.App/Features/RoomDesigner/Controls/RoomDesignerDirectionStripControl.xaml](StoryboardDesigner.App/Features/RoomDesigner/Controls/RoomDesignerDirectionStripControl.xaml)
3. RoomDesigner dialog service internals are split one-type-per-file for maintainability.
4. Validation closeout completed:
- dotnet build .\StoryboardDesigner.slnx
- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"
5. Naming cleanup pass applied for remaining room-designer-specific composition/test naming and smoke baseline now asserts the room designer workspace pane is present.

Remaining from this plan section:

1. Optional future pass only if desired: broader domain-term rename from "RoomImage" to new vocabulary in shared model/export types (not required for this prototype slice).
