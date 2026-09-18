# Waypoint Command Standardization Plan

## Goal
Standardize waypoint movement so click-driven and text-driven movement flow through the same command/action pipeline, with GameManager limited to host orchestration and DTO mapping.

## Why This Plan Exists
Current waypoint behavior improved after moving more logic toward shared movement mutation, but waypoint flow still has pipeline-specific planning and interpretation behavior in GameManager. This creates drift risk, makes failures harder to reason about, and complicates stack-landing expectations.

## Current Findings Captured
- End-to-end tracing now exists using CommandCorrelationId across:
  - host outbound waypoint request details
  - runtime waypoint normalization and leg interpretation
  - runtime per-leg/sub-leg execution outcomes
  - host inbound delta summary
- Observed/likely friction points:
  - click point to cell-anchor normalization can still produce unintuitive landing for larger footprints
  - stack-aware landing behavior is not fully first-class in the waypoint pipeline
  - GameManager still contains movement planning logic that should live in shared runtime services

## Scope
- In scope:
  - waypoint command standardization architecture
  - migration of waypoint planning/execution out of GameManager
  - new runtime action for waypoint movement
  - optional text-command waypoint syntax support in preprocessor
  - tests and diagnostics parity
- Out of scope (for this change):
  - broad movement physics redesign
  - generalized pathfinding
  - UI redesign in simulator

## Constraints and Guardrails
- Keep Storyboard.Simulator independent from StoryboardDesigner.App.
- Keep producer-owned grammar rules explicit (no hidden fallback vocabulary).
- Preserve host/runtime contract safety and avoid manual edits to generated contract DTO outputs.
- Prefer additive rollout with compatibility staging and deterministic diagnostics.

## Target Architecture
1. Introduce a new runtime command action type for waypoint movement (for example, MoveRoomObjectByWaypoints) with typed payload.
2. Move waypoint normalization, leg planning, decomposition, and execution into shared runtime service/action execution.
3. Keep GameManager responsible for:
   - correlation and command queueing
   - host result mapping
   - journaling and event publication integration
4. Route simulator click-waypoint submission into standard command/action processing rather than a unique side pipeline.
5. Optionally support explicit text syntax for waypoints in preprocessor (producer-controlled format).

## Direction Committed (2026-09-03)
1. Point-based gameplay operations will execute through the same command preprocess -> action binding -> runtime action execution pipeline as other commands.
2. Point tuples (single and ordered multi-point forms) will be parsed by the preprocessor and surfaced as structured command inputs for actions.
3. Verbs remain producer-owned and explicit; no hidden fallback verbs, directionals, or synonym shortcuts are introduced for point flows.
4. Runtime remains authoritative for coordinate normalization, leg interpretation, and movement legality.
5. Host gesture handling remains a host concern, but host submission uses standard command pipeline entry points instead of debugger-only or waypoint-only seams.
6. Primary and secondary room-object selection flows are treated as first-class command/action behaviors, not debugger-only point intent behavior.
7. Point-based movement actions must call the same shared movement mutation core used by existing movement actions; point-specific actions are wrappers/adapters, not alternate movement implementations.
8. Existing single numeric-argument command behavior (for example direction plus distance forms such as "move east 3") must remain fully compatible while adding point tuple parsing.
9. Object selection semantics use ordered selection state: first selected object is primary; subsequent selections are tracked as secondary selections (set-capable), and a clear-selections command resets to no selected objects.

## Point Command Examples (Intent-Level)
1. Single-point selection intent family: `selectobject (x,y)`.
2. Point-select plus move intent family: `selectandmoveobject (x0,y0) (x1,y1)` plus optional additional points for multi-leg movement.
3. Active-object move intent family: `moveprimaryactiveobject (x,y)` plus optional additional points for multi-leg movement.
4. Final producer-facing verb surface is configurable and can vary by game; these examples are design placeholders, not locked canonical names.

## Locked New Runtime Action Set (2026-09-03)
1. New action count for this initiative: 3.
2. Action: `SelectRoomObjectByPoint`.
3. Action: `MoveRoomObjectByPoints`.
4. Action: `ClearRoomObjectSelections`.
5. Composite command forms such as select-and-move should prefer linked-action composition over introducing a fourth dedicated action.

## Open Design Questions (Resolve Sequentially)
Process rule for this section:
1. Resolve exactly one question at a time.
2. Record the decision and rationale inline before moving to the next question.

