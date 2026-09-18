# Top-Left Anchor Placement Simplification Plan

## Purpose
Create a clear, deterministic placement model with no implicit centering magic.

Core rule:
- Canonical authored anchor is footprint top-left.
- Placement is driven only by explicit controls (scale, rotation, local offset X/Y).
- Runtime normalizes to final draw values.
- Host paints normalized values only.

## Goals
1. Remove confusion from implicit footprint centering.
2. Keep designer preview and simulator/runtime behavior aligned.
3. Migrate safely with phase gates and rollback points.
4. Let producer manually retune local offsets at the correct step.

## Non-Goals
1. No host-side fudge factors.
2. No hidden per-direction adjustments.
3. No mixing large refactors with behavior changes in one phase.

## Baseline Expectations
Before each behavior phase:
1. Build passes.
2. Playback smoke gate passes.
3. Existing diagnostics remain available.

## Validation Gates
Run after each phase:
1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"

Use this focused runtime gate during normalization changes:
1. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"

## Phase 0 - Freeze Baseline (No Behavior Changes)
1. Capture current tuple logs for N/E/S/W doors in the same scene.
2. Record current visual observations for each door direction.
3. Confirm baseline build and smoke gates pass.

Exit criteria:
- Reference tuple packet and visual notes are captured.
- Status: Complete (2026-07-23) via TOP_LEFT_ANCHOR_PHASE0_BASELINE_PACKET.md

## Phase 1 - Introduce Explicit Placement Mode (Default Legacy)
1. Add placement mode plumbing for two behaviors:
- LegacyCentered
- ExplicitTopLeft
2. Wire through designer preview and runtime normalization paths.
3. Keep default as LegacyCentered to avoid immediate behavior change.

Exit criteria:
- No visual delta at default mode.
- All gates pass.

## Phase 2 - Implement ExplicitTopLeft Math Behind Mode
1. Implement equation behind ExplicitTopLeft:
- draw = anchor(top-left) + rotated(localOffset)
2. Keep transform ordering explicit and deterministic.
3. Expand diagnostics to print placement mode and all placement terms.

Exit criteria:
- Mode toggle changes tuple values in the expected deterministic way.
- All gates pass.
- Execution note (2026-07-23): Designer preview implicit centering removed (top-left anchored icon layer offsets). Build and playback smoke gates passed.

## Phase 3 - Manual Retuning Window (Producer Step)
1. Enable ExplicitTopLeft in designer preview.
2. Manually retune local offsets for sample objects/doors.
3. Save authored offsets to project content.

Exit criteria:
- Designer preview placement is correct for target sample scenes.
- Updated offsets are committed as authored content.
- Current checkpoint: Complete for current calibration scope (producer retuning/sign-off performed).

## Phase 4 - Runtime/Simulator Cutover for Target Flow
1. Switch simulator/runtime target flow to ExplicitTopLeft.
2. Verify tuple parity:
- runtime terms
- payload terms
- host draw terms
3. Validate visual parity between designer preview and simulator.

Exit criteria:
- N/E/S/W doors match expected placement.
- No directional drift beyond acceptable tolerance.

## Phase 4.5 - Post-Cutover Host Parity Alignment and Convergence
Purpose:
- Lock Designer and Simulator rendering pipelines as close as reasonable.
- Allow differences only when there is a clear feature/function reason.

Scope:
1. Simulator-first convergence work (five slices).
2. Adopt shared placement math in Room Designer preview path.
3. Preserve runtime-host boundaries (no Designer dependency from Simulator/Shared).

Slice plan (detailed execution packet):

### Slice 1 - Room Image Transform Parity in Simulator
Objective:
1. Match Designer transform composition semantics for directional room-image layers.

Implementation targets:
1. Align transform order between scale, rotation, and translation for room-image draw path.
2. Align transform origin/pivot assumptions used for directional room images.
3. Remove any simulator-only corrective transform layers not present in Designer semantics.

Evidence required:
1. Before/after RENDER TUPLE deltas for N/E/S/W calibration scene.
2. PREVIEW TRACE output showing applied transform chain and origin values.
3. PREVIEW VISUAL bounds capture proving no directional drift.

Done criteria:
1. Directional room-image tuple deltas are explainable and deterministic.
2. No unexplained N/E/S/W transform drift remains.

### Slice 2a - Room Image Stretch Parity in Simulator
Objective:
1. Match Designer stretch behavior for room directional image layers.

Implementation targets:
1. Align effective Stretch mode and sizing inputs used by simulator room-image containers.
2. Ensure width/height sizing source matches Designer semantics for calibration fixtures.
3. Remove ad-hoc stretch overrides that diverge from Designer behavior.

