# Host-Buffered Waypoint Movement Plan

Status: Completed 2026-08-01 (implemented and validated)
Owner: Storyboard.Shared runtime plus host adapters
Last updated: 2026-08-01

## 0. Closeout Summary (2026-08-01)

Completed outcomes:

1. Host-buffered waypoint submission is implemented end-to-end with runtime-authoritative validation.
2. Simulator supports selecting waypoint execution mode (`ProgressiveStopOnFirstFailure` vs `AtomicRollback`) at submit time.
3. Mode-aware waypoint coverage is in place for `ActiveObject` and `ObjectAtFirstWaypoint` using ChessDemo sample regression scenarios.
4. Direct-command echo sidecar skeleton generation and additive refresh behavior are implemented.

Lock-off decisions finalized in implementation:

1. Target resolution: Option A (explicit `targetResolutionIntent`).
2. Execution semantics: both `ProgressiveStopOnFirstFailure` and `AtomicRollback` supported; simulator exposes explicit mode selection.
3. Waypoint payload: host x/y points resolved by runtime.
4. Failure coding: distinct result codes are emitted (`UnresolvedTarget`, `InvalidWaypointPayload`, `WaypointOutOfBounds`, `LegBlockedByCollision`, `LegBlockedByRestriction`, `InvalidRestrictedLegDirection`, `LegCountExceeded`, `ExecutionModeRollbackApplied`).
5. Passive target handling: runtime movement eligibility governs execution (`isMovable` and restrictions).

Residual notes:

1. `Player`-mode ChessDemo coverage is intentionally deferred; ChessDemo-focused waypoint regressions currently target `ActiveObject` and `ObjectAtFirstWaypoint` only.

## 1. Purpose

Define a small, explicit plan for point-and-click movement where hosts buffer waypoint clicks locally and submit one blind runtime request.

This plan is intentionally scoped to lock decisions first, then implement.

## 2. Problem Statement

We need a host gesture flow that supports click-driven movement without pushing pathfinding into runtime or requiring host-side movement validity checks.

Required boundary:

1. Host sends intent requests, including invalid ones.
2. Runtime is authoritative for target resolution, movement legality, and failure reasons.
3. Host displays runtime outcomes and diagnostics; it does not pre-validate movement correctness.

## 3. Scope

In scope:

1. New request/response contract for waypoint movement intent.
2. Runtime-side target resolution policy and per-leg evaluation semantics.
3. Host-side buffering UX for waypoint plotting and explicit submit.
4. Regression tests for runtime behavior and host wiring.

Out of scope:

1. Automatic route discovery/pathfinding.
2. Designer authoring-model changes.
3. Replacing command text movement flows.

## 4. Locked Direction (Already Agreed)

1. Host supports a plotting mode that captures an ordered list of click waypoints locally.
2. Host submits buffered waypoints as one request, not one request per click.
3. Runtime evaluates validity; host is allowed to submit blind requests.
4. Runtime returns deterministic success or failure outcome.
5. No hidden automatic leg discovery in runtime for this feature.

## 5. Lock-Off Questions (Decision Set)

1. Target resolution policy:
2. Option A: target resolution intent required (`ObjectAtFirstWaypoint`, `ActiveObject`, `Player`) with no hidden fallback chain.
3. Option B: target resolution intent optional; if omitted runtime applies default (`ActiveObjectThenPlayer`).
4. Option C: support target resolution intent plus explicit target id override for advanced hosts.
Decision default: Option A for strict host-neutral intent and deterministic runtime behavior.

5. Execution semantics for multi-leg requests:
6. Option A: progressive stop-on-first-failure (keep completed legs).
7. Option B: atomic all-or-nothing (rollback completed legs on any failure).

8. Waypoint payload format:
9. Option A: raw x/y points, runtime resolves cells.
10. Option B: resolved cell ids from host.
11. Option C: support both forms, prefer one canonical form.

12. Failure coding granularity:
13. Option A: single high-level failure code plus diagnostics detail.
14. Option B: distinct failure codes for unresolved target, invalid waypoint, blocked leg, restriction failure, out-of-bounds.

15. Passive target handling:
16. Option A: passive targets are always non-movable in waypoint flow.
17. Option B: use existing move action eligibility only, no extra passive rule.

18. Direct-command echo source:
19. Option A: runtime-only fallback text generated from command result code.
20. Option B: project-owned direct-command echo map looked up and evaluated by runtime.
21. Option C: host-owned text mapping (not preferred; bypasses producer ownership).

