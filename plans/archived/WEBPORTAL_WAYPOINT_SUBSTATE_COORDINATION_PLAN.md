# WebPortal Waypoint Substate Coordination Plan

Last updated: 2026-09-05
Status: Completed (core scope lock-off)

## Goal

Keep `SessionActive` as the top-level shell state while introducing a focused waypoint interaction substate (`WaypointMoveSetup`) coordinated above the renderer.

## Scope

1. WebPortal orchestration and UI coordination only.
2. Renderer interaction-mode API surface and behavior only.
3. No producer command grammar expansion in this plan.
4. No simulator architecture changes in this plan.
5. Top bar remains unchanged in this effort.

## Non-Goals

1. Replace state-driven shell composition with full substate-driven composition.
2. Introduce new runtime contract fields unless explicitly approved in lock-off.
3. Add hidden parser fallbacks or synonym command verbs.
4. Replace or redesign top bar behaviors/components.
5. Use top bar expansion as the primary path for in-game action controls.
6. Add host/game-engine emitted standalone styled-point instance control in v1.

## Implementation Plan (Small)

0. Prerequisite: define a new presentation cue type for waypoint point placement feedback.
- Add a dedicated presentation cue category/effect type for in-surface waypoint point placement.
- Define runtime behavior contract (trigger timing, playback ownership, and fallback behavior).
- Register baseline cue key(s) in the presentation cue catalog so `waypointDefaults.pointPlacementCueEffectKey` resolves to a valid effect.
- Add focused validation/tests for catalog resolution and fallback handling when cue keys are missing.

1. Define interaction-mode contract between coordinator and renderer.
- Add explicit mode and waypoint APIs: `setInteractionMode(mode)`, `getWaypointsSnapshot()`, `clearWaypoints()`, `removeLastWaypoint()`.
- Keep renderer command grammar agnostic; return canonical room-space points only.
- Source waypoint point presentation cue from `webportal-settings.v1.json` (`waypointDefaults.pointPlacementCueEffectKey`).

2. Add app-level substate in WebPortal coordinator layer.
- Keep top-level `SessionActive` unchanged.
- Introduce a coordinator-owned substate enum for gameplay interaction (for example `DefaultClick` and `WaypointMoveSetup`).
- Map substate changes to explicit renderer API calls.

3. Update controls to drive and reflect substate.
- Keep base shell composition resolved by state as-is.
- Add a dedicated in-game action control in column 1 under command handler.
- Default footprint is compact (single button row) in standard gameplay substate.
- In specific substates (including `WaypointMoveSetup`), augment with additional buttons and allow the control to grow to additional rows.
- Keep `SessionPlaySurfaceV1` mounted; branch click semantics by renderer mode, not by parsing shell state strings.

4. Define optional limited substate slot override behavior (if approved).
- Support additive per-substate slot patching for specific slots only.
- Merge rule: effective slots = base state/profile assignments + optional substate overrides.
- Unknown substate override keys are ignored with diagnostics.

5. Validation and regression hardening.
- Add unit tests for coordinator-to-renderer mode mapping.
- Add unit tests for renderer waypoint snapshot/clear behavior and command-click suppression in waypoint mode.
- Run `npm test` and `npm run build` for WebPortal.

## Lock-Off Questions

1. Substate vocabulary: should the canonical name be `WaypointMoveSetup` or a shorter host-only term?
2. Mode API shape: should renderer expose explicit methods or a single `SetInteractionMode(mode)` API?
3. Snapshot timing: should `GetWaypointsSnapshot` be allowed in all modes or only in waypoint mode?
4. Empty snapshot behavior: should empty return block submit in UI, or allow backend to validate?
5. Coordinate contract: should waypoint coordinates be rounded integers or preserve decimals?
6. Lifecycle on submit failure: keep waypoint draft for retry or always clear?
7. Cancel semantics: should cancel always clear and exit mode, or support exit-without-clear?
8. Cue taxonomy prerequisite: which cue category/type name should be canonical for waypoint point placement effects?
9. Visual ownership: should waypoint draft styling be fully renderer-owned with no host overlays?
10. In-game action control placement: should waypoint confirm/cancel live in a dedicated in-game action control in column 1 under command handler?
11. Substate slot overrides: do we enable limited per-substate slot patches now or defer until first proven need?
12. Diagnostics policy: should unknown substate or illegal mode transitions emit warning diagnostics in production?
13. Test gate: do we require one targeted end-to-end click-to-submit flow test before enabling by default?

## Lock-Off Decisions

1. Question 1 (Substate vocabulary)
- Decision: Use `WaypointMoveSetup` as the canonical v1 substate name.
- Status: Locked on 2026-09-04.

2. Question 2 (Mode API shape)
- Decision: Use a single mode setter on the renderer-facing API, with TypeScript naming `setInteractionMode(mode)`.
- Locked v1 mode values: `CommandClick`, `WaypointMoveSetup`.
- Companion renderer APIs in scope for this effort: `getWaypointsSnapshot()`, `clearWaypoints()`, and `removeLastWaypoint()`.
- Status: Locked on 2026-09-04.