Evidence required:
1. PREVIEW TRACE capture including resolved stretch mode and final measured size.
2. PREVIEW VISUAL comparisons at minimum and maximum expected room-image sizes.

Done criteria:
1. Stretch behavior is parity-accepted for calibration fixtures.
2. No host-specific stretch exception remains without documented reason.

### Slice 2b - Room Image Clipping Parity in Simulator
Objective:
1. Match Designer clipping and bounds behavior for room directional layers and overlays.

Implementation targets:
1. Align clipping region origin and dimensions with Designer room canvas semantics.
2. Align clipping timing/order relative to transform application.
3. Verify directional overlays inherit the same clipping contract.

Evidence required:
1. PREVIEW VISUAL bounds block including clip rect diagnostics.
2. PREVIEW TRACE entries showing clip application stage in pipeline.

Done criteria:
1. Clipped content bounds match Designer expectations for all door directions.
2. No directional-specific clipping artifacts remain.

### Formal Approval Gate - Required Before Slice 3
Entry condition:
1. Slices 1, 2a, and 2b are complete and evidence packets are reviewed.
2. Build, playback smoke, and focused runtime gates are green for the current checkpoint.

Approval rule:
1. Do not start Slice 3 until producer gives explicit formal approval.
2. Approval must explicitly confirm transition from simulator-only parity work into Room Designer shared-math adoption.

If approval is not granted:
1. Continue only simulator-side parity investigation/hardening.
2. Do not modify Room Designer placement math paths.

### Slice 3 - Shared-Math Adoption in Room Designer
Objective:
1. Move Room Designer preview placement math onto shared placement helpers to eliminate drift.

Implementation targets:
1. Replace duplicated preview placement math with calls into shared placement computation.
2. Keep designer UX behavior unchanged while swapping the calculation source.
3. Preserve boundaries: shared owns math; host owns visualization and interaction only.

Evidence required:
1. Tuple parity checks before/after adoption for doors and one large scaled object fixture.
2. Targeted tests proving shared placement outputs are consumed by Room Designer preview.

Done criteria:
1. Room Designer preview placement path uses shared math helpers.
2. No regression in current calibration fixtures.

### Slice 4 - Proof and Guardrails
Objective:
1. Lock parity with durable diagnostics and regression coverage.

Implementation targets:
1. Extend parity diagnostics packet coverage for doors and large scaled asset case.
2. Add or update focused regression tests for transform/stretches/clipping parity-sensitive cases.
3. Keep A/B comparative evidence active while both modes exist.

Evidence required:
1. Full evidence packet per fixture:
- RENDER TUPLE
- PREVIEW TRACE
- PREVIEW VISUAL bounds
2. Focused test results attached to this slice sign-off.

Done criteria:
1. Guardrails fail on reintroduced parity drift.
2. Parity packet is reproducible and reviewed for calibration fixtures.

### Slice 5 - A/B Mode Consolidation Decision
Objective:
1. Decide and execute the final A/B lifecycle outcome.

Implementation targets:
1. Define explicit soak threshold for parity stability (for example: N successful parity runs with no unexplained drift).
2. If threshold met, demote/remove B from primary workflow and retain only required debug toggles.
3. If threshold not met, keep B scoped to diagnostics-only while tracking unresolved deltas.

Evidence required:
1. Soak run summary with parity result counts and fixture list.
2. Decision note documenting keep/demote/remove outcome and rationale.

Done criteria:
1. A/B policy is explicit and documented.
2. Primary workflow no longer depends on temporary comparison mode.

Validation additions for this phase:
1. Keep existing build and playback gates.
2. Keep focused runtime gate.
3. Add simulator parity evidence packet per slice:
- RENDER TUPLE block
- PREVIEW TRACE block
- PREVIEW VISUAL bounds block
4. Apply slice-by-slice stop points:
- Do not start next slice until current slice evidence and done criteria are signed off.
- If a slice regresses parity, rollback only that slice change set.

Exit criteria:
1. Simulator and Designer parity accepted on calibration fixtures (doors + scaled object case).
2. Room Designer preview is using shared placement math path.
3. A/B mode either stabilized as debug-only or approved for removal in follow-on cleanup.
4. This phase is complete before final regression lock-off and legacy cleanup.

### Phase 4.5 Evidence Snapshot (2026-07-24)
Evidence source:
1. Unified bottom Output console with High diagnostics enabled.
2. Room-wide diagnostic emission enabled (all renderable objects at High level).

Sample and mode coverage:
1. Room: Atrium
- Objects captured: Door_N, Door_E, Door_S, Door_W
- Modes captured: CompareUseLegacy, CompareUseShared, LegacyOnly, SharedOnly
2. Room: Supply Closet
- Objects captured: Door_W, Crowbar, Backpack, BrassKey
- Modes captured: CompareUseLegacy, CompareUseShared, LegacyOnly, SharedOnly