### Q1. Action Strategy for Point Commands
Question:
Should point-capable behavior be implemented as new runtime action family members, or by augmenting existing movement/selection actions with point payload support?

Recommendation:
Adopt a new point-action family for this initiative.

Decision (2026-09-03):
Approved with caveat: point-action family is accepted only if movement execution reuses the existing shared movement mutation core.

Rationale:
1. Keeps existing action semantics stable and limits regression risk in already-shipped non-point behaviors.
2. Makes legacy seam retirement clearer by providing an explicit replacement surface.
3. Improves extensibility for future point-driven interactions without overloading existing action contracts.
4. Preserves movement parity across entry points and prevents special-case movement bugs.

### Q2. Preprocessor Point Argument Contract
Question:
What structured shape should the preprocessor emit for point tuples and ordered point sequences?

Recommendation:
Emit ordered point arguments with deterministic validation diagnostics and preserve original lexical order.

Decision (2026-09-03):
Approved with compatibility caveat: preserve current single numeric-argument parsing and semantics for existing movement commands while introducing multi-number point parsing.

Rationale:
1. Supports both single-point and multi-leg commands using one parse model.
2. Simplifies binder/action logic by avoiding host-side shape guessing.
3. Protects existing room-cell movement grammar behavior from regression as point command support expands.

### Q3. Selection Role Modeling
Question:
How should primary versus secondary selection intent be represented in command/action payloads?

Recommendation:
Model selection as ordered selection state instead of fixed per-command role targeting.

Decision (2026-09-03):
Approved with directional change: selection commands are object-selection based; first selected becomes primary, later selected objects become secondary selections, and the model supports N secondary selections.

Required companion command:
Add a clear-selections command (for example `clearobjectselections`) that resets to no selected objects.

Rationale:
1. Aligns user interaction with natural selection order rather than forcing separate primary/secondary target verbs.
2. Supports selection-set workflows needed for future multi-object interactions.
3. Eliminates dependency on debugger mutation interfaces for core gameplay behavior.

### Q4. Gesture-to-Verb Bootstrap Contract
Question:
How should hosts discover which producer-owned verbs to use for point gestures?

Recommendation:
Ship a producer-authored gesture-to-verb mapping payload at session attach/bootstrap.

Decision (2026-09-03):
Approved.

Rationale:
1. Keeps command grammar producer-owned.
2. Avoids hardcoded host verb assumptions across Simulator and WebPortal.

### Q5. Legacy Seam Retirement Order
Question:
What deprecation order should be used for legacy waypoint and debugger point-intent seams?

Recommendation:
Retire only after parity gates pass for Simulator and WebPortal command-pipeline paths, then remove legacy host contract methods/DTOs in one focused cleanup phase.

Decision (2026-09-03):
Approved.

Rationale:
1. Reduces production risk during migration.
2. Keeps compatibility staging explicit and time-bounded.

## Phased Plan
### Prerequisite Gate - Engine Isolation Proof (Before Simulator Changes)
1. Complete GameEngine point-command pipeline implementation (preprocessor parse, action binding, runtime execution).
2. Validate behavior manually through command text submission (no simulator gesture wiring) for representative commands:
  - single-point select command
  - active-object point move command
  - select-and-move point sequence command
  - clear selections command
3. Confirm diagnostics and outcome parity for success and deterministic failure cases.
4. Do not begin simulator migration work until this gate is captured as passing evidence in plan notes/test artifacts.

### Phase 0.5 - Designer Authoring Enablement
1. Expose `SelectRoomObjectByPoint`, `MoveRoomObjectByPoints`, and `ClearRoomObjectSelections` in Designer action-type pickers.
2. Ensure Designer payload defaults and persistence mappings are wired for project save/load and clean runtime export.
3. Keep UI changes intentionally minimal for this pass: actions must be selectable and persist correctly even if richer editor affordances follow in a later UX pass.

### Phase 0 - Baseline Lock
1. Preserve and use current trace outputs as behavioral evidence.
2. Capture representative traces for:
   - simple cardinal move
   - diagonal/off-axis move
   - stack landing attempt
   - known failure case where distance/direction feels wrong
3. Store traces in a test artifact note for regression comparison.

### Phase 1 - New Waypoint Runtime Action
1. Add new command action type and typed payload model in shared runtime contracts/services.
2. Add runtime executor that delegates movement mutation through existing shared gateway and multi-leg primitives.
3. Ensure result-code and telemetry behavior aligns with existing movement action patterns.