22. Off-axis leg geometry handling:
23. Option A: always require each leg to resolve to one of 8 directions for all targets.
24. Option B: require 8-direction legs only when target has movement restrictions; allow freeform vectors for unrestricted targets.
25. Option C: auto-normalize off-axis legs into multiple legal sub-legs (not preferred; introduces hidden path behavior).
Decision default: Option B with whole-request rejection when restricted target includes any off-axis leg.

26. Maximum legs per command restriction:
27. Option A: no explicit max-leg restriction; rely on existing distance/restriction rules only.
28. Option B: support `maxLegCountPerCommand` in movement restrictions and reject requests exceeding the configured cap.
29. Option C: force single-leg globally for all waypoint commands.
Decision default: Option B; enforce in preflight and allow per-object policy (for example, queen = 1 leg).

## 6. Proposed Contract Shape (Draft)

Concrete shape is still draft and not yet locked. Use this as the discussion baseline for lock-off.

### 6.1 Request (Draft JSON)

```json
{
	"requestKind": "MoveByWaypoints",
	"commandCorrelationId": 207,
	"intent": {
		"targetResolutionIntent": "ObjectAtFirstWaypoint"
	},
	"waypoints": [
		{ "x": 320, "y": 128 },
		{ "x": 320, "y": 192 },
		{ "x": 384, "y": 192 }
	],
	"options": {
		"executionMode": "ProgressiveStopOnFirstFailure",
		"persistLegTelemetry": true
	},
	"diagnosticsLevel": "Medium"
}
```

Notes:

1. `intent.targetResolutionIntent` is user-declared intent, not host-side object identity.
2. Recommended target resolution intents: `ObjectAtFirstWaypoint`, `ActiveObject`, `Player`.
3. Host does not resolve object ids for active object or player; runtime resolves at execution time.
4. `ObjectAtFirstWaypoint` means runtime must resolve a movable object at waypoint index 0 or fail with `UnresolvedTarget`.
5. `waypoints` are ordered and represent intermediate plus final destinations.
6. `executionMode` is one of the lock-off choices in Section 5.

### 6.2 Success Response (Draft JSON)

```json
{
	"requestKind": "MoveByWaypoints",
	"commandCorrelationId": 207,
	"hostResultCode": "Success",
	"success": true,
	"matchedCommand": true,
	"roomChange": null,
	"roomObjectChanges": [
		{
			"changeKind": "Updated",
			"objectId": "9de4a8eb-9b12-4f69-a6d0-4065380f4e4b",
			"objectName": "Player",
			"renderableRoomObject": {
				"x": 384,
				"y": 192
			}
		}
	],
	"moveLegTelemetry": [
		{
			"targetObjectId": "9de4a8eb-9b12-4f69-a6d0-4065380f4e4b",
			"targetObjectName": "Player",
			"legIndex": 0,
			"requestedDirection": "S",
			"requestedDistanceInCells": 1,
			"appliedDistanceInCells": 1,
			"success": true,
			"resultCode": "Moved",
			"fromX": 320,
			"fromY": 64,
			"toX": 320,
			"toY": 128,
			"travelVisualizationMode": "LegByLeg"
		},
		{
			"targetObjectId": "9de4a8eb-9b12-4f69-a6d0-4065380f4e4b",
			"targetObjectName": "Player",
			"legIndex": 1,
			"requestedDirection": "S",
			"requestedDistanceInCells": 1,
			"appliedDistanceInCells": 1,
			"success": true,
			"resultCode": "Moved",
			"fromX": 320,
			"fromY": 128,
			"toX": 320,
			"toY": 192,
			"travelVisualizationMode": "LegByLeg"
		},
		{
			"targetObjectId": "9de4a8eb-9b12-4f69-a6d0-4065380f4e4b",
			"targetObjectName": "Player",
			"legIndex": 2,
			"requestedDirection": "E",
			"requestedDistanceInCells": 1,
			"appliedDistanceInCells": 1,
			"success": true,
			"resultCode": "Moved",
			"fromX": 320,
			"fromY": 192,
			"toX": 384,
			"toY": 192,
			"travelVisualizationMode": "LegByLeg"
		}
	],
	"outputLines": [
		"Moved Player through 3 plotted legs."
	],
	"diagnostics": [
		"Target resolved from intent mode: Player.",
		"All waypoint legs completed."
	],
	"directCommandOutcome": {
		"commandResultCode": "Completed",
		"resolvedTargetObjectId": "9de4a8eb-9b12-4f69-a6d0-4065380f4e4b",
		"movementOutcome": {
			"targetResolutionStrategy": "Player",
			"completedLegCount": 3,
			"failedLegIndex": null
		}
	}
}
```