3. Question 3 (Snapshot timing)
- Decision: `getWaypointsSnapshot()` is allowed in all modes and always returns a snapshot array (never `null`).
- In `CommandClick`, expected default is an empty array.
- Status: Locked on 2026-09-04.

4. Question 4 (Empty snapshot behavior)
- Decision: Block submit in UI when waypoint snapshot is empty.
- Coordinator confirms `waypointCount > 0` before dispatch; backend validation remains as defense in depth.
- Status: Locked on 2026-09-04.

5. Question 5 (Coordinate contract)
- Decision: v1 command payload formatting uses rounded integer room-space coordinates.
- Renderer remains room-space source of truth; coordinator applies rounding when constructing command text.
- Status: Locked on 2026-09-04.

6. Question 6 (Lifecycle on submit failure)
- Decision: Preserve waypoint draft on submit failure.
- Success and cancel clear the draft and exit waypoint mode; failure keeps draft and remains in waypoint mode for retry.
- Status: Locked on 2026-09-04.

7. Question 7 (Cancel semantics)
- Decision: In v1, cancel always clears waypoint draft and exits waypoint mode.
- Any future preserve-draft behavior requires a separate explicit action, not cancel overloading.
- Status: Locked on 2026-09-04.

8. Question 8 (Cue taxonomy prerequisite)
- Decision: Locked via generalized point-based effect cue work.
- Canonical v1 shape uses `StyledPointEffect` catalog entries for waypoint point placement cues (for example `waypoint.point.place.pulse.medium`).
- Status: Locked on 2026-09-05.

9. Question 9 (Visual ownership)
- Decision: Waypoint draft styling is renderer-owned for v1; host does not render duplicate waypoint overlays.
- Host remains owner of mode controls and status text.
- Status: Locked on 2026-09-04.

10. Question 10 (In-game action control placement)
- Decision: Do not use top bar for in-game actions.
- Implement a dedicated in-game action control in column 1 under command handler.
- Default to compact single-row height in standard gameplay and expand to additional rows in specific substates as needed.
- Top bar is explicitly unchanged by this effort.
- Status: Locked on 2026-09-04.

11. Question 11 (Substate slot overrides)
- Decision: Defer framework-level substate slot overrides.
- SessionActive composition remains unchanged; substate-aware behavior lives inside controls (including the new in-game action control).
- Status: Deferred on 2026-09-04.

14. Waypoint in-game tools row growth policy
- Decision: Keep current single-row tools layout for now.
- Follow-up trigger: revisit only if waypoint action button count grows enough to cause usability pressure.
- Status: Deferred follow-up on 2026-09-05.

12. Question 12 (Diagnostics policy)
- Decision: Emit non-blocking warning diagnostics for unknown substate and illegal mode transitions in production.
- Apply rate limiting/throttling to avoid repeated noise.
- Status: Locked on 2026-09-04.

13. Question 13 (Test gate)
- Decision: Require one targeted end-to-end waypoint click-to-submit flow test before enabling by default.
- Gate must validate mode entry, waypoint collection, confirm submit, rounded coordinate formatting, clear/reset behavior, and suppression of `pointclicked` dispatch while in waypoint mode.
- Status: Locked on 2026-09-04.

## Architecture Notes (MVP Scope Containment)

1. Introduce substates without changing state-composition framework behavior.
2. Keep controls shown by `SessionActive` stable; make selected controls substate-aware internally.
3. New in-game action control remains always present/visible in `SessionActive` and grows/contracts by substate.
4. New styled point cue configuration should support array-driven animation stops where useful (for example color, scale, alpha, and density) so pulse behavior can evolve over time similar to silhouette stop arrays.
5. For styled point cue fields, use arrays-only semantics for values that may vary over time; fixed values are represented as single-entry arrays.
6. Do not define duplicate scalar fallback properties for the same semantic values.
7. Prefer explicit direct configuration fields over `stylePreset` when the desired behavior can be expressed clearly with concrete settings.
8. Reserve `stylePreset` for cases where direct settings cannot express the intended behavior without excessive complexity.
9. Styled point lifetime semantics support two clear policies: `timebased` (requires `lifetimeMs`) and `manual-removal` (no auto-expire).
10. Manual-removal effects must be addressable by handle key and controlled through explicit renderer intent operations (`show` / `cancel`).
11. Deferred follow-on when engine emit support is added: define runtime instance operations carrying `instanceId`, point position, and action (`show` / `cancel`) so engine-owned standalone effects can be started and later canceled deterministically.
12. v1 explicitly keeps styled point emission local to WebPortal interaction flow and does not require host transport contract changes for standalone effect operations.
13. Temporary validation harness: current click-driven styled-point spawn uses `timebased` lifecycle with `lifetimeMs=10000` to verify both spawn and auto-expire behavior before waypoint/substate integration; remove this temporary harness when waypoint mode wiring is complete.