Observed parity results:
1. Every captured object in both rooms reported delta=(0,0).
2. For every captured object, selected, legacy, and shared offsets matched in all four modes.
3. Rotated room case was included (Supply Closet roomRot=270) with no drift.
4. Non-integer offset case was included (Crowbar selected=(2.721,-7.832)) with no drift.

Representative lines (excerpt):
1. object='Door_E'; mode=SharedOnly;selected=(40,6);legacy=(40,6);shared=(40,6);delta=(0,0)
2. object='Door_W'; mode=CompareUseLegacy;selected=(-0,234);legacy=(-0,234);shared=(-0,234);delta=(0,0)
3. object='Crowbar'; mode=CompareUseShared;selected=(2.721,-7.832);legacy=(2.721,-7.832);shared=(2.721,-7.832);delta=(0,0)

Checkpoint conclusion:
1. Shared placement math is parity-equivalent to legacy for the covered fixtures and mode combinations.
2. Diagnostics/output plumbing is sufficient for ongoing A/B evidence capture and review.
3. Producer approval received (2026-07-24).

Status:
1. Complete (approved 2026-07-24).

## Phase 5 - Regression Protection
1. Add or update focused tests for explicit top-left normalization and transform order.
2. Keep playback regression gate mandatory.

Execution checklist:
1. Add locked vector tests for approved door parity samples. Status: Complete (2026-07-24).
2. Keep shared math rotation/local-compensation tests green in Storyboard.Shared.Tests. Status: Complete (2026-07-24).
3. Keep designer parity diagnostics tests green (snapshot contains mode/delta/object coverage). Status: Complete (2026-07-24).
4. Run mandatory gates:
- dotnet build .\StoryboardDesigner.slnx
- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"
5. Document Phase 5 completion with date + gate outputs before Phase 6 edits. Status: Complete (2026-07-24).

Phase 5 completion evidence (2026-07-24):
1. RuntimeRenderablePlacementMathTests: passed (6/6).
2. Designer parity diagnostics focused tests: passed (18/18).
3. Playback regression gate: passed (6/6).
4. Focused runtime gate: passed (60/60).
5. Solution build: passed.

Exit criteria:
- Tests prevent reintroduction of implicit centering behavior.

Status:
1. Complete (2026-07-24).

## Phase 6 - Cleanup Legacy Mode
1. After confirmed stability, remove legacy-centered path.
2. Keep useful diagnostics for future troubleshooting.

Current execution status (2026-07-24):
1. SharedOnly is now the default Room Designer preview mode.
2. Primary Room Designer mode picker is narrowed to SharedOnly for normal workflow.
3. Legacy/compare placement selection branches were removed from Room Designer preview object placement logic.
4. Diagnostics and unified console capture remain enabled.

Validation evidence after SharedOnly-first switch:
1. Solution build passed.
2. Shared placement parity vectors passed in Storyboard.Shared.Tests.
3. Updated Designer mode/diagnostics tests passed.
4. Playback regression gate passed (6/6).
5. Focused runtime gate passed (60/60).

Remaining work to close Phase 6:
1. None.

Exit criteria:
- One placement model remains.
- All gates pass.

Status:
1. Complete (2026-07-24).

Phase 6 completion evidence (2026-07-24):
1. Solution build passed.
2. SharedOnly cleanup targeted Designer tests passed (18/18).
3. Playback regression gate passed (6/6).
4. Focused runtime gate passed (60/60).

## Change Control and Safety Rules
1. One phase per change set.
2. Stop for visual sign-off after each phase.
3. If a phase regresses placement, rollback only that phase.
4. Do not combine behavior and unrelated refactor work.

## Operational Notes
1. If build hits simulator exe lock warnings/errors, close running simulator and rerun build.
2. Prefer no-apphost compile for quick simulator compile checks while app is open:
- dotnet build .\Storyboard.Simulator\Storyboard.Simulator.csproj -p:UseAppHost=false

## Ownership Split
1. Assistant:
- implement phase changes
- run gates
- provide tuple deltas and risk notes
2. Producer:
- manual retuning during Phase 3
- visual sign-off between phases

## Design Decision Checklist
Status key:
- Undecided
- Agreed

1. Canonical anchor definition
- Question: Is canonical object position always footprint top-left for all objects, all rotations, and all hosts?
- Status: Agreed

2. Single placement equation
- Question: Do we formally adopt draw = anchor + rotated(localOffset) and explicitly forbid implicit footprint-centering terms?
- Status: Agreed (with caveat: exact transform/order semantics must be explicitly defined)