### 6.3 Failure Response (Draft JSON)

```json
{
	"requestKind": "MoveByWaypoints",
	"commandCorrelationId": 207,
	"hostResultCode": "Failure",
	"success": false,
	"matchedCommand": true,
	"roomChange": null,
	"roomObjectChanges": [
		{
			"changeKind": "Updated",
			"objectId": "9de4a8eb-9b12-4f69-a6d0-4065380f4e4b",
			"objectName": "Red Crate",
			"renderableRoomObject": {
				"x": 320,
				"y": 128
			}
		}
	],
	"moveLegTelemetry": [
		{
			"targetObjectId": "9de4a8eb-9b12-4f69-a6d0-4065380f4e4b",
			"targetObjectName": "Red Crate",
			"legIndex": 0,
			"requestedDirection": "S",
			"requestedDistanceInCells": 1,
			"appliedDistanceInCells": 1,
			"success": true,
			"resultCode": "Moved",
			"fromX": 320,
			"fromY": 64,
			"toX": 320,
			"toY": 128,
			"travelVisualizationMode": "LegByLeg"
		},
		{
			"targetObjectId": "9de4a8eb-9b12-4f69-a6d0-4065380f4e4b",
			"targetObjectName": "Red Crate",
			"legIndex": 1,
			"requestedDirection": "E",
			"requestedDistanceInCells": 1,
			"appliedDistanceInCells": 0,
			"success": false,
			"resultCode": "RestrictionFailed",
			"fromX": 320,
			"fromY": 128,
			"toX": 320,
			"toY": 128,
			"travelVisualizationMode": "LegByLeg"
		}
	],
	"outputLines": [
		"Movement failed on plotted leg 2."
	],
	"diagnostics": [
		"Leg 2 blocked by movement restriction.",
		"Direction E exceeds allowed distance for current category."
	],
	"directCommandOutcome": {
		"commandResultCode": "LegBlockedByRestriction",
		"resolvedTargetObjectId": "9de4a8eb-9b12-4f69-a6d0-4065380f4e4b",
		"movementOutcome": {
			"targetResolutionStrategy": "ActiveObject",
			"completedLegCount": 1,
			"failedLegIndex": 1
		}
	}
}
```

### 6.4 Command Result Code Candidate Set (Draft)

1. `Completed`
2. `PartialCompleted`
3. `UnresolvedTarget`
4. `InvalidWaypointPayload`
5. `WaypointOutOfBounds`
6. `LegBlockedByCollision`
7. `LegBlockedByRestriction`
8. `InvalidRestrictedLegDirection`
9. `LegCountExceeded`
10. `ExecutionModeRollbackApplied`

### 6.5 Direct-Command-Unique Response Data (Compared To Standard Command Response)

The following fields are specific to host-driven direct commands and are additive over the existing command response envelope:

1. `directCommandOutcome.commandResultCode`
- Domain outcome for direct-command execution (`Completed`, `PartialCompleted`, `UnresolvedTarget`, etc.).
- Distinct from host envelope result (`hostResultCode`) used by existing command flow.
2. `directCommandOutcome.resolvedTargetObjectId`
- Resolved runtime target identity shared across direct-command types.
3. `directCommandOutcome.movementOutcome`
- Movement-only outcome container that isolates movement-specific fields from generic direct-command result metadata.
4. `directCommandOutcome.movementOutcome.targetResolutionStrategy`
- Machine-readable runtime-applied target resolution strategy for this request (`Player`, `ActiveObject`, `ObjectAtFirstWaypoint`).
5. `directCommandOutcome.movementOutcome.completedLegCount`
- Convenience summary count of successfully applied legs.
- Can be derived from `moveLegTelemetry`, but included to simplify host summaries.
6. `directCommandOutcome.movementOutcome.failedLegIndex`
- Index of the first failed leg when failure occurs, else `null`.
- Can be derived from `moveLegTelemetry`, but included for direct host UX handling.

### 6.6 Project-Owned Direct Command Echo Map (Draft)

Runtime should load a producer-authored mapping file from clean export artifacts and evaluate mapped script text through `ActionScriptEvaluationService`.
Host does not map keys; host only renders returned `outputLines`.
The echo map must be emitted as a sidecar file parallel to the main clean project file, not embedded inside the main clean project JSON.