## Phased Execution Plan (Resumable)

Phase A: Cue Type Prerequisite and Taxonomy Lock
1. Decide generalized cue type/category naming and payload shape for point-placement feedback.
2. Extend catalog effect contract parsing for the new cue payload.
3. Seed at least one baseline cue entry in canonical presentation-effects catalogs.
4. Add resolver tests for lookup, invalid payload handling, and fallback behavior.
5. Model pulse/evolution behavior with optional stop arrays to match the established silhouette-style authoring approach.
6. Enforce array-only payload fields for style values; single-entry arrays are required when no animation variation is needed.
Checkpoint to save:
1. New cue type naming and payload rules are documented in this plan and validated by tests.
Resume rule:
1. If interrupted, restart at failing cue resolver tests first, then re-open taxonomy decision notes.

Phase B: Renderer Interaction-Mode API
1. Add renderer-facing APIs: `setInteractionMode(mode)`, `getWaypointsSnapshot()`, `clearWaypoints()`, `removeLastWaypoint()`.
2. Ensure waypoint draft is renderer-owned and snapshot returns immutable room-space data.
3. Keep command-click behavior unchanged in `CommandClick` mode.
Checkpoint to save:
1. Unit tests cover mode switch behavior and waypoint snapshot/clear invariants.
Resume rule:
1. If interrupted, verify mode default and snapshot immutability tests before touching coordinator wiring.

Phase C: Coordinator Substate Wiring
1. Add app-level interaction substate state machine while keeping top-level `SessionActive` unchanged.
2. Map substate transitions to renderer `setInteractionMode` calls.
3. Keep submit/clear lifecycle rules from locked decisions.
Checkpoint to save:
1. Coordinator tests pass for mode transitions and failure-path draft retention.
Resume rule:
1. If interrupted, run transition tests and inspect diagnostics for illegal transition warnings.

Phase D: In-Game Action Control
1. Introduce dedicated in-game action control in column 1 under command handler.
2. Keep compact one-row default in standard substate and grow rows in `WaypointMoveSetup`.
3. Keep top bar unchanged.
Checkpoint to save:
1. Component tests verify compact default layout, substate expansion, confirm disabled on empty snapshot.
Resume rule:
1. If interrupted, verify control rendering in `SessionActive` first, then substate-specific expansion.

Phase E: End-to-End Gate and Hardening
1. Add targeted end-to-end waypoint click-to-submit flow test per Q13 lock.
2. Verify `pointclicked` suppression in waypoint mode.
3. Run WebPortal validation gates (`npm test`, `npm run build`).
Checkpoint to save:
1. Test evidence and command outputs summarized in plan notes.
Resume rule:
1. If interrupted, run the targeted gate test before full test/build.

## Interruption Recovery Checklist

1. Re-open this plan and continue from the first incomplete phase.
2. Re-run tests for the current phase before writing new code.
3. Confirm locked decisions still hold (Q1-Q7, Q9-Q10, Q12-Q13).
4. Reconfirm deferred decisions (Q8 cue taxonomy, Q11 slot overrides) before expanding scope.

## Exit Criteria

1. `SessionActive` remains the active top-level state during waypoint setup flows.
2. Coordinator controls renderer mode with explicit intent APIs.
3. Renderer no longer emits `pointclicked` command dispatch while in waypoint build mode.
4. Coordinator can fetch final waypoint set via `GetWaypointsSnapshot` and clear it deterministically.
5. Lock-off decisions are captured for all questions above.

## Completion Evidence (2026-09-05)

1. Diagnostics policy completion (Q12)
- Added non-blocking warning diagnostics for illegal waypoint interaction actions.
- Added throttling/rate limiting to reduce repeated warning noise for identical illegal actions.
- Unknown substate and invalid action-context cases now classify to warning reasons and are ignored safely.

2. Validation coverage
- Focused waypoint/policy tests passed:
	- `npm test -- useHostWorkflow.waypointSubmitPolicy WaypointClickToSubmitFlow InGameToolsMenuV1`
- WebPortal build gate passed:
	- `npm run build`

3. Deferred follow-up retained by decision
- In-game tools multi-row growth behavior remains deferred until button count creates a concrete usability need.

## Fast Follower (Post-Core)

1. After the core WebPortal waypoint substate scope is complete, evaluate a backend target-resolution enhancement for `MoveRoomObjectByPoints` with a new `BestFit` resolution mode so UI continues submitting waypoints without requiring user-facing target mode selection.
2. Candidate precedence for `BestFit` evaluation is: if an object exists at the first waypoint, resolve that as movement target; otherwise, if a primary object is selected, resolve that as movement target; otherwise, keep existing unresolved-target behavior.
3. This item is explicitly out of current core scope and should be handled as an immediate fast follower design/implementation step.