3. Transform order contract
- Question: What exact order is authoritative across designer preview and runtime normalization?
- Status: Agreed
- Agreed contract terms and order:
	1. Anchor is footprint top-left.
	2. effectiveNormalizedRotation = Normalize(inRoomRotation + localRotation).
	3. localNudgeVector is converted to roomSpaceNudgeVector by rotating with effectiveNormalizedRotation.
	4. finalDrawPosition = anchor + roomSpaceNudgeVector.
	5. Host applies effectiveNormalizedRotation and scale for visual rendering only, without recomputing placement semantics.

4. Coordinate space and units
- Question: Are anchor and local offsets always render-space pixels in authored data, runtime variables, and payloads?
- Status: Agreed
- Clarification:
	1. Anchor is in render-space pixels.
	2. localNudgeVector is authored in icon-local axes.
	3. roomSpaceNudgeVector is the rotation-converted delta in render-space pixels.
	4. finalDrawPosition = anchor + roomSpaceNudgeVector.

5. Rotation pivot semantics
- Question: Is rotation pivot always icon center, and is that a fixed invariant rather than inferred behavior?
- Status: Agreed

6. Scope of simplification
- Question: Does this simplification apply only to room objects, or also to room directional images and future renderable types?
- Status: Agreed

7. Compatibility strategy
- Question: Do we keep a temporary compatibility switch (LegacyCentered vs ExplicitTopLeft) during migration, or cut over directly?
- Status: Agreed (direct cutover, no long-lived compatibility mode)
- Agreed migration shape:
	1. Implement ExplicitTopLeft contract directly.
	2. Add explicit manual retuning checkpoint before runtime/simulator sign-off.
	3. Use phase gates and phase-scoped rollback for safety instead of dual runtime modes.

8. Retuning policy
- Question: Do we intentionally require manual local-offset retuning for existing content that relied on implicit centering?
- Status: Agreed

9. Acceptance criteria
- Question: What visual tolerance and sample-scene sign-off gate defines success?
- Status: Agreed

10. Diagnostics contract
- Question: Which tuple fields are mandatory long-term for fast verification and troubleshooting?
- Status: Agreed

11. Runtime-host ownership boundary
- Question: Do we reaffirm that runtime performs all normalization and host remains a dumb painter with no cell or footprint awareness?
- Status: Agreed

12. Rollback boundaries
- Question: If a phase regresses behavior, is rollback always constrained to the current phase change set only?
- Status: Agreed

## Placement Logic Lock-Off Questions (Current)
Purpose:
- Lock placement correctness first.
- Do not proceed to commonization until these are explicitly agreed.

Status key:
- Undecided
- Agreed

1. Source-of-truth policy
- Question: Is the object appearance editor authored local layout (scale, local rotation, local offset X/Y) the authoritative source that must be preserved by in-room preview/runtime placement?
- Status: Agreed
- Clarification:
	1. The footprint-to-icon relationship is fully defined by the appearance editor.
	2. Outside local appearance, only final in-room placement and in-room rotation may be applied.
	3. Aside from those in-room transforms, preview/runtime must preserve the authored local relationship exactly.

2. Pivot policy re-open
- Question: Is icon-center pivot still required, or should pivot be changed if needed to preserve authored placement predictably during in-room rotation?
- Status: Agreed (implementation-defined)
- Clarification:
	1. Pivot/rotation implementation is a technical detail, not a hard requirement.
	2. Choose whatever pivot/rotation method best preserves the authored footprint-to-icon relationship from Question 1.

3. No hidden correction policy
- Question: Do we forbid auto-centering, bounding-box compensation, and any other implicit correction layers that are not explicitly authored?
- Status: Agreed
- Clarification:
	1. No implicit correction layers should be used.
	2. Authored appearance data already provides sufficient detail for final image placement behavior.
	3. Keep the pipeline simple, explicit, and manual-by-design.

4. Pipeline simplification
- Question: Do we collapse to a single transform pipeline for room preview placement and remove mixed/legacy alternatives introduced during iteration?
- Status: Agreed (conditional)
- Clarification:
	1. Simplification is approved if and only if it best achieves Question 1 source-of-truth behavior.

5. Correctness-before-commonization
- Question: Do we agree to finish and lock room-preview placement correctness first, then commonize identical math into shared runtime only after sign-off?
- Status: Agreed

6. Door-first calibration set
- Question: Do we use only the four Atrium doors as the calibration set for finalizing placement logic before expanding to other objects?
- Status: Agreed (with required secondary verification)
- Clarification:
	1. Start calibration with the four Atrium doors.
	2. Before lock-off completion, verify that supply-room objects remain correct.

7. Validation contract for lock-off
- Question: Is lock-off achieved only when authored door alignments from appearance editor are preserved in room preview across N/E/S/W without re-authoring local data?
- Status: Agreed
