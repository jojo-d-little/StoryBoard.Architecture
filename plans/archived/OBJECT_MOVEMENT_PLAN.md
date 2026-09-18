# Object Movement Plan

Status: Closed 2026-08-01; movement baseline and lean Phase 5 hardening complete
Owner: StoryboardDesigner authoring + Storyboard.Shared runtime
Last updated: 2026-08-01

## Closeout Snapshot (2026-08-01)

Closed with the following completion evidence:

1. Simulator tools/menu cleanup and reinitialize behavior landed and validated.
2. Stack scale min/max constraints are centralized and enforced across authoring and runtime paths.
3. Clean navigation export direction mapping regression was corrected and validated.
4. Height pinning export shape was normalized so paired fields remain consistent in clean artifacts.
5. Birmingham clean-export snapshot baseline was refreshed to match corrected export behavior.
6. Validation gates are green:
- `dotnet build .\StoryboardDesigner.slnx`
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`

Deferred follow-on remains explicitly out of this closure window:

1. Transition-hint chooser evolution remains post-v1.
2. Waypoint plotting/host-buffered intent remains in dedicated follow-on planning.

## Reconciliation Snapshot (2026-07-26)

Completed baseline through current checkpoints:

1. Phase F closeout is complete and stable.
2. Phase 3A contract/gateway surface is complete.
3. Phase 3B core move/rotate/stack evaluator behavior is implemented and under hardening.
4. Command integration is active for movement actions, including active-selection fallback behavior.
5. Core command parsing now carries canonical parsed direction/numeric outputs used by movement actions.
6. Phase 3C recompute debug-code emission and high-diagnostics verification are complete (host-visible, non-contract diagnostics).

Remaining for movement-plan closeout:

1. Execute Phase 5 targeted hardening/regression/doc sweep (lean closeout profile).
2. Keep deferred transition-hint chooser as explicit post-v1 follow-up.

## Checkpoint Note (2026-07-25) - Move/Stack Runtime Hardening

Context:

1. Runtime move/stack behavior received multiple fixes driven by simulator playtesting edge cases.
2. The active focus was deterministic handling of stacked movement, partial-support transitions, and path-vs-destination blocking policy.

Completed in this checkpoint:

1. Added move option `allowJumpOver` and wired it through authoring payload, persistence, runtime request mapping, and execution.
2. Locked behavior so destination conflicts remain strict while intermediate path conflicts are optionally bypassed only when `allowJumpOver=true`.
3. Corrected stacked traversal behavior so moving across support without `allowJumpOver` fails with `PartialSupportNotAllowed`.
4. Added move-to-stack snap behavior for valid support approach cases while preserving strict failure when support cannot contain the subject footprint.
5. Implemented runtime stacked render-scale computation in shared command output:
- effective scale now derives from base image scale + stack order + optional per-object overrides.
6. Added runtime variable mapping for stack scale inputs (`baseImageScale`, `stackScaleStepOverride`, `minStackScaleOverride`) in both project and clean bootstrap paths.

Validation status:

1. `dotnet build .\StoryboardDesigner.slnx` passes.
2. `dotnet test .\Storyboard.Shared.Tests\Storyboard.Shared.Tests.csproj --filter "RuntimeScopeMutationGatewayPhase3ATests|RuntimeMoveRoomObjectOnGridActionTests|RuntimeActionPayloadAccessorsTests|RuntimeRenderablePlacementNormalizationTests"` passes.
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameCommandProcessorFixtureTests|GameSimulatorPlaybackRegressionTests"` passes.

Resume intent:

1. Add one explicit host-level key/backpack command fixture that mirrors manual playtext phrasing for across-support movement with and without `allowJumpOver`. Completed via automated tests.
2. Four previously-manual scenarios are now covered by automated tests:
- stacked move-off-support one cell (expected success)
- across-support without `allowJumpOver` (expected fail)
- across-support with `allowJumpOver` (expected success)
- destination collision with `allowJumpOver` (expected fail)
3. If simulator behavior ever differs from tests, treat simulator-observed outcome as source case and add a focused regression before further behavior changes.

## Transition Note (2026-07-23) - Phase F Closeout and Phase 3 Start

Decision:

1. Phase F (Foundation Contract Realignment) is formally closed.
2. Work now formally transitions into Phase 3 (Runtime Grid Movement Core).

Signoff rationale:

1. Foundation naming and ownership split is implemented and validated.
2. Grouped mapping/compatibility foundations are stable with regression coverage.
3. Deterministic occupancy/effective spatial recompute and diagnostics plumbing are in place.

Deferred into Phase 3:

1. Formal diagnostic-code taxonomy for recompute conflict categories remains an explicit Phase 3 hardening item and does not block Phase F closure.

## Checkpoint Note (2026-07-23)

Context:

1. Current effort focused on visual scaling/alignment correctness between the object appearance dialog and room designer preview before movement implementation proceeds.

Completed in this checkpoint:

1. Native-pixel scaling semantics aligned so scale `1.0` maps to native image size.
2. Appearance and room preview pipelines aligned to width/height-based scaling (instead of mixed transform-based scaling behavior).
3. Appearance scale UX improved with live typed-apply and finer nudge granularity.
4. Project grid cell size propagation into appearance guide behavior completed.

Validation status:

1. `dotnet build .\StoryboardDesigner.slnx` passes.
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~RoomDesignerTabViewModelPreviewWorkflowTests"` passes.

Resume intent:

1. Perform additional visual review with real assets before moving forward with new movement slices.
2. Treat this visual correctness area as a hard quality gate for the next implementation phase.

## Checkpoint Note (2026-07-23) - Render Order Property Split

Context:

1. We identified that a single property (`renderZOrder`) currently conflates designer-authored ordering intent with runtime-effective render ordering.
2. With stack/height behavior in play, authored intent and runtime-resolved outcome can diverge and should not share one semantic field.

Locked direction:

1. Introduce a distinct authored ordering field (working name: `authoredRenderOrder`) for room-designer intent.
2. Preserve runtime-effective ordering as a separate concept/property (current `renderZOrder`, or explicit rename to `effectiveRenderZOrder` during implementation planning).
3. Runtime remains authoritative for final effective order at init and during state changes.

Runtime init corner-case policy direction:

1. When authored startup stacking/order is not legal, runtime performs deterministic normalization using a stable tie-break sequence.
2. Runtime emits diagnostics for each auto-resolved conflict.
3. Authored data is not mutated during runtime normalization; only runtime-effective state changes.

## 1. Purpose

Define a safe, phased approach for allowing players to move objects within the current room while preserving existing runtime/designer boundaries and avoiding regressions in inventory/container flows.

## 2. Problem Statement

Today, movement-related behavior is mainly represented by container transfer mechanics and inventoriable semantics. We need explicit room-scoped object movement behavior that:

1. Applies to objects present in the current room context.
2. Distinguishes movable vs non-movable objects.
3. Supports a class of movable-but-not-inventoriable objects.
4. Keeps one clear source of truth for movement eligibility.

## 3. Scope and Non-Goals

In scope:

1. Define object-level movement capability semantics.
2. Introduce a movement eligibility flag model (proposed: `IsMovable`).
3. Add room-scoped runtime movement rules for current-room objects.
4. Add/update authoring UX and validation for movement capability.
5. Add regression tests across model, runtime command flow, and guardrails.
6. Add room-designer snap-to-grid placement support.
7. Add optional grid-visual overlay support in room designer.

Out of scope (initial slices):

1. Cross-room teleport/move mechanics for arbitrary objects.
2. Physics/pathfinding simulation.
3. Broad redesign of inventory/container architecture.
4. Unplanned clean export contract break.

## 4. Boundaries and Placement

1. `StoryboardDesigner.App`:
- Authoring UI, feature toggles, editing constraints, and validation messages.

2. `Storyboard.Shared`:
- Runtime movement eligibility evaluation and room-scoped movement execution contracts.

3. `Storyboard.Simulator`:
- Host-specific UX for invoking/testing movement behavior without introducing designer coupling.

Boundary rule:

1. No dependency from `Storyboard.Simulator` to `StoryboardDesigner.App`.

## 4.1 Shared-First Logic Placement Rule (Locked Direction)

When grid/snap/movement behavior is required by both hosts, place reusable implementation in `Storyboard.Shared` and keep host UX orchestration in each host project.

Placement matrix:

1. `Storyboard.Shared` owns:
- Effective grid resolution policy (project default plus room override precedence).
- Grid coordinate math, direction step translation, boundary checks, and collision-step evaluation primitives.
- Runtime movement eligibility evaluation and active-item resolution core.
- Shared DTO/contracts used by both hosts for movement and active-item state.

2. `StoryboardDesigner.App` owns:
- Room-designer snap toggle UX and overlay rendering UX.
- Designer-specific placement workflows (drag/drop authoring interaction, editor affordances, diagnostics messaging).
- Authoring persistence wiring for project/room grid settings and object feature editing.