### Phase 2 - GameManager Thinning
1. Replace GameManager-specific waypoint planning logic with action request composition and execution dispatch.
2. Keep host-facing output and direct command echo behavior stable.
3. Preserve current CommandCorrelationId trace lines during migration.
4. Normalize waypoint user-facing message channels so gameplay-facing outcome text is emitted through one canonical output path (not mixed across diagnostics/output), and retire temporary acceptance/trace phrasing from player-visible echo expectations.

### Phase 3 - Optional Text Waypoint Syntax
1. Add explicit producer-owned syntax parse support (example family: verb target (x,y) ...).
2. Convert parsed coordinates into typed waypoint action payload.
3. Reject invalid formats with deterministic diagnostics.

### Phase 4 - Landing Semantics Hardening
1. Decide policy for stack-aware final placement:
   - strict exact-anchor policy (current style), or
   - support-aware landing candidate resolution for final destination.
2. If adopting support-aware resolution, implement as shared action/gateway policy, not host special case.
3. Add focused tests for full-support landing expectations and failure messaging.

### Phase 5 - Simulator Command-Driven Parity Migration
1. Route simulator surface clicks through command text submission for standard click selection (`pointclicked (x,y)`) in normal mode.
2. Add/confirm simulator waypoint plotting mode that buffers ordered points and submits one command-driven waypoint request path.
3. Ensure waypoint mode suppresses per-click `pointclicked` dispatch while plotting and only submits on explicit confirm.
4. Keep simulator UX intentionally basic (no web-style orchestration parity required), but align behavior contracts with WebPortal outcomes.
5. Add focused simulator tests for mode switch behavior, command text formatting, submit behavior, and clear/cancel semantics.

### Phase 6 - Legacy Interface Deprecation and Removal
1. After WebPortal and Simulator parity gates pass, deprecate legacy point/waypoint seams and migrate remaining call sites.
2. Remove legacy host contract seams in one focused cleanup pass:
  - `IHostRuntimeCommandProcessorClient.ProcessMoveByWaypoints(...)`
  - `IHostRuntimeGameDebugger.TryHandleRoomPointIntent(...)`
  - `HostRoomPointIntentRequest`
3. Keep removal gated by regression evidence from command pipeline tests plus simulator/web host behavior checks.
4. Document removal completion and residual compatibility decisions in plan closeout notes.

## Diagnostics Requirements for the Future Change
- Keep CommandCorrelationId as the single trace join key.
- Require trace coverage for:
  - raw submitted points
  - normalized points and derived cell labels
  - interpreted legs and decomposed sub-legs
  - per-leg mutation outcomes
  - final returned deltas and final runtime position/stack fields
- Keep diagnostics level-gated to avoid noisy default operation.

## Test Plan
1. GameManager focused tests for host contract behavior stability.
2. Command processor fixture tests for new action semantics.
3. Linked action tests for chained behavior.
4. Engine-isolation manual command-text validation for point-command verbs before simulator changes.
5. Playback regression tests for command/event sequencing stability.
6. Architecture separation guardrails to ensure no designer/simulator boundary regressions.

## Risks
- Behavior drift during extraction if old and new pathways overlap.
- Increased parse ambiguity if text waypoint syntax is not tightly specified.
- Regression in telemetry expectations when one plotted leg decomposes into multiple canonical legs.

## Mitigations
- Stage by phases and keep instrumentation active throughout.
- Add parity tests before deleting legacy pathway logic.
- Keep a temporary compatibility switch if needed for rollout safety.

## Legacy Interface Retirement Targets (End of Plan)
- `IHostRuntimeCommandProcessorClient.ProcessMoveByWaypoints(...)`
- `IHostRuntimeGameDebugger.TryHandleRoomPointIntent(...)`
- `HostRoomPointIntentRequest`

## Exit Criteria
- Waypoint movement is executed through the same command/action runtime pipeline primitives as textual movement.
- GameManager no longer owns waypoint movement planning/interpreting logic beyond orchestration/mapping.
- Trace evidence confirms consistent intent-to-interpretation-to-delta behavior for known scenarios.
- Focused runtime and architecture guardrail suites pass.
- Simulator click/waypoint interaction paths use command-driven flow parity with WebPortal for `pointclicked` and explicit waypoint submit behavior.
- Legacy point/waypoint host interfaces are removed (or explicitly documented with time-bounded deprecation rationale if temporary hold is required).

## Deferral Note
This plan is intentionally deferred while current work stays focused on events feature delivery. It is ready to execute when waypoint standardization is prioritized.
