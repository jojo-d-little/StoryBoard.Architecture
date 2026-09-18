# Multi-Story Vertical Traversal Plan

Date: 2026-08-24
Status: Completed (Archived)

## Completion Summary

Implemented and validated:

1. Runtime vertical hardening uses existing navigation action pipelines for `UP` and `DOWN`.
2. Floor-aware area map authoring is implemented with `floorElevation` contract/model support and floor-level editing.
3. Vertical room/traversal indicators and seeded vertical traversal authoring workflow are implemented.
4. Focused runtime tests include vertical success paths plus negative-path hardening:
- missing vertical leg
- blocked vertical leg
- door-closed vertical traversal
5. Save/load and bootstrap seams were updated and covered for vertical traversal direction handling.
6. Validation now includes per-room/per-direction effective target uniqueness guardrail (`TRV-007`).
7. Replay-style vertical regression coverage was added in designer tests via in-memory snapshot progression.

Validation evidence (latest run set):

1. `dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~Process_NavigateDirection_WithUpDirection_WithoutVerticalLeg_DoesNotMove|FullyQualifiedName~Process_NavigateDirection_WithUpDirection_WhenVerticalLegBlocked_DoesNotMove|FullyQualifiedName~Process_NavigateToAdjacent_WithUpDirection_WhenDoorClosed_DoesNotMove"`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~TraversalDirectionalTargetUniquenessRuleTests|FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_VerticalTraversal_InMemoryScenario_OutputLinesMatchAtEachStep"`
3. `dotnet build .\StoryboardDesigner.slnx`

## Purpose

Enable multi-story area authoring and runtime vertical movement (`UP`/`DOWN`) while minimizing risk and complexity through additive changes, strict boundary isolation, and phased rollout.

## Core Direction (Locked Intent Draft)

1. Host room-change contract remains unchanged.
2. Runtime must use the existing navigation action pipeline for all directions.
3. `go up` and `go down` are processed through the same command -> direction resolution -> traversal -> move execution path as `go east`.
4. No new vertical-only movement action family is introduced.
5. Existing 8-direction traversal wizard remains unchanged.
6. Vertical traversal authoring is implemented through a new lightweight designer workflow.

## Why This Is Next

1. Current traversal model is strong on a single plane (N/NE/E/SE/S/SW/W/NW).
2. Canonical direction model already includes `UP` and `DOWN`, but area authoring/UX and regression depth are not equivalent.
3. Multi-story maps are a natural extension of area design and world progression.

## Scope

In scope:

1. Multi-story area map authoring (one story visible/edited at a time).
2. Visual room indicators for above/below relationships and vertical traversal presence.
3. New lightweight Up/Down traversal wizard.
4. Runtime traversal resolution/execution hardening for `UP`/`DOWN` through existing movement actions.
5. Validation and tests for vertical traversal integrity.

Out of scope:

1. Host payload redesign.
2. Replacing or broad refactor of existing 8-direction traversal wizard.
3. Full pathfinding or 3D rendering changes.
4. Broad movement physics redesign.

## Constraints and Guardrails

1. Keep `Storyboard.Simulator` independent of `StoryboardDesigner.App`.
2. Keep reusable runtime traversal semantics in shared/runtime layers.
3. Keep producer command grammar producer-owned (no hidden aliases/fallback words).
4. Keep changes additive and reversible by slice.
5. Preserve deterministic replay and diagnostics behavior.
6. Contract/schema updates are required first; no feature implementation begins before contract lock decisions are approved.

## Contract-First Gate (Required Before Implementation)

This project requires schema/contracts to be adjusted and agreed before implementation changes.

1. Primary schema touchpoint:
- `Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Core/RoomPlacement.core.schema.json`
- Add `floorElevation` as an integer field for map placement semantics.

2. Contract propagation expectations:
- Designer contract consumes core room placement shape via:
  - `Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Designer/RoomPlacementDto_contract.schema.json`
- Runtime room placement contract consumes core shape via:
  - `Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Runtime/RuntimeRoomPlacementDto_contract.schema.json`
- Area dto schemas already reference room placements and will inherit the new field through core references:
  - `Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Designer/AreaDto_contract.schema.json`
  - `Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Core/Area.core.schema.json`

3. Direction contract verification:
- Verify direction enum contract already includes `UP` and `DOWN`; if already present, no direction contract change is needed.
- If any contract gap is found, resolve in schema first before runtime/designer behavior changes.