3. `Storyboard.Simulator` owns:
- Runtime-host UX for player-facing object selection gestures and active-item controls.
- Simulator-specific visualization/interaction behavior that consumes shared contracts.
- Host command wiring and display of runtime movement outcomes.

Deferred runtime-rendering guidance (apply when simulator/runtime rendering alignment work is activated):

1. Keep simulator host cell-agnostic.
- Simulator should consume final runtime render values (`X`, `Y`, `Scale`, `Rotation`, image path) and should not recompute placement from cell metadata.

2. Keep runtime/shared as placement authority.
- Cell-aware placement math stays in runtime/shared logic; host only renders resolved output.

3. Avoid layout-coupled transforms in simulator preview.
- Prefer `RenderTransform` over `LayoutTransform` for scaled/rotated runtime images to avoid WPF layout re-measure shrink artifacts.

4. Avoid designer-coupling in simulator render inputs.
- Do not introduce simulator dependency on designer-only footprint authoring fields.

4. Never place host-view concerns in shared:
- No WPF visual overlay rendering primitives in `Storyboard.Shared`.
- No direct viewmodel or control dependencies crossing host boundaries.

5. Never duplicate business rules across hosts:
- If both hosts need the same movement decision or coordinate-selection behavior, extract once to `Storyboard.Shared` and consume via adapters.

## 4.2 Cross-Boundary Implementation Gate

Before implementing any movement-grid slice, classify each change item as one of:

1. Shared rule/contract.
2. Designer-only authoring UX.
3. Simulator-only host UX.

Acceptance gate per slice:

1. Shared behavior must be unit-tested in shared or host tests that target shared contracts.
2. Host projects should only add adapters and UX-specific logic around shared behavior.
3. Architecture separation guardrail tests must remain green.

## 5. Baseline Semantics (Locked For v1)

1. Movement applies only to game objects currently discoverable in the active room scope.
2. Non-room objects are not valid targets for room movement actions.
3. `IsMovable` is the canonical movement feature flag.
4. Inventoriable objects should default to movable.
5. Non-inventoriable objects may still be movable.
6. Movement input uses direction and distance.
7. Candidate direction set: `N`, `E`, `S`, `W`, `NE`, `NW`, `SE`, `SW`.
8. Movement model is grid-based for v1.
9. Perceived smoothness is a host rendering concern guided by runtime-provided visual transition hints.

## 5.1 Movement Model Direction (Locked For v1)

v1 lock:

1. Support grid movement only.
2. Do not implement arbitrary distance movement in v1.
3. Keep arbitrary model as out-of-scope future expansion candidate if a proven gameplay gap emerges.

Rationale:

1. Input: direction + number of squares.
2. Distance unit is always grid-square count.
3. Grid cell size is producer-configurable.
4. Deterministic movement is better for puzzle-oriented design and replay validation.
5. Visual smoothness can be achieved by host rendering hints without adding a second movement semantics model.

## 5.6 Movement Visual Transition Hint (Locked For v1)

1. Runtime carries a non-authoritative visual transition hint for host rendering.
2. Proposed v1 enum values:
- Quantum
- Slow
- Medium
- Fast
3. Hint controls only presentation timing/animation style, not movement result semantics.
4. Hosts may choose equivalent presentation if exact animation implementation differs.
5. Runtime remains source of truth for final object position/state regardless of animation.

## 5.6.1 Deferred Evolution - Action-Scoped Transition Hint Chooser

1. Future expansion may allow action-level script evaluation to choose transition hint dynamically per mutation attempt.
2. This chooser is action-scoped and evaluated in the execution context of the subject object/mutation, not as an object-only static image chooser.
3. Request carries caller-suggested hint; mutation evaluator may preserve or override and returns the effective hint in result.
4. Host rendering should consume result hint as authoritative effective presentation guidance.
5. No interface changes should be required for this evolution because hint exists on both request and result mutation DTOs.

## 5.8 Stack Visual Indicator (Proposed For v1)

1. Stacked objects should render with progressive scale reduction to provide a clear visual depth cue.
2. Default behavior should be driven by project-level settings.
3. Objects may optionally override stack-visual settings per object.
4. Runtime should compute final render scale from stack depth and configured scale settings.
5. Host continues consuming final render scale through existing update payloads (no new render API method required).

Suggested baseline formula:

1. EffectiveScale = BaseScale * (StackScaleStep ^ StackDepth)
2. StackDepth is zero for base object in a stack, then increments upward.
3. Apply a minimum clamp to avoid unreadably small icons.

Suggested v1 defaults:

1. Project StackScaleStep default: 0.92
2. Project MinStackScale default: 0.60
3. Per-object override: optional StackScaleStepOverride and optional MinStackScaleOverride

## 5.9 3D Cell-Volume Guidance (Forward-Compatible)

1. Conceptual model: treat room space as a 3D lattice of cell-sized cubes, not only a 2D grid.
2. Unit model: the same cell unit applies to X, Y, and Z semantics.
3. Height semantics: upward movement/stack height uses Z expressed in cell units.
4. Current-phase projection rule: as effective Z height increases, rendered icon scale decreases per stack-scale policy.
5. Future-facing guidance: planned parallax/multiplane camera behavior should consume the same runtime effective Z value (no separate depth system).
6. Contract safety: this guidance adds conceptual/semantic alignment only and does not expand current v1 runtime-host payload surface by itself.

## 5.7 Movement Outcome Payload Baseline (Locked For v1)

1. Successful movement and rotation updates flow through the existing room object update channel (no new dedicated render callback API).
2. For successful movement updates, include both from and to location values in the runtime outcome/update payload.
3. For failed movement or rotation with no visual state change, do not emit an extra render update.
4. Action result codes and diagnostics remain the source for no-op failure explanation.

## 5.2 Grid Configuration Baseline (Locked For v1)

1. Project defines default room movement grid.
2. Room may optionally override project grid settings.
3. Initial practical target guidance for an 800x600 play surface:
- 40x40 cell size -> 20 columns x 15 rows.
4. Grid should remain coarse enough for usability while dense enough for puzzle design.
5. Grid cell dimensions are data, not hard-coded constants.

## 5.5 Designer Grid Authoring Assist (Locked For v1)

1. Room designer supports snap-to-grid placement mode for object positioning.
2. Snap mode affects drag/drop and direct placement operations in the room canvas.
3. Room designer supports a toggleable visual grid overlay.
4. Overlay and snapping must use the same effective grid settings that runtime movement uses.
5. Designer aids must not alter clean export contract data unless explicit room/object coordinates are already part of authored content.

## 5.3 Active Item Concept (Locked For v1)

1. Add an optional runtime "active item" focal object used as fallback for object resolution.
2. In v1, active item fallback applies to in-room movement commands only.
3. Active item is runtime session state, not authored project content.
4. Host UX for choosing active item is host-specific, but runtime-host contract must expose enough object identity and location data for reliable selection.

## 5.4 Host/Runtime Selection Contract (Locked For v1)

Runtime -> Host communication:

1. Runtime reports active item state through the existing host update channel.
2. No new dedicated active-item callback method is introduced.

Host -> Runtime selection input:

1. Support both select-by-object-id and select-by-point methods in the shared host interface.
2. Simulator v1 uses command text and point selection paths; object-id path remains supported but unused by simulator for now.
3. Point selection request uses pixel coordinates; runtime resolves selection from current room context.
4. Point selection chooses topmost selectable object at that position in v1.
5. No candidate-list API in v1.

Selection outcomes:

1. If point or object-id selection fails validation, keep existing active item unchanged and return a specific failure result code.
2. If the active item becomes invalid due to runtime state changes, auto-clear it.

## 6. Decision Ledger (Locked Snapshot 2026-07-21)

D-01 Inventoriable implies movable strictness:

1. Locked: author override.
2. Inventoriable auto-enables movable by default, but producer may turn movable off.

D-02 Runtime variable contract for movement:

1. Locked: emit runtime boolean isMovable for movable-capable objects.
2. Designer sets default value; runtime actions/scripts may toggle it.
3. isMovable exists only when the movable feature is enabled.

D-03 Command/action integration model:

1. Locked: use explicit new room action types.
2. Locked names: MoveRoomObjectOnGrid, RotateRoomObjectOnGrid, and StackRoomObjectOnAnother.

D-04 Room target constraints:

1. Locked: in-room movement targets direct room children only.
2. Locked: explicit command ambiguity requires clarification; active item is not used as tie-break for explicit ambiguous noun matches.

D-05 Movement mode strategy:

1. Locked for v1: implement Grid-only as the single movement model.
2. Arbitrary distance model is deferred and out of current scope.

D-06 Grid settings contract:

1. Locked: square grid model based on single CellSize.
2. Locked: room override may use same or finer CellSize only, and finer value must be an exact divisor of project CellSize.
3. Locked UX: designer presents valid CellSize dropdown values and echoes resulting grid columns x rows.

D-07 Directional behavior and path policy:

1. Locked: diagonal movement enablement is configurable at project level with optional room override.
2. Locked: multi-cell movement evaluates path stepwise.
3. Locked: partial movement support is per-action via AllowPartialMove (default false).

D-08 Active item fallback policy:

1. Locked: active item fallback applies only to in-room movement commands in v1.
2. Locked precedence: explicit object reference, then active item fallback, then clarification/failure.

D-09 Host -> runtime active-item selection API:

1. Locked: support both object-id and point selection APIs plus clear API.
2. Locked names: SetActiveRoomObjectById, SetActiveRoomObjectAtPoint, ClearActiveRoomObject.
3. Locked: SetActiveRoomObject action name for command/action path.

D-10 Coordinate selection semantics:

1. Locked: host sends point coordinates in pixel space.
2. Locked: runtime resolves hit target in current room context.
3. Locked: topmost selectable object wins in v1.

D-11 Active item reporting:

1. Locked: reuse existing host update channel (no dedicated callback API).
2. Locked: invalid select requests keep current active item unchanged and return specific result codes.
3. Locked: room-scope active item; room changes clear active item.

D-12 Snap-to-grid behavior policy:

1. Locked: snap-to-grid default is on.
2. Locked: snap/overlay preference persistence is app/session-level state.

D-13 Grid overlay behavior policy:

1. Locked: overlay default is off with quick toolbar toggle.

D-14 Designer-state persistence boundary:

1. Locked: snap/overlay preferences remain out of clean export.

D-15 Shared-vs-host placement lock:

1. Locked: shared owns reusable grid, collision, movement, and resolution logic.
2. Locked: hosts own UX and interaction surfaces only.

D-16 Movement visual hint contract:

1. Locked v1 set: Quantum, Slow, Medium, Fast.
2. Locked: hint is presentation-only and does not alter gameplay result semantics.

D-17 Visual hint precedence:

1. Locked: Project default -> Room override -> Action override.

D-18 Movement/rotation result-code behavior:

1. Locked: detailed move and rotate result-code sets.
2. Locked: if code-specific echo is missing, fallback to generic success/failure echo.
3. Locked move result codes: MovedFullDistance, MovedPartialDistance, BlockedByCollision, OutOfBounds, TargetNotMovable, RuntimeMovableDisabled, NoResolvedTarget, InvalidDirection, InvalidDistance, InvalidConfiguration.
4. Locked rotate result codes: Rotated, BlockedByCollision, OutOfBoundsAfterRotation, TargetNotMovable, RuntimeMovableDisabled, NoResolvedTarget, InvalidRotationStep, InvalidFacing, MissingOrientationData, InvalidConfiguration.
5. Locked ordering principle: success first, world-state blockers second, resolution/target failures next, then input/configuration failures.
6. Locked: failed move/rotate with no visual state change emits no extra render update.

D-19 Rotation and orientation model:

1. Locked: single rotate action supports both turn mode and face mode.
2. Locked: turn mode supports multiples of 45 degrees.
3. Locked: command text numeric degree input rounds to nearest 45 with warning.
4. Locked: designer-guided authored degree values must be exact multiples of 45.
5. Locked: face-mode path ties default to clockwise unless explicit preference provided.
6. Locked: facing supports 8 directions; collision orientation is cardinal-only.
7. Locked payload disambiguation: rotate action payload includes explicit rotate mode (`Turn` or `Face`) so semantics are action-bound, not inferred from command wording.
8. Locked payload validation by mode:
- `Turn` mode requires turn-degree input and rejects missing/invalid degree values.
- `Face` mode requires facing-direction input and rejects missing/invalid facing values.
- Runtime execution mapping should honor only the mode-selected input channel.

D-25 Direction shortcut for movement actions:

1. Locked: movement actions do not introduce a serialized or producer-authored special direction token for heading continuation.
2. Locked fallback behavior: when a movement action executes without an explicit direction argument, runtime may fall back to the resolved active target object's HeadingDirection.
3. Locked producer-vocabulary rule: command verbs/direction words remain producer-controlled; engine-internal fallback logic must not require special authored keywords.
4. Locked failure behavior: if explicit direction is absent and fallback heading is missing/invalid, action fails with InvalidDirection.

D-20 Collision, stack, and height model:

1. Locked: object size/shape/orientation data applies to all objects, including non-movable objects.
2. Locked: rectangular footprint with orientation support.
3. Locked: StackOrder model.
4. Locked semantics: StackOrder = 0 means hard blocker/non-stackable; StackOrder > 0 participates in stacking; lower value stacks above higher value; equal values stack either way.
5. Locked: same stack/collision rule applies for destination checks and mid-path checks.
6. Locked: non-movable objects still participate in support/stack and collision evaluation.
7. Locked: add object height property (object-level authored), and runtime computed HeightInRoom reflects cumulative stack height.
8. Locked naming: runtime cumulative field is HeightInRoom only.

D-21 Host render contract clarifications:

1. Locked: host consumes final render transform fields (x, y, scale, rotation).
2. Locked: runtime maintains and diagnoses object-orientation and icon-alignment components internally; host does not require front/back metadata in v1 payload.

D-24 HeightInRoom and render z-order relationship:

1. Locked: HeightInRoom is a runtime-computed vertical-stacking semantic and is not a replacement for authored render z-order.
2. Locked: rendering order logic must be a composite that preserves authored z-order intent while allowing HeightInRoom to influence layered depth for stacked scenarios.
3. Locked: when both factors apply, authored z-order remains the base ordering channel and HeightInRoom is an additive influence for stack depth disambiguation.
4. Locked: diagnostics and test assertions should expose both authored z-order and effective runtime layering inputs to prevent regressions.

D-26 Cell-volume depth model guidance:

1. Locked guidance: room positioning semantics use a conceptual 3D cell-volume model with consistent cell units across X, Y, and Z.
2. Locked guidance: HeightInRoom remains the runtime depth/height signal for stacked rendering behaviors in current phases.
3. Locked guidance: future parallax/multiplane camera work should be derived from the same effective height/depth signal rather than introducing a parallel depth channel.

D-22 Stack visual scaling policy:

1. Locked: add project-level stack scaling settings for visual depth cue.
2. Locked: allow optional per-object override for stack scaling behavior.
3. Locked defaults: StackScaleStep = 0.92 and MinStackScale = 0.60.
4. Locked: runtime-side formula and clamping behavior drive deterministic render scale output.
5. Deferred: lock allowed override ranges after visual calibration pass in designer/simulator.
6. Locked: this is visual-only and independent from gameplay collision/height semantics.

D-23 Defaults, parser capability, and session persistence:

1. Locked defaults (applied when fields are missing):
- Movable defaults true for inventoriable objects and false for non-inventoriable objects.
- StackOrder = 0.
- FootprintWidthCells = 1 and FootprintHeightCells = 1.
- FootprintOrientation = N.
- ObjectHeightUnits = 1.
- Project CellSize = 40.
- StackScaleStep = 0.92 and MinStackScale = 0.60.
2. Locked parser capability scope:
- Command vocabulary remains producer-defined.
- Runtime/parser must support numeric distance extraction, numeric degree extraction, and 8-direction token extraction.
3. Locked diagnostics surface:
- Runtime outcomes always carry diagnostics.
- Simulator reuses existing diagnostics/output surfaces.
- Designer reuses existing validation/reporting surfaces.
- Degree rounding warning is runtime execution diagnostic (not authoring validation error).
4. Locked future save/load behavior for active item:
- Persist active item state.
- On restore, keep it only if still valid in current room scope; otherwise auto-clear.

## 6.1 Naming Locks

1. MoveRoomObjectOnGrid
2. RotateRoomObjectOnGrid
3. StackRoomObjectOnAnother
4. SetActiveRoomObject
5. SetActiveRoomObjectAtPoint
6. SetActiveRoomObjectById
7. ClearActiveRoomObject

## 7. Phased Plan

## Phase F - Foundation Contract Realignment (Merged Implementation Tranche)

Status: Complete (2026-07-23)

Scope note:

1. This is an execution umbrella that merges Phase 1, Phase 2.5, and Phase 2.6 work into one implementation pass.
2. Phase 1, Phase 2.5, and Phase 2.6 remain below as detailed specification sections and acceptance anchors.

Execution direction:

1. Implement Phase 1, Phase 2.5, and Phase 2.6 as one merged foundation tranche while preserving their separate headings for traceability.
2. Treat this as a single gate before Phase 3 runtime movement implementation.
3. Keep runtime player-movement behavior work out of this tranche except for contract/model/mapping scaffolding.
4. Apply one-time sample project correction to canonical field names/shapes during this tranche; avoid open-ended dual-shape maintenance.
5. Any compatibility fallback added for transition must be bounded and removable after sample/project correction closure.