Draft file location:

1. `<CleanExportFolder>/<ProjectName>.sbe.clean.direct-command-echoes.json`
2. This file sits next to `<ProjectName>.sbe.clean.json` in the same clean export folder.

Draft lookup key:

1. `requestKind`
2. `targetResolutionIntent`
3. `commandResultCode`

Draft JSON:

```json
{
	"version": "1.0",
	"directCommandEchoes": [
		{
			"requestKind": "MoveByWaypoints",
			"targetResolutionIntent": "Player",
			"commandResultCode": "Completed",
			"script": "Player movement completed across plotted waypoints."
		},
		{
			"requestKind": "MoveByWaypoints",
			"targetResolutionIntent": "ObjectAtFirstWaypoint",
			"commandResultCode": "UnresolvedTarget",
			"script": "No movable object was found at the first plotted point."
		},
		{
			"requestKind": "MoveByWaypoints",
			"targetResolutionIntent": "ActiveObject",
			"commandResultCode": "LegBlockedByRestriction",
			"script": "Movement blocked by restriction on a plotted leg."
		}
	]
}
```

Notes:

1. Phase 1 can treat `script` as static text only.
2. Phase 2 can allow token replacement/evaluation through `ActionScriptEvaluationService` using a small direct-command token set.
3. Runtime fallback chain when no entry exists: exact key match -> requestKind + hostResultCode default -> requestKind default -> built-in generic text that emits `directCommandOutcome.commandResultCode`.
4. Keep this mechanism generic so future host-driven direct commands can reuse it.

### 6.7 Skeleton Generation And Refresh Strategy (Draft)

Goal: ensure each project can materialize a starter set of known `(requestKind, targetResolutionIntent, commandResultCode)` combinations and refresh safely as runtime support expands.

Registry source of truth:

1. Add a runtime registry in shared code for direct-command descriptors.
2. Each descriptor declares:
- `requestKind`
- supported `targetResolutionIntent` values
- supported `commandResultCode` values
3. Registry is code-owned by runtime, not host.

Initial skeleton generation:

1. On first runtime load, if `<CleanExportFolder>/<ProjectName>.sbe.clean.direct-command-echoes.json` is missing, runtime creates it.
2. Runtime emits one entry per cartesian combination of:
- registered `requestKind`
- each supported `targetResolutionIntent`
- each supported `commandResultCode`
3. Generated entries use empty or generic script text placeholders.
4. File is written in deterministic ordering to minimize source-control churn.

Refresh/update behavior for later runtime additions:

1. Provide an explicit non-destructive refresh operation (runtime API + optional simulator menu action).
2. Refresh adds newly introduced combinations that do not yet exist in file.
3. Refresh never overwrites producer-authored scripts for existing keys.
4. Refresh can optionally mark deprecated keys in diagnostics, but does not delete by default.
5. Optional strict cleanup mode can remove deprecated keys only when user explicitly requests cleanup.

Matching rules for identity and merge:

1. Identity key is exact triple: `(requestKind, targetResolutionIntent, commandResultCode)`.
2. Comparison is case-insensitive for matching, canonical casing on write.
3. Unknown extra fields are preserved for forward compatibility.

Versioning:

1. Include top-level `version` in file.
2. For additive registry growth, keep same major schema version.
3. On breaking schema changes, bump version and provide migration logic.

Draft phase-2 token vocabulary (small initial set):

1. `currentRoom.name`
2. `target.name`
3. `target.id`
4. `blockedBy.name`
5. `command.resultCode`
6. `command.targetResolutionIntent`
7. `command.completedLegCount`
8. `command.failedLegIndex`

Token syntax is intentionally not locked in this plan; choose syntax that is directly compatible with existing `ActionScriptEvaluationService` parsing conventions.

### 6.8 Off-Axis Leg Examples (Draft)

These examples make the locked off-axis behavior explicit.

#### 6.8.1 Restricted Target Off-Axis Rejection (Failure)

Input intent:

1. Target has movement restrictions.
2. First plotted leg is off-axis (`dx=96`, `dy=64`), which is not one of canonical 8 directions.

Result:

1. Runtime rejects request before applying movement.
2. `commandResultCode` is `InvalidRestrictedLegDirection`.
3. `failedLegIndex` points to first invalid leg.