4. Gate exit before implementation:
- Schema field names/types/default semantics for `floorElevation` are approved.
- Contract generation/acceptance plan is approved.
- Any lock-manifest updates are planned as a separate explicit lock workflow step.

## Target Architecture

1. Runtime direction handling:
- Extend/verify normalization and traversal lookup for canonical `UP` and `DOWN`.
- Keep one movement semantic path for all directions.

2. Area authoring model:
- Introduce floor-level placement metadata for map layout.
- Preserve backward compatibility by defaulting existing content to floorElevation `0`.

3. Area map designer:
- Add floor selector for viewing/editing one floor at a time.
- Render room-level visual markers:
  - room exists above
  - room exists below
  - up traversal exists
  - down traversal exists

4. Vertical traversal authoring:
- Add a new lightweight Up/Down traversal workflow launched from room indicator/context action.
- Reuse existing traversal concepts:
  - passability (`isPassable` and linked shared variable patterns)
  - optional door/lock pairing model
  - deterministic validation rules

## Phased Plan

### Phase 0 - Baseline and Lock

1. Confirm current runtime behavior and tests for `UP`/`DOWN` command parsing and traversal resolution.
2. Capture baseline regression evidence for existing horizontal traversal behavior.
3. Finalize design lock decisions listed below.

### Phase 1 - Runtime Vertical Hardening (No New Action Types)

1. Verify/complete canonical direction mapping and inverse logic for `UP <-> DOWN`.
2. Ensure existing move action executor and traversal resolver handle vertical legs identically to horizontal legs.
3. Reuse existing result-code/diagnostic channels; add only additive diagnostics if needed.
4. Add focused runtime tests for:
- passable vertical leg success
- blocked vertical leg
- door/lock-gated vertical leg
- missing vertical leg diagnostics

### Phase 2 - Floor-Aware Area Map Authoring

1. Add `floorElevation` metadata for area map room placement state.
2. Add floor selector in area map designer.
3. Keep map editing interactions unchanged within a single selected floor.
4. Add migration/back-compat handling for legacy projects (implicit `floorElevation = 0`).

### Phase 3 - Vertical Indicators and Wizard

1. Add per-room visual indicators for above/below room presence and up/down traversal presence.
2. Add lightweight Up/Down traversal wizard/workflow:
- create/edit/remove up/down traversal
- optional paired reciprocal leg creation/edit
- optional door/lock linkage controls
3. Keep existing 8-direction wizard untouched.

### Phase 4 - Validation, Replay, and Guardrails

1. Add designer validation for vertical traversal consistency.
2. Add replay and command regression coverage for `go up`/`go down`.
3. Rerun focused runtime and playback smoke gates.
4. Confirm no host contract changes required.