Merged tranche order:

1. Semantic alignment and naming lock (authored vs effective ownership, height/pinning intent, occupancy authority and drift policy).
2. Model/contract/JSON grouped-shape implementation across authored save, clean export, and runtime bootstrap mappings.
3. Defaults, migration/update behavior, and deterministic derived-occupancy metadata/recompute policies.
4. Regression, snapshot, and compatibility validation closure for the entire foundation surface.

Merged tranche exit gate:

1. No unresolved naming/ownership ambiguity remains.
2. Grouped JSON and mapping behavior are stable and test-covered.
3. Derived occupancy policies are deterministic and diagnostics-ready.
4. Runtime Phase 3 can begin without contract or migration churn risk.
5. One-time sample corrections are complete and no indefinite migration burden remains.

## Phase 0 - Definition Lock and Acceptance Criteria

Status: Complete (2026-07-21)

Goals:

1. Lock core movement, grid, collision, rotation, active-item, and host-boundary decisions.
2. Finalize naming and precedence policies.
3. Confirm v1 scope boundaries and explicit post-v1 follow-up items.

Deliverables:

1. Updated this plan with locked decisions.
2. Acceptance checklist with concrete examples.
3. Command target-resolution examples showing explicit noun vs active-item fallback behavior.
4. Room-designer examples covering snap placement and overlay on/off behavior.

Exit criteria:

1. No unresolved core semantics.
2. Test matrix approved for implementation phases.

## Phase 1 - Grid Contract and Model Scaffolding

Goals:

1. Add `IsMovable` to authored `GameObject` model.
2. Add project-level default movement-grid settings.
3. Add room-level optional movement-grid override settings.
4. Add serializer/export/runtime DTO coverage as additive fields.
5. Add runtime session model fields for active-item identity and optional grid position metadata.
6. Extract shared grid/movement calculation utilities used by both hosts.
7. Add shared movement visual hint contract types and defaults.
8. Add shared movement outcome contract carrying explicit from/to location fields.
9. Add shared stack-visual scaling settings contract and runtime scale calculator.

Key tasks:

1. Extend model properties and mapping paths.
2. Define effective grid resolution (room override -> project default).
3. Apply compatibility defaults when fields are absent in legacy data.
4. Define runtime contract DTO(s) for active-item reporting and host selection commands.
5. Define host adapter interfaces so Designer and Simulator consume shared behavior without cross-host references.
6. Define runtime action/session fields that carry movement visual hints without embedding host animation implementation details.
7. Define runtime->host movement DTO field requirements for source/destination coordinates and identity.
8. Define project-level and optional object-level stack scale settings with deterministic inheritance rules.

Exit criteria:

1. Legacy projects load with stable behavior.
2. Effective grid settings resolve deterministically.
3. New saves/export include movement/grid fields without schema break.
4. Runtime-host contract can express active-item set/clear and observe current active item.
5. Shared code contains no host-specific UI dependencies.
6. Movement visual hints are available to both hosts through shared contracts.
7. Movement outcome contracts expose explicit from/to data for host rendering and diagnostics.
8. Stack visual scaling is deterministic and surfaced via existing final scale render fields.

## Phase 2 - Authoring UX and Validation

Boundary note:

1. Designer phase scope is authoring-time UX and validation only.
2. Player-time movement interaction (selection, move invocation, resolution feedback) belongs to runtime plus simulator host UX.
3. Keep room-designer visual complexity constrained; prefer validation-rule diagnostics for questionable layouts over heavier editor interaction logic.

Goals:

1. Add `Movable` control to object settings.
2. Implement auto-selection behavior when `Inventoriable` is enabled (per D-01).
3. Add project and room editors for movement-grid settings.
4. Add validation/diagnostics for illegal combinations and invalid grid dimensions.
5. Add authoring-time configuration and validation support for runtime active-item behavior (designer does not provide player-time active-item interaction UX).
6. Add room-canvas snap-to-grid toggle and placement behavior.
7. Add room-canvas grid overlay toggle and rendering behavior.
8. Add simulator-host UX workflows for active-item selection and movement invocation using shared contracts.
9. Add authoring UX for selecting movement visual hint defaults/overrides (per lock decision).
10. Add authoring UX for project stack-scale defaults and optional per-object overrides.
11. Prefer warning/diagnostic surfaces for questionable layout states instead of adding high-complexity room-canvas behaviors when equivalent safety can be achieved through validation.

Key tasks:

1. Update dialog/viewmodel update paths.
2. Add tests for object, project, and room edit flows.
3. Add validation rule(s) for locked policy and grid bounds.
4. Add simulator host tests for player-time active-item selection and reflecting runtime-reported active state.
5. Add room-designer canvas interaction updates for snapping drag/drop placements.
6. Add overlay rendering and toggle state management with boundary-safe persistence.
7. Keep host UX behavior thin by delegating core movement math and selection rules to shared services, with simulator owning player-time interaction surfaces.
8. Keep visual-hint authoring and display wiring host-specific while honoring shared hint values.
9. Keep stack visual scaling authored in designer and computed in runtime with no host-specific scale math required.
10. Add focused layout validation rules for overlap/stackability/cell-occupation anomalies and expose actionable diagnostics to producers.

Exit criteria:

1. Producer workflow is clear and deterministic.
2. Feature relationship policy is enforced consistently.
3. Grid settings are discoverable and safely constrained.
4. Simulator player-selection flow is functional without requiring excessive clarification loops.
5. Snap placement and overlay controls are discoverable and stable in room designer UX.
6. Both hosts demonstrate consistent movement/selection outcomes for identical shared inputs.
7. Both hosts honor movement hint semantics consistently (within host UX freedom).
8. Stacked object scale cues are visually consistent across hosts for identical runtime state.
9. Questionable room layouts are surfaced through deterministic validation diagnostics without increasing room-canvas interaction complexity.

## Phase 2.5 - Grouped Model/JSON Reorganization (Pre-Runtime Gate)

Goals:

1. Reorganize movement and appearance-related object data into logical nested subsections in designer project JSON.
2. Mirror the same logical grouping in clean export/runtime JSON contracts.
3. Omit safe default/null values to reduce JSON noise and improve readability.
4. Perform one-time in-place migration of sample project files to the grouped shape.
5. Do not implement legacy/backward-compatible loading for the old flat shape in this feature slice.

Key tasks:

1. Define subsection boundaries and names (for example: movement, footprint/orientation, stack/height, visual overrides).
2. Update project save/load mappings and clean export/runtime bootstrap mappings to grouped sections.
3. Define deterministic per-field omit-default/null behavior and lock serializer expectations.
4. Migrate sample project files and refresh fixtures/snapshots.
5. Add tests asserting grouped JSON shape and omission behavior for project, clean, and runtime bootstrap flows.

Exit criteria:

1. Grouped subsection shape is stable across designer save, clean export, and runtime bootstrap.
2. Default/null omission behavior is deterministic and test-covered.
3. Sample files are migrated and validated with no old-shape fallback path.
4. Runtime movement implementation (Phase 3+) does not start until this phase is complete.

## Phase 2.6 - Property Intent Clarification and Ownership Split

Goals:

1. Separate designer-authored ordering intent from runtime-effective ordering outcome.
2. Clarify property ownership boundaries for stacking/height/render-order semantics.
3. Lock deterministic runtime-init conflict-resolution behavior for illegal authored startup stacks.
4. Align naming across designer model, persisted JSON, clean export, and runtime contracts.
5. Support authored non-floor starting height for objects (including non-stackable objects).
6. Plan for future gravity/falling by allowing authored pin-to-height behavior.
7. Prevent drift when exposing cell-occupation data for diagnostics/validation.

Key tasks:

1. Introduce authored ordering field (working name: `authoredRenderOrder`) in authored data.
2. Keep runtime-effective ordering as a separate field/concept (`renderZOrder` or explicit `effectiveRenderZOrder`).
3. Define and document precedence/ownership:
- Designer owns authored intent fields.
- Runtime owns effective computed fields.
4. Define deterministic init normalization algorithm and stable tie-break sequence for illegal startup stack order.
5. Define diagnostics contract for auto-resolved conflicts (stable codes/messages with object identity context).
6. Ensure authored source data is not mutated by runtime normalization.
7. Add bounded migration/update mapping rules for existing files that only contain `renderZOrder`, and remove long-term dual-shape dependence after one-time sample correction.
8. Define authored starting-height field semantics (working name: `authoredBaseHeightInRoom`).
9. Define authored height pinning semantics for future gravity/falling compatibility (working name: `isHeightPinned`).
10. Define occupancy-data authority and drift policy:
- Canonical authored source remains x/y plus anchor, footprint/orientation, and effective grid settings.
- Cell occupation is derived secondary data for diagnostics/validation convenience and is never an authoritative placement source.
11. Define occupancy recomputation timing policy:
- Recompute during room-edit mutations that affect occupancy inputs.
- Recompute as authoritative final pass during save before persistence.
- Recompute on load for verification and self-heal if persisted derived values mismatch.
12. Define mismatch handling and diagnostics policy:
- Runtime and validation logic operate on recomputed occupancy, not persisted cached occupancy.
- Persisted/recomputed mismatch emits deterministic diagnostics and rewrites derived occupancy from canonical inputs.
13. Define persisted occupancy derivation metadata policy (human-readable default):
- Persist occupancy cache with source-echo metadata describing authoritative derivation inputs (x, y, anchor mode, footprint size, orientation, effective cell size, and optional room bounds/version).
- Optional compact fingerprint/hash may be included for fast stale detection, but source-echo metadata is required for human diagnostics readability.
14. Define deterministic cell naming policy for diagnostics/validation references:
- Use letter-based identifiers in row-major order from top-left origin (for example A.A, B.A, ... Z.A, AA.A; next row A.B).
15. Lock how authored base height participates in runtime final z-order derivation:
- Runtime resolves stack relationships first.
- Runtime computes effective height in room using authored base height for non-stacked objects and as baseline where applicable.
- Runtime computes effective render order from resolved effective height, then deterministic tie-breakers.
16. Define deterministic interaction rules between stacking and pinning:
- Pinned objects keep authored base height regardless of future gravity/fall simulation.
- Non-pinned objects are eligible for future gravity/fall adjustments.
- Illegal startup overlaps at equal effective height resolve via deterministic tie-breakers plus diagnostics.

Exit criteria:

1. Property naming/intent matrix is locked and documented.
2. Serialization and mapping paths preserve authored-vs-effective separation end-to-end.
3. Runtime deterministic normalization behavior is test-covered and replay-stable.
4. Diagnostics coverage exists for startup auto-resolution outcomes.
5. Downstream movement/stack phases can proceed without property-role ambiguity.
6. Authored non-floor starts and pinning behavior are contract-defined and test-plannable.
7. Occupancy-derived data behavior is locked with explicit authority, recompute timing, and drift-protection rules.

## Phase 3 - Runtime Grid Movement Core

Status: Active (started 2026-07-23)

Goals:

1. Introduce room-scoped movement eligibility checks.
2. Block movement for non-movable targets.
3. Execute direction + distance movement in grid squares.
4. Ensure movement operation does not leak beyond current-room rules.
5. Add runtime active-item state lifecycle management.
6. Emit/propagate movement visual hints with movement outcomes where applicable.
7. Emit runtime movement outcomes with explicit source/destination payload data.
8. Add relation-based stack attempt support (`StackRoomObjectOnAnother`) where runtime derives direction/distance from resolved subject and target object state.

Key tasks:

1. Add runtime evaluator/service logic in shared layer.
2. Add robust failure diagnostics/result codes.
3. Implement effective-grid resolution and movement-step evaluation.
4. Preserve existing container/inventory behavior unless explicitly modified.
5. Implement active-item resolution pipeline and invalidation rules.
6. Ensure movement result contracts include enough data for hosts to animate from source to destination with selected hint.
7. Ensure blocked/failed movement outcomes carry deterministic source/destination semantics.
8. Define and implement stack-attempt request/result contracts that identify subject and target objects explicitly and carry derived vector details.
9. Implement deterministic stack anchor and tie-break rules for target-footprint cases involving multiple candidate objects.

Exit criteria:

1. Runtime passes focused movement and regression tests.
2. Existing container transfer scenarios remain green.
3. Grid movement is deterministic for identical inputs.
4. Active-item fallback behavior is deterministic and well-diagnosed.
5. Visual-hint data is deterministic and contract-stable.
6. From/to movement payload data is deterministic, complete, and replay-safe.
7. StackRoomObjectOnAnother runtime outcomes are deterministic and include resolved subject/target plus derived movement vector diagnostics.

## Phase 4 - Command and Interaction Integration

Goals:

1. Expose movement via command/action authoring path (per D-03).
2. Ensure command target resolution honors room scope and ambiguity policies.
3. Integrate active-item fallback into command processing resolution rules.
4. Expose relation-oriented stacking command/action path for explicit subject-target formulations (for example, object-on-object phrasing).

Key tasks:

1. Add/update action payload schema and editors if new action type is chosen.
2. Add runtime execution path and result-code registration.
3. Add token/echo provider coverage if needed.
4. Ensure command parser/dispatcher accepts direction + square-distance arguments for movement.
5. Ensure parser/dispatcher supports active-item selection commands and coordinate selection commands (if D-09 locks Option C).
6. Ensure command/action flow can set or inherit movement visual hint values.
7. Ensure parser/dispatcher supports two-object stack intent extraction (subject + target) and routes it to StackRoomObjectOnAnother.
8. Ensure stack action payload and diagnostics expose resolved target, derived direction/distance, and deterministic failure reasons.

Exit criteria:

1. Movement is authorable, executable, and debuggable.
2. Guardrail tests cover registration and payload integrity.
3. Ambiguity and active-item fallback behaviors are stable under replay tests.
4. Movement visual hints round-trip and surface correctly through runtime-host interaction.
5. StackRoomObjectOnAnother command/action flow is authorable, executable, and replay-stable for duplicate-name and overlap-heavy rooms.

## Phase 3 Execution Checklist (3A-3C)

The following checklist is the execution packet for current active runtime work.

### Phase 3A Kickoff Checklist (2026-07-24)

Objective:
1. Re-enter runtime implementation with the smallest safe additive slice: typed contracts plus gateway surface only.

Implementation sequence:
1. Add or verify shared move-attempt request/result contracts in `Storyboard.Shared`:
- request includes target resolution handle, direction, distance in cells, and policy flags (for example partial-move allow).
- result includes stable machine-readable code, human-readable message, from/to location payload, and diagnostics collection.
2. Add or verify shared rotate-attempt request/result contracts in `Storyboard.Shared`:
- request includes target resolution handle plus rotate mode inputs (turn/facing data).
- result includes stable code/message, from/to orientation/location payload, and diagnostics.
3. Add or verify shared stack-attempt request/result contracts in `Storyboard.Shared`:
- request includes explicit subject/target references and stack-policy options.
- result includes resolved subject/target identity, derived vector diagnostics, and deterministic result code.
4. Extend runtime mutation gateway interface(s) with additive methods:
- TryMoveRoomObjectOnGrid
- TryRotateRoomObjectOnGrid
- TryStackRoomObjectOnAnother
5. Ensure all new contracts are DTO-only and host-agnostic:
- no WPF types
- no Designer/Simulator project references
6. Register/verify result-code mapping paths for new outcomes in shared runtime result registries.

Immediate test closure for 3A:
1. Add shared contract compile/shape tests for request/result defaults and required fields.
2. Add gateway surface tests ensuring additive methods are reachable and source-compatible for current callers.
3. Add deterministic result-code presence tests (code always set even at low diagnostics verbosity).

Phase 3A stop gate:
1. No movement behavior execution logic beyond surface plumbing (behavior starts in 3B).
2. Build passes.
3. Focused runtime guardrail test filter passes.
4. No new architecture-separation violations.

Validation commands for kickoff completion:
1. `dotnet build .\\StoryboardDesigner.slnx`
2. `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`

### Phase 3A - Contracts and Mutation Gateway Surface

Status: Complete (2026-07-24)

Completion note:
1. Added typed move/rotate/stack attempt request/result contracts in shared runtime context.
2. Extended runtime mutation gateway with additive move/rotate/stack attempt methods.
3. Implemented deterministic Phase 3A surface behavior returning stable InvalidConfiguration failure codes until Phase 3B evaluator wiring.
4. Added focused shared tests for contract/result-code presence and gateway additive surface behavior.

Scope:

1. Add typed move/rotate attempt request and result contracts in shared runtime.
2. Add typed stack-attempt request and result contracts for StackRoomObjectOnAnother.
3. Extend runtime mutation gateway with additive methods for move, rotate, and stack attempts.
4. Include deterministic diagnostics fields and from/to payload fields in all movement outcomes.

Completion checks:

1. Contracts compile with no host-project coupling.
2. Existing gateway consumers remain source-compatible.
3. Result-code mapping for move/rotate/stack is available through shared registry paths.

### Phase 3B - Shared Runtime Evaluator and Deterministic Resolution

Status: In progress (B1 move evaluator slice completed 2026-07-24)

Checkpoint note (2026-07-24):
1. Implemented initial move evaluator behavior in shared session mutation gateway.
2. Added deterministic validation and resolution for direction token, distance, target resolution, movable gate, collision blocking, out-of-bounds blocking, and AllowPartialMove semantics.
3. Added focused shared tests covering unresolved target, target-not-movable, full-distance success, collision fail without partial, and collision partial-success behavior.
4. Verified focused shared tests, solution build, and focused runtime guardrail suite all pass.