```json
{
	"requestKind": "MoveByWaypoints",
	"commandCorrelationId": 611,
	"hostResultCode": "Failure",
	"success": false,
	"matchedCommand": true,
	"roomChange": null,
	"roomObjectChanges": [],
	"moveLegTelemetry": [],
	"outputLines": [
		"Movement failed: plotted leg 1 is not a valid restricted-direction leg."
	],
	"diagnostics": [
		"Restricted target requires canonical 8-direction legs.",
		"Leg 1 vector dx=96, dy=64 is off-axis."
	],
	"directCommandOutcome": {
		"commandResultCode": "InvalidRestrictedLegDirection",
		"resolvedTargetObjectId": "9de4a8eb-9b12-4f69-a6d0-4065380f4e4b",
		"movementOutcome": {
			"targetResolutionStrategy": "ActiveObject",
			"completedLegCount": 0,
			"failedLegIndex": 0
		}
	}
}
```

#### 6.8.2 Unrestricted Target Off-Axis Acceptance (Success)

Input intent:

1. Target has no movement restrictions.
2. A plotted leg is off-axis (`dx=96`, `dy=64`).

Result:

1. Runtime allows freeform vector for unrestricted target.
2. Movement is applied to requested destination points.

```json
{
	"requestKind": "MoveByWaypoints",
	"commandCorrelationId": 612,
	"hostResultCode": "Success",
	"success": true,
	"matchedCommand": true,
	"roomChange": null,
	"roomObjectChanges": [
		{
			"changeKind": "Updated",
			"objectId": "2f0ab802-f94d-4d2f-8f69-e7d8c2f985c3",
			"objectName": "Paper Plane",
			"renderableRoomObject": {
				"x": 416,
				"y": 224
			}
		}
	],
	"moveLegTelemetry": [
		{
			"targetObjectId": "2f0ab802-f94d-4d2f-8f69-e7d8c2f985c3",
			"targetObjectName": "Paper Plane",
			"legIndex": 0,
			"requestedDirection": "Freeform",
			"requestedDistanceInCells": 0,
			"appliedDistanceInCells": 0,
			"success": true,
			"resultCode": "Moved",
			"fromX": 320,
			"fromY": 160,
			"toX": 416,
			"toY": 224,
			"travelVisualizationMode": "LegByLeg"
		}
	],
	"outputLines": [
		"Moved Paper Plane through plotted waypoints."
	],
	"diagnostics": [
		"Target has no movement restrictions; off-axis leg allowed."
	],
	"directCommandOutcome": {
		"commandResultCode": "Completed",
		"resolvedTargetObjectId": "2f0ab802-f94d-4d2f-8f69-e7d8c2f985c3",
		"movementOutcome": {
			"targetResolutionStrategy": "ObjectAtFirstWaypoint",
			"completedLegCount": 1,
			"failedLegIndex": null
		}
	}
}
```

All other fields in Sections 6.2 and 6.3 intentionally follow the existing command response style:

1. `hostResultCode`
2. `success`
3. `matchedCommand`
4. `roomChange`
5. `roomObjectChanges`
6. `moveLegTelemetry`
7. `outputLines`
8. `diagnostics`

## 7. Runtime Behavior Rules (Draft)

1. Resolve movement target using locked target policy.
2. Validate request shape (non-empty waypoint list, valid numeric values).
3. Convert waypoints to ordered legs from current object position through each point.
4. Determine whether target has movement restrictions that require 8-direction evaluation.
5. If target is restriction-governed, validate every leg direction as one of the canonical 8 directions before execution.
6. If any restricted-target leg is off-axis, reject the whole request before movement with `commandResultCode=InvalidRestrictedLegDirection` and set `failedLegIndex` to first invalid leg.
7. If target defines `maxLegCountPerCommand`, validate total leg count before execution and reject with `commandResultCode=LegCountExceeded` when request exceeds configured limit.
8. If target is unrestricted, allow freeform leg vectors and apply movement directly to provided points.
9. Evaluate each executable leg using existing movement restrictions and occupancy rules.
10. Produce deterministic command result code and per-leg telemetry.
11. Populate direct-command outcome metadata with general fields at `directCommandOutcome` and movement-specific fields under `directCommandOutcome.movementOutcome`.
12. Resolve echo script from project-owned direct-command echo map using (`requestKind`, `targetResolutionIntent`, `commandResultCode`).
13. Evaluate resolved script through `ActionScriptEvaluationService` and append final text to `outputLines`.
14. If no matching map entry exists, emit deterministic runtime fallback output text that includes `directCommandOutcome.commandResultCode`.
15. Never rely on host-side validity assumptions.