## Validation and Regression Plan

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`
4. Add new targeted test group for vertical traversal and story-map authoring behavior.

## Risks

1. Direction parsing regressions if `UP`/`DOWN` alias normalization is inconsistent.
2. Traversal pairing bugs between single-direction and reciprocal vertical links.
3. Designer complexity creep if vertical and horizontal workflows are merged.
4. Replay drift if diagnostics or traversal ordering become nondeterministic.

## Risk Mitigations

1. Keep runtime movement pipeline unified across all directions.
2. Keep 8-direction wizard unchanged; build separate lightweight vertical workflow.
3. Stage rollout with strict slice boundaries and focused tests per phase.
4. Add deterministic diagnostics assertions and replay checks before broader expansion.

## Exit Criteria

1. `go up`/`go down` execute through the same existing movement action pipeline as horizontal movement.
2. Area map supports per-floor editing without breaking existing single-floor workflows.
3. Vertical traversal authoring exists and supports passability plus optional door/lock linkage.
4. Room indicators clearly convey above/below presence and traversal linkage status.
5. Focused runtime, architecture guardrail, and playback smoke gates pass.
6. No host contract changes are required.

## Design Lock-Off Questions

Use this section to ratify defaults before implementation. Each item should be marked with a final decision.

1. Runtime action path lock
- Question: Do we explicitly prohibit new vertical-only movement actions and require `UP`/`DOWN` to use the existing navigation action pipeline?
- Decision: Approved. Vertical traversal is an additive direction extension of existing navigation actions, not a new action family.

2. Vertical command vocabulary ownership
- Question: Should command words like `up`, `down`, `ascend`, `descend` be resolved only through producer-owned mappings (with canonical output `UP`/`DOWN`)?
- Decision: Approved. Directional vocabulary remains producer-owned; runtime movement consumes canonical direction outputs only, including `UP` and `DOWN`.

3. Reciprocal link policy
- Question: When authoring an `UP` link, should the designer auto-create the reciprocal `DOWN` link by default?
- Decision: Approved with parity. Follow existing traversal pattern: a single traversal connection represents both directions and defaults to `TwoWay` access.

4. One-way vertical traversal allowance
- Question: Are one-way vertical links allowed (for example, slide down but cannot climb up)?
- Decision: Approved. Vertical traversal supports both `TwoWay` and one-way access modes, with default `TwoWay` to preserve existing traversal parity.

5. Vertical door linkage model
- Question: Can one door/lock definition gate both directions, or must each leg own its own gate binding?
- Decision: Approved with strict parity. Use the exact same traversal door/lock support model as existing traversals (including per-leg state and Together/Independent binding options); the only change is traversal direction.

6. Story ownership model
- Question: Is story index metadata a map-placement concern (recommended) or intrinsic room identity?
- Decision: Approved. This is a map-placement concern, not a room identity concern.

Naming note (locked guidance):
1. Do not use `zOrder` or `z-index` terminology because those terms already have different semantics in the engine/rendering model.
2. Use `floorElevation` as the canonical field name for this feature.
3. `floorElevation` is an integer-only value: `0` is ground level, positive values are above ground, and negative values are below ground.

7. Default story migration behavior
- Question: For existing projects, should all rooms initialize to floorElevation `0` with no additional migration prompt?
- Decision: Approved. Legacy projects initialize room placement `floorElevation` to `0` with no migration prompt, and this must behave as a net-nothing compatibility migration (existing layouts continue to work unchanged).

8. Floor elevation constraints
- Question: Do we enforce floorElevation as integer-only with `0` ground, positive above-ground, and negative below-ground; and what bounds, if any, are enforced?
- Decision: Approved. `floorElevation` is integer-only; negative/zero/positive values are all valid, and no additional product-level min/max bounds are enforced in v1.

9. Room overlap semantics across floors
- Question: Can different-floor rooms share the same X/Y map coordinate without warning?
- Decision: Approved. Rooms on different `floorElevation` levels may share the same X/Y coordinates without warning; this is expected behavior (not just allowed).

10. Indicator semantics lock
- Question: Do we always show separate indicators for (a) room above/below existence and (b) traversal presence?
- Decision: Approved with compact-UI guidance. The UI must communicate both concepts (vertical room presence and vertical traversal presence) for up/down independently, but this may be implemented as either separate indicators or a single stateful indicator (state/color/tooltip) to minimize map real estate.

11. Vertical wizard entry points
- Question: Should vertical traversal editing launch from map indicators, room context menu, or both?
- Decision: Approved. Vertical traversal editing is available from both map indicator interaction and room context menu actions; map indicator is the primary shortcut.

Additional UI requirement:
1. If a room has both an above and a below relationship, the map UI must present a distinct visual state for each direction (`UP` and `DOWN`) so they are independently discoverable/editable.

12. Validation strictness for vertical pairs
- Question: Should missing reciprocal links be warning-level or error-level by default?
- Decision: Approved with directional-control clarity. Per-direction control is first-class: `UP` and `DOWN` may be intentionally different (including one-way now, reverse traversal enabled later via unlock/door/state changes). Missing reciprocal traversal is not itself an error when one-way intent is explicit. Integrity validation escalates only when authored intent is `TwoWay` and pair/state configuration is inconsistent.

13. Traversal uniqueness constraints
- Question: Per room/direction, do we allow only one vertical traversal target or multiple disambiguated targets?
- Decision: Approved. In v1, each room has at most one effective traversal target per direction (`UP` and `DOWN`) within an area context; multiple competing targets for the same direction are not allowed.

14. Replay assertion depth
- Question: For vertical movement, do replay tests require strict step-level parity by default or allow semantic mode for selected cases?
- Decision: Approved. Replay strictness for vertical movement follows the exact same strictness policy used by existing navigation/traversal coverage.

## Lock-Off Question Count

1. Total design lock-off questions in this plan: 14.