Sequencing note (2026-07-24):
1. Continue move end-to-end first as pure translation behavior with occupied destination treated as collision.
2. Do not introduce implicit move-to-stack fallback during move hardening.
3. After rotate and explicit stack paths are stable, reconsider optional move-to-stack fallback as a separate integration slice.

Scope:

1. Implement grid move evaluator with room-scope, movable checks, bounds, and stepwise collision handling.
2. Implement rotate evaluator with deterministic orientation and collision/out-of-bounds handling.
3. Implement stack-attempt evaluator that resolves subject and target, derives direction/distance, and applies deterministic tie-break rules when target footprint overlaps multiple candidates.
4. Emit deterministic diagnostics including derived-vector details for stack attempts.

Completion checks:

1. Deterministic outcomes for identical runtime snapshots and identical requests.
2. Failed operations do not produce unintended visual/update churn.
3. Container/inventory behavior remains unaffected by default.

### Phase 3C - Command Processor Integration and Recompute Formalization

Status: Complete (2026-07-26)

Completion note:
1. Runtime action wiring for MoveRoomObjectOnGrid, RotateRoomObjectOnGrid, and StackRoomObjectOnAnother remains active and stable.
2. Recompute debug diagnostic codes are formalized and emitted in High diagnostics mode.
3. Diagnostics are host-visible for debugging and explicitly treated as non-contract signals.
4. Focused command-processor and shared recompute regression tests are green.

Scope:

1. Wire new runtime evaluator paths into action execution for MoveRoomObjectOnGrid, RotateRoomObjectOnGrid, and StackRoomObjectOnAnother.
2. Keep action payload and outcome metadata compatible with existing diagnostics and echo-script paths.
3. Formalize recompute conflict diagnostic codes and include them alongside existing text diagnostics.
4. Run recompute after successful mutations and preserve diagnostics-level gating behavior.

Completion checks:

1. Focused command-processor regression tests are green.
2. Replay output remains stable for unchanged scenarios.
3. High-diagnostics mode exposes sufficient details for runtime debugging of movement and stack attempts.

## Phase 3 Runtime Deepening Lock-Off Questions

Resolve these before deeper runtime implementation to avoid churn:

1. SQ-01 Subject/target ambiguity policy for StackRoomObjectOnAnother:
- If subject or target name resolves to multiple room objects, should runtime always require clarification, or should it apply deterministic auto-selection in defined contexts?

2. SQ-02 Stack anchor selection policy:
- When target footprint includes multiple stacked objects, which anchor is authoritative (topmost effective height, authored target id only, or deterministic best-fit rule)?

3. SQ-03 Same-cell occupancy tolerance:
- Should stack action require full subject footprint containment on target footprint, or allow partial overlap with configurable threshold?

4. SQ-04 Auto-nudge policy:
- If exact target alignment is blocked, do we allow deterministic nearest-cell nudge search, or fail immediately with explicit diagnostics?

5. SQ-05 Collision participation rule while stacking:
- During stack attempts, do non-target objects at the destination always block, or can specific object classes be ignored as pass-through/support-only?

6. SQ-06 Height resolution ownership during stack action:
- Does stack action directly set authored baseline-like runtime values, or only invoke placement then rely entirely on recompute for effective height/order finalization?

7. SQ-07 Partial success semantics:
- For multi-cell movement requests, is partial movement allowed for move action only, or also for stack attempts that can close some but not all required displacement?

8. SQ-08 Rotation interaction with stacking:
- If rotation changes footprint and causes overlap on an otherwise valid stack location, should rotate auto-fallback to previous orientation or fail hard?

9. SQ-09 Result-code granularity lock:
- Do we need dedicated stack-specific result codes (for example TargetNotStackable, AmbiguousTarget, DerivedVectorInvalid) beyond generic move/rotate failures?

10. SQ-10 Diagnostics contract lock:
- Should movement/stack diagnostics include stable machine-readable codes plus human text in all modes, or only in medium/high diagnostics levels?

11. SQ-11 Active-item participation for stack commands:
- If command omits explicit subject, may active item supply subject for StackRoomObjectOnAnother under D-08 precedence, or must stack always require explicit subject and target?

12. SQ-12 Cross-host rendering handoff:
- Do hosts consume only runtime from/to payload and hint fields, with no host-side recomputation of derived vectors, as a strict contract?

### Resolved Lock Decisions (2026-07-23)

1. SQ-01 locked:
- Ambiguous subject or ambiguous target requires clarification in v1.

2. SQ-02 locked:
- Anchor is the explicitly resolved target object.
- Placement policy is support-surface-first on that target footprint.
- If direct space exists on the stated landing object, succeed.
- If direct space does not exist on the stated landing object, fail.

3. SQ-03 locked:
- Stack placement requires full subject-footprint containment on available target support cells.

4. SQ-04 locked:
- No auto-nudge in v1; blocked exact placement fails immediately with deterministic diagnostics.

5. SQ-05 locked:
- All non-target occupants block stack placement in v1.

6. SQ-06 locked:
- Stack action applies placement intent only; recompute remains authoritative for final effective height and effective render order.

7. SQ-07 locked:
- Partial movement is allowed only for MoveRoomObjectOnGrid when explicitly enabled.
- StackRoomObjectOnAnother is all-or-fail in v1.

8. SQ-08 locked:
- Invalid post-rotation placement fails hard; prior orientation/state remains unchanged.

9. SQ-09 locked:
- StackRoomObjectOnAnother uses dedicated stack-specific result codes.

10. SQ-10 locked:
- Outcomes always include stable machine-readable code plus human-readable text.
- Diagnostics level controls verbosity, not code presence.

11. SQ-11 locked:
- Active-item fallback may supply subject only when subject is omitted.
- Target remains explicit for StackRoomObjectOnAnother.

12. SQ-12 locked:
- Hosts consume runtime outcome payloads and do not recompute derived placement/vector intent.

13. Additional lock (support footprint size rule):
- Larger-on-smaller support placement is disallowed for support-surface placement contexts.
- If subject footprint exceeds available support footprint, fail deterministically with explicit result code.

## Phase 5 - Hardening and Regression Sweep

Goals:

1. Validate against representative sample projects.
2. Ensure replay and architecture guardrail suites stay green.
3. Document behavior examples for producers.
4. Validate latest multi-leg movement and movement-restriction behavior without broad redundant suite expansion.

Key tasks:

1. Re-run focused movement runtime tests, emphasizing multi-leg and movement restriction paths.
2. Re-run focused runtime guardrail/replay filters already used by this plan.
3. Update producer-facing notes with the final lock decisions (including D-22 bounds and transition-hint chooser deferral).
4. Run one representative sample scenario to verify no behavioral regression in practical authoring/runtime flow.

Lean validation profile for closeout:

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\Storyboard.Shared.Tests\Storyboard.Shared.Tests.csproj --filter "RuntimeMoveRoomObjectOnGridActionTests|RuntimeScopeMutationGatewayPhase3ATests"`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`

Exit criteria:

1. No known regressions in baseline runtime flows.
2. Multi-leg and movement-restriction coverage remains green in focused runs.
3. Plan status updated to complete with deferred transition-hint chooser tracked post-v1.

## 8. Test Strategy

Mandatory gates per meaningful implementation slice:

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`
4. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`

Proposed new targeted tests:

1. GameObject feature contract tests for movable defaults and override semantics.
2. Project/room effective-grid settings resolution tests.
3. Serialization/export round-trip tests for `IsMovable` and grid settings.
4. Runtime movement evaluator tests for room-scope enforcement.
5. Runtime grid movement tests for orthogonal and diagonal movement.
6. Runtime grid movement tests for blocked-path and out-of-bounds behavior.
7. Command processor tests for move success/failure result paths.
8. Command processor tests for StackRoomObjectOnAnother success/failure and ambiguity paths.
9. Ambiguous-name resolution tests for room movement targets.
10. Active-item resolution tests for explicit noun precedence over fallback.
11. Active-item invalidation tests when selected object becomes unavailable.
12. Host->runtime select-by-id contract tests.
13. Host->runtime select-by-coordinate contract tests, including tie-break behavior.
14. Runtime->host active-item reporting tests (set, update, clear).
15. Room-designer snap placement tests for drag/drop and direct placement operations.
16. Room-designer overlay tests for render alignment against effective grid settings.
17. Persistence-boundary tests proving snap/overlay preferences are excluded from clean export.
18. Architecture placement tests/guardrails preventing Designer-Simulator direct coupling during movement feature rollout.
19. Cross-host parity tests proving shared movement/grid rules produce consistent outcomes.
20. Movement visual hint contract tests covering defaults, explicit values, and compatibility handling.
21. Cross-host integration tests ensuring hint values are observable and mapped to host animations/transitions.
22. Runtime outcome/update tests asserting successful movement includes explicit source and destination fields.
23. Failed move/rotate tests asserting no extra render update is emitted when no visual state changes.
24. Stack visual scaling tests for depth-to-scale mapping, clamp behavior, and per-object override inheritance.
25. Cross-host visual parity tests for stacked render scaling using identical runtime snapshots.
26. Effective layering tests proving HeightInRoom influences stacked render depth while authored z-order remains the base ordering mechanism.
27. Stack action derived-vector tests proving computed direction/distance is deterministic for identical subject/target layouts.