## 8. Host UX Rules (Draft)

1. Plotting mode is explicit and visible.
2. Clicks append waypoints in order.
3. Host provides undo-last, clear-all, and submit controls.
4. Host submit supports invocation choices: `Move Object At First Waypoint`, `Move Active Object`, `Move Player`.
5. For simulator MVP, invocation choices are exposed through a right-click menu after plotting.
6. Host can submit without prechecking target validity.
7. Host renders runtime success or failure message and final state.

## 9. Implementation Phases

1. Phase 0: completed.
2. Phase 1: completed.
3. Phase 2: completed.
4. Phase 3: completed.
5. Phase 4: completed.
6. Phase 5: completed.
7. Phase 6: completed (targeted mode-aware waypoint tests added for `ActiveObject` and `ObjectAtFirstWaypoint`).

## 10. Acceptance Criteria

1. Host can submit a waypoint movement request with no pre-validation.
2. Runtime can resolve target via locked policy and report clear failure if unresolved.
3. Multi-leg movement honors existing movement restrictions per leg.
4. Failure behavior matches locked execution semantics.
5. Existing command-based movement behavior remains unchanged.
6. Focused runtime regression gate passes.

## 11. Validation Plan

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"
3. Add focused tests for waypoint request success, unresolved target, blocked intermediate leg, and stop mode behavior.

## 12. Lock-Off Checklist

1. Confirmed and implemented.
2. Confirmed and implemented.
3. Confirmed and implemented.
4. Confirmed and implemented.
5. Confirmed and implemented.
6. Plan status updated to completed.

## 13. Design Questions For Lock-Off

1. Should `intent.targetResolutionIntent` be required for all direct-command requests, or may runtime apply a default mode when omitted?
2. For `ObjectAtFirstWaypoint`, do we require the first waypoint to be both object-resolvable and movable before any leg execution?
3. For multi-leg execution semantics, do we lock `ProgressiveStopOnFirstFailure` now, or keep `AtomicRollback` as a supported option in v1?
4. For restricted targets, do we lock whole-request rejection on first off-axis leg with `InvalidRestrictedLegDirection` before any movement?
5. For unrestricted targets, do we lock freeform vector acceptance as-is, or normalize vectors into a canonical directional representation for telemetry only?
6. Should `moveLegTelemetry.requestedDirection` allow `Freeform` token explicitly, or should off-axis unrestricted legs be represented by empty direction plus coordinates only?
7. Should `directCommandOutcome.movementOutcome.targetResolutionStrategy` always echo the runtime-applied strategy actually executed, even if request intent was omitted or transformed?
8. Which `commandResultCode` values are mandatory for v1 (`Completed`, `UnresolvedTarget`, `InvalidRestrictedLegDirection`, `LegBlockedByRestriction`, etc.)?
9. Should movement-specific summary fields (`directCommandOutcome.movementOutcome.completedLegCount`, `directCommandOutcome.movementOutcome.failedLegIndex`) remain first-class fields, or be derived by hosts from `moveLegTelemetry` only?
10. Should project-owned echo map lookup require exact triple match first, then fallback order (`requestKind+hostResultCode`, then requestKind default), as currently drafted?
11. In v1, do we lock direct-command echo map scripts to static text only, with ActionScript token evaluation gated to v2?
12. What minimum direct-command token set is required for first dynamic scripting pass (`currentRoom.name`, `target.name`, `command.resultCode`, etc.)?
13. How should runtime behave when echo script evaluation fails: emit fallback generic text only, or append both script error diagnostic and fallback text?
14. Should direct-command echo skeleton refresh be runtime-triggered on load, manual-only, or both with a configuration flag?
15. During skeleton refresh, do we preserve deprecated keys indefinitely by default, or mark with explicit `deprecated` metadata in file?
16. Should direct-command echo map be included in clean export artifacts as a sidecar file parallel to `<ProjectName>.sbe.clean.json` (simulator/runtime load path), or remain native project-side runtime config only?
17. Do we need deterministic ordering guarantees for both generated skeleton entries and persisted producer edits to reduce source-control churn?
18. What regression matrix is required before lock: restricted off-axis rejection, unrestricted off-axis acceptance, unresolved target, blocked intermediate leg, and echo fallback behavior?
19. Should `maxLegCountPerCommand` live in existing movement restrictions and default to unlimited when omitted?