## 9. Risks and Mitigations

1. Risk: semantic conflict between inventoriable and movable flags.
- Mitigation: lock D-01 early and enforce via validation + UX affordances.

2. Risk: contract drift across authored save vs clean export vs runtime bootstrap.
- Mitigation: implement additive field propagation in one slice and test round-trip.

3. Risk: breaking existing container transfer behavior.
- Mitigation: keep movement path separate first; integrate only after parity tests pass.

4. Risk: command ambiguity with duplicate room object names.
- Mitigation: define deterministic selection/ambiguity diagnostics before parser integration.

5. Risk: grid defaults and room overrides become difficult to reason about.
- Mitigation: use explicit effective-value presentation in UX and add dedicated precedence tests.

6. Risk: diagonal and multi-step semantics create hidden gameplay inconsistencies.
- Mitigation: lock D-07 early and test atomic vs stepwise behavior explicitly.

7. Risk: active-item fallback may create surprising command resolution.
- Mitigation: enforce explicit precedence rules (D-08) and provide clear diagnostics/echo feedback.

8. Risk: host/runtime contract mismatch for selection events and state reporting.
- Mitigation: lock D-09 to D-11 with explicit DTO contracts and compatibility tests in both directions.

9. Risk: coordinate-based selection can be unstable with dense object layouts.
- Mitigation: define deterministic tie-break ordering and expose selection diagnostics for host tooling.

10. Risk: designer overlay/snap may drift from runtime effective grid rules.
- Mitigation: centralize effective-grid resolution and reuse it in both runtime and designer paths.

11. Risk: overlay rendering impacts editor performance on large canvases.
- Mitigation: define efficient rendering strategy and provide visibility toggle defaults tuned for usability.

12. Risk: movement behavior divergence between designer and simulator implementations.
- Mitigation: keep movement/grid/selection core in shared and add parity tests across both hosts.

13. Risk: accidental cross-host coupling while wiring active-item UX.
- Mitigation: enforce D-15 boundary lock and keep host integrations behind shared contracts/adapters.

14. Risk: visual hint semantics drift between hosts and create inconsistent player perception.
- Mitigation: lock canonical hint meanings in shared docs/contracts and verify with cross-host behavior tests.

15. Risk: visual hints accidentally alter gameplay outcomes.
- Mitigation: enforce that hints are presentation-only and keep gameplay position resolution runtime-deterministic.

16. Risk: host animation/state drift when movement payload omits origin or destination.
- Mitigation: lock D-18 and require explicit from/to fields in every movement outcome contract.

17. Risk: host pixel-input to runtime-grid conversion drift causes inconsistent point selection and movement targeting.
- Mitigation: centralize conversion logic in shared runtime, keep host input as pixel-space contract, and validate with deterministic integration tests.

18. Risk: stack scale settings produce unreadable tiny icons or insufficient visual differentiation.
- Mitigation: provide bounded defaults (for example step and minimum clamp), preview in designer, and validate with representative stack heights.

19. Risk: host-side custom scaling could diverge from runtime-computed stacked scale.
- Mitigation: treat runtime final scale as authoritative and avoid host recomputation of stack scaling math.

20. Risk: HeightInRoom and authored z-order become conflated, causing layering regressions.
- Mitigation: lock D-24 composite layering rule and add dedicated tests/diagnostics that show both authored z-order and HeightInRoom contributions.

## 10. Remaining Open Details (Implementation-Level)

1. None for D-22: allowed override bounds are locked and centrally enforced via shared policy constants.

## 11. Immediate Next Step

Execute Phase 5 closure sweep:

1. Run focused regression/replay gates and update producer-facing behavior notes.
2. Validate one representative sample scenario and close plan.

## 12. Follow-Up Items (Post-v1)

1. Evaluate expanding active-item fallback from in-room movement actions to additional object-targeted action families, with explicit opt-in and regression coverage.
2. Reconsider action-scoped transition-hint chooser after v1 closeout.

## 13. Implementation Execution Plan (High-Level)

This phase plan is execution-oriented and includes explicit stop points for manual verification before continuing.

### Phase A - Shared Contracts and Model Scaffolding

Scope:

1. Add shared/runtime contract types and model fields for movement, rotation, collision/stack/height, active-item, and visual hint defaults.
2. Add designer model fields and serialization wiring using locked defaults.
3. Add compatibility readers that apply default values for older data.

Stop and manual check milestone A:

1. Open a legacy sample project and verify it loads with defaults and no fatal validation issues.
2. Create a new object and confirm new fields are present with expected defaults.
3. Run build and baseline test gates.

### Phase B - Core Runtime Grid Movement and Rotation Engine

Scope:

1. Implement stepwise movement evaluation with AllowPartialMove behavior.
2. Implement collision/stack/height rules and HeightInRoom recomputation.
3. Implement rotation turn-mode and face-mode rules (8-way facing, cardinal collision orientation).
4. Implement detailed result codes and generic echo fallback behavior.

Stop and manual check milestone B:

1. Run scripted test scenarios for full move, partial move, blocked move, and out-of-bounds.
2. Run rotate scenarios for cardinal and diagonal-facing outcomes.
3. Verify no-op failures do not emit extra render updates.

### Phase C - Active Item API and Resolution Integration

Scope:

1. Add shared host-interface methods: SetActiveRoomObjectAtPoint, SetActiveRoomObjectById, ClearActiveRoomObject.
2. Implement runtime point resolution from pixel coordinates and topmost-selection rule.
3. Integrate movement command resolution precedence (explicit reference, then active item).

Stop and manual check milestone C:

1. In simulator, select via text command and point gesture; verify both succeed.
2. Verify invalid point/id leaves existing active item unchanged with proper result code.
3. Verify room change clears active item.

### Phase C.5 - Grouped JSON Reorganization and Migration Gate

Scope:

1. Refactor movement/appearance object persistence into grouped subsections across project, clean export, and runtime bootstrap data.
2. Apply omit-default/null serialization behavior where safe and deterministic.
3. Migrate sample project files in place and refresh affected fixtures/snapshots.
4. Keep this slice forward-only: no legacy flat-shape compatibility loader path.

Stop and manual check milestone C.5:

1. Open migrated samples and verify load/save stability with grouped JSON.
2. Verify clean/runtime JSON contracts use grouped sections and expected omission rules.
3. Verify runtime movement phases remain blocked until grouped-shape tests are green.

### Phase D - Designer UX for Grid, Object Shape/Orientation, and Authoring Controls

Scope:

1. Add project and room grid editors (valid CellSize dropdown + row/column echo).
2. Add object authoring controls for movable, stack order, footprint, orientation, height, and movement defaults.
3. Add rotation authoring controls and strict 45-multiple validation for designer inputs.

Stop and manual check milestone D:

1. Author several objects with different footprints/orientations and verify persisted values.
2. Verify invalid designer inputs are blocked and validation messages are clear.
3. Confirm command vocabulary remains producer-defined while parser extracts numeric values/directions.

### Phase E - Designer Room Canvas Snap/Overlay and Stack Visual Cue

Scope:

1. Implement snap-to-grid default-on behavior and overlay default-off toolbar toggle.
2. Implement runtime-driven stack scale rendering cue with locked defaults.
3. Keep preference persistence app/session-scoped and out of clean export.

Stop and manual check milestone E:

1. Manually place and move stacked objects; verify visible scale-depth cue.
2. Verify overlay and snap toggles persist in app/session scope only.
3. Perform visual calibration pass for D-22 allowed ranges and lock final bounds.

### Phase F - Cross-Host Parity, Hardening, and Regression Sweep

Scope:

1. Validate shared behavior parity across Designer and Simulator for identical runtime states.
2. Finalize diagnostics surfaces and regression tests.
3. Close remaining implementation-level open detail(s) and update plan status.

Stop and manual check milestone F:

1. Execute focused regression suite and replay gate.
2. Run representative puzzle-room smoke tests (movement, rotate, stack, active-item workflows).
3. Sign off on D-22 bounds and mark plan implementation-ready for coding slices.

### Phase G - End-of-Plan JSON Shape Review (Deferred Reorg Candidates)

Scope:

1. Review additional flat object JSON clusters for logical subsection grouping, following the same pattern used for appearance/movability.
2. Identify candidate areas for grouped persistence/contract shape (for example: lock requirements, composite recipe/parts, command/action metadata, and variable-related payload clusters).
3. Document per-candidate default/null omission opportunities and contract-risk notes.
4. Propose sequencing for follow-up JSON reorg slices after movement v1 completion.

Stop and manual check milestone G:

1. Produce a reviewed candidate list with priority and migration impact notes.
2. Confirm which grouped-shape follow-ups become post-v1 implementation work.
3. Confirm no unplanned contract changes are folded into movement v1 late in the cycle.

### Suggested Delivery Rhythm

1. Do not implement past a milestone without manual signoff.
2. After each milestone, run:
- `dotnet build .\StoryboardDesigner.slnx`
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`
3. For runtime-heavy slices, also run the focused runtime filter gate already defined in this plan.

### Alternate Ordering: Feedback-First (Recommended For This Initiative)

This alternate order is valid and often preferred for producer-facing features where data shape and authoring UX need early real-world feedback.

Recommended sequence:

1. Shared contract scaffolding first (slice A1).
2. Designer UX slices next (D1, D2, D3) with placeholder/non-executing action wiring where needed.
3. Serialization/default wiring next (A2) so authored data can be saved/loaded and reviewed.
4. Runtime core slices after UX/data shape feedback (B1, B2, B3, C1, C2).
5. Canvas/visual layering slices (E1, E2), then hardening/calibration (F1, F2).

Why this ordering can be better here:

1. Producer look-and-feel and object authoring semantics are validated early.
2. Data model issues are discovered before deep runtime implementation.
3. JSON/default compatibility concerns are surfaced before action execution complexity.

Guardrails required when using this order:

1. Keep runtime behavior behind explicit feature flags or safe stubs until B-slices are complete.
2. Mark UX elements that depend on not-yet-implemented runtime behavior as preview/non-executing where needed.
3. Do not finalize contract field names after D-slices without rerunning serialization round-trip checks.
4. Run build/tests at each stop point even before runtime slices begin to prevent drift.

## 14. Documented Implementation Slices (Historical Slice Backlog)

Execution status:

1. This section is retained as historical planning detail.
2. It is no longer the source of truth for current execution status.
3. Use Status, Reconciliation Snapshot, and Phase status blocks above for current state.

### Slice A1 - Shared Contract Skeletons

Objective:

1. Introduce shared contract types and enums for movement, rotation, active-item APIs, result codes, and visual hints.

Dependencies:

1. Decision lock complete.

Deliverables:

1. Shared contract placeholders with locked names and payload semantics.
2. Result-code enum definitions aligned to D-18.

Completion evidence:

1. Build passes.
2. Contract-focused unit tests compile and run.

### Slice A2 - Model Defaults and Serialization Wiring

Objective:

1. Add model fields and persistence mappings for movable, footprint, orientation, stack order, object height, and visual settings.

Dependencies:

1. Slice A1 complete.

Deliverables:

1. Default-value application for missing fields.
2. Save/load mappings for new fields.

Completion evidence:

1. Legacy sample opens with defaults applied.
2. Round-trip serialization tests pass.

### Slice B1 - Movement Core (Stepwise + Partial Move)

Objective:

1. Implement movement execution with stepwise path checks and per-action AllowPartialMove behavior.

Dependencies:

1. Slice A2 complete.

Deliverables:

1. Full-distance, partial-distance, blocked, and out-of-bounds pathways.
2. Result-code emission and generic echo fallback.

Completion evidence:

1. Runtime movement tests pass for all four pathways.
2. No-op failures produce no extra render update.

### Slice B2 - Collision, Stacking, and HeightInRoom

Objective:

1. Implement StackOrder collision policy and cumulative HeightInRoom recomputation.

Dependencies:

1. Slice B1 complete.

Deliverables:

1. StackOrder rule handling for destination and mid-path checks.
2. HeightInRoom update behavior for stack changes.

Completion evidence:

1. Stack/collision tests pass.
2. HeightInRoom assertions pass for multi-level stacks.

### Slice B3 - Rotation Core (Turn + Face Modes)

Objective:

1. Implement rotate behavior supporting both turn-mode and face-mode under locked rules.

Dependencies:

1. Slice B2 complete.

Deliverables:

1. 45-degree multiple handling.
2. Clockwise default tie behavior.
3. 8-way facing with cardinal collision orientation.

Completion evidence:

1. Rotation tests pass for cardinal/diagonal endpoints and collision-orientation outcomes.
2. Numeric command rounding warning path validated.

### Slice C1 - Active Item API and Resolution

Objective:

1. Add and wire active-item selection API paths with current-room scope behavior.

Dependencies:

1. Slice A1 complete.
2. Slice B1 complete.

Deliverables:

1. SetActiveRoomObjectAtPoint, SetActiveRoomObjectById, ClearActiveRoomObject.
2. Point-based topmost selection from pixel coordinates.

Completion evidence:

1. Selection success/failure behaviors match locked rules.
2. Invalid selection leaves active item unchanged.

### Slice C2 - Command Resolution Integration

Objective:

1. Integrate active-item fallback precedence into movement command target resolution.

Dependencies:

1. Slice C1 complete.

Deliverables:

1. Explicit reference first, active-item fallback second, clarification/failure third.
2. No active-item tie-break for explicit ambiguity.

Completion evidence:

1. Command precedence tests pass.
2. Ambiguity tests pass.

### Slice D1 - Grid Authoring UX

Objective:

1. Add project/room grid editors with constrained CellSize dropdown and row/column echo.

Dependencies:

1. Slice A2 complete.

Deliverables:

1. Dropdown options constrained to valid divisors.
2. Real-time echo of effective grid dimensions.

Completion evidence:

1. UX tests pass for valid and invalid grid configurations.
2. Manual milestone confirms expected options and echo text.

### Slice D2 - Object Authoring UX (Physical + Movement Fields)

Objective:

1. Add object settings UX for movable/isMovable default, stack order, footprint, orientation, and object height.

Dependencies:

1. Slice D1 complete.

Deliverables:

1. Editor controls with bounds enforcement.
2. Validation messages for invalid values.

Completion evidence:

1. Authoring tests pass.
2. Manual authoring round-trip confirms persisted values.

### Slice D3 - Rotation Authoring UX

Objective:

1. Add rotate action authoring controls for turn/face mode and strict 45-multiple validation.

Dependencies:

1. Slice B3 complete.
2. Slice D2 complete.

Deliverables:

1. Designer-side strict validation for guided inputs.
2. Preview/status messaging for expected facing outcomes.

Completion evidence:

1. Rotate authoring tests pass.
2. Manual checks confirm invalid values are blocked.

### Slice E1 - Room Canvas Snap and Overlay

Objective:

1. Implement snap-default-on and overlay-default-off with toolbar toggle.

Dependencies:

1. Slice D1 complete.

Deliverables:

1. Snap behavior for placement interactions.
2. Overlay toggle and app/session preference persistence.

Completion evidence:

1. Manual canvas checks pass.
2. Preference persistence tests pass.

### Slice E2 - Stack Visual Scaling and Layering Integration

Objective:

1. Implement stack scale cue formula and HeightInRoom plus authored z-order composite layering behavior.

Dependencies:

1. Slice B2 complete.
2. Slice E1 complete.

Deliverables:

1. Runtime final scale calculation from depth.
2. Composite layering behavior preserving authored z-order as base.

Completion evidence:

1. Scale and layering tests pass.
2. Manual stacked-scene visual validation passes.

### Slice F1 - Cross-Host Parity and Hardening

Objective:

1. Validate parity, regression stability, and operational readiness across hosts.

Dependencies:

1. Slices A1 through E2 complete.

Deliverables:

1. Cross-host parity assertions for movement, rotation, stacking, active-item behavior.
2. Runtime-focused regression gate pass.

Completion evidence:

1. Full test and replay gates pass.
2. Phase signoff notes recorded in this plan.

### Slice F2 - D-22 Calibration Lock

Objective:

1. Run visual calibration and lock allowed override ranges for StackScaleStep and MinStackScale.

Dependencies:

1. Slice E2 complete.

Deliverables:

1. Final locked range values added to D-22.
2. Calibration evidence notes with representative stack scenes.

Completion evidence:

1. D-22 marked fully locked.
2. Remaining-open-details section reduced to zero items.

## End-of-Plan Addendum - Transition Hint Chooser Reconsideration

Before marking this plan complete, perform one explicit reconsideration pass on action-scoped transition hint chooser scripts while movement context is still fresh.

Reconsideration checklist:

1. Decide whether adding chooser-script support now is low-risk and materially beneficial for v1 producer workflows.
2. If yes, implement only if it can be done additively without reopening core movement/rotation contract shapes.
3. If no, capture a concrete post-v1 follow-up item with scope boundaries, dependencies, and target phase.
4. In either case, keep result-side transition hint authoritative for host rendering.
