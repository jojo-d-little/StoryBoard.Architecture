# Simulator CurrentSession Isolation Plan

## Goal
Constrain `CurrentSession` usage to debugger-only responsibilities, with runtime-state tab as the only direct consumer in thin mode planning.

## Why This Plan Exists
`CurrentSession` is currently used across simulator behaviors beyond runtime-state tree projection. Some uses expose host-contract gaps that should become thin-safe API fields, while other uses are fallback or convenience logic that should be removed or routed through existing presentation/session-delta APIs.

## Status Update (2026-08-25)
1. `SimulatorViewModel` `CurrentSession` usage was reduced from 35 references to 1 approved reference.
2. The only remaining direct read is runtime-state tab tree materialization in `RebuildGameStateTree`.
3. Non-debugger fallback reads were removed; readiness checks were migrated to `_gameManager.HasActiveRuntime` where appropriate.
4. Guardrail test coverage now enforces an allowlist for `SimulatorViewModel` `CurrentSession` usage.
5. Runtime-state tab and debugger actions are now gated by centralized simulator capabilities (`SupportsDebuggerRuntimeState`, `SupportsLocalRuntimeProjectLoad`, `SupportsDebuggerMutations`).
6. Regression coverage now includes capability-gating tests plus full-solution validation with all tests passing.
7. Capability values are now sourced from host contract implementation via `IHostRuntimeCommandProcessorClient.SupportsCapability(...)` (with legacy fallback only when host leaves a capability undeclared).

## Closure Status (2026-08-25)
Status: Closed and archive-ready.

Closure summary:
1. All non-debugger simulator fallback reads from `CurrentSession` were removed.
2. Remaining direct graph usage is intentionally limited to runtime-state debugger surface.
3. Runtime-state tab and debugger mutation behaviors are capability-gated.
4. Capability truth is host-owned via `IHostRuntimeCommandProcessorClient.SupportsCapability(...)`.
5. Focused regression gates and full solution tests passed at closeout.

## Outcomes
1. Classify every `CurrentSession` dependency as one of:
- Contract Gap
- Debugger-Only Allowed
- Fallback Debt
2. Remove or isolate all non-allowed usages.
3. Keep simulator behavior stable in thick mode.
4. Enable thin mode to disable debugger/runtime-state features cleanly.

## Scope
In scope:
1. `Storyboard.Simulator` composition and `SimulatorViewModel` use paths.
2. Minimal host-contract additions for thin-safe data needs.
3. Focused simulator tests for mode gating and behavior parity.

Out of scope:
1. Final remote strategy for debugger `CurrentSession` payload shape.
2. Full redesign of runtime-state tab projection model.

## Classification Rules
1. Contract Gap
- Data is required by non-debugger UX in both thick and thin modes.
- Data should come from thin-safe interfaces (`IHostRuntimeCommandProcessorClient`, `ISessionDeltaPolling`, `IHostSessionManagementClient`, discovery/asset interfaces).

2. Debugger-Only Allowed
- Feature is explicitly debug tooling and can be disabled in thin mode.
- Example: direct variable mutation or debug room jump.

3. Fallback Debt
- Logic uses `CurrentSession` only as backup when better host-facing payloads are expected.
- Should be removed or replaced by deterministic host payload usage.

## Initial Dependency Buckets
1. Contract Gap candidates:
- Render surface dimensions consumed by simulator display setup.
- Any non-debugger room metadata required to render baseline UX when no local runtime object graph exists.

2. Debugger-Only Allowed candidates:
- `LoadGameRuntimeProject` path-based local loading.
- `TryMovePlayerToRoom`, `TrySetVariable`, `TryHandleRoomPointIntent` actions.
- Runtime-state tab tree source from full runtime object graph.

3. Fallback Debt candidates:
- Project/game identity fallback from `CurrentSession.ProjectName` when session/discovery identity should be authoritative.
- Narrative and preview fallback paths that read runtime graph after command/poll payload handling.

## Phases

## Phase 0: Usage Inventory and Classification Lock
1. Build a line-by-line inventory of `CurrentSession` references in simulator.
2. Assign each reference to Contract Gap, Debugger-Only Allowed, or Fallback Debt.
3. Review and lock classification before behavior changes.

Deliverable:
1. Checked-in inventory table in this plan.

## Phase 1: Thin-Mode Capability Model
1. Add simulator runtime mode model:
- `ThickLocal`
- `ThinRemote`
2. Add centralized capability flags:
- `SupportsDebuggerRuntimeState`
- `SupportsLocalRuntimeProjectLoad`
- `SupportsDebuggerMutations`
3. Route command enable/disable logic through capabilities, not raw `CurrentSession` checks.

Deliverable:
1. Runtime-state tab and debugger actions can be disabled by capability.

## Phase 2: Contract Gap Patches
1. Add thin-safe host payload fields needed by non-debugger UX.
2. Prefer additive contract updates only.
3. Source required UI state from attach/presentation/baseline/delta envelopes.

Candidate additions (subject to lock):
1. Render surface dimensions in baseline/current presentation envelope.
2. Stable game identity metadata for asset-cache and labeling flows.
3. Optional narrative path display fields if currently inferred from runtime graph fallback.

Deliverable:
1. Non-debugger simulator views no longer need `CurrentSession` reads.

## Phase 3: Fallback Debt Removal
1. Remove `CurrentSession` fallback branches where thin-safe payloads are authoritative.
2. Keep diagnostics for missing payload fields rather than silently reading local graph.
3. Ensure deterministic precedence:
- Host payload first
- Explicit warning if incomplete
- No debugger graph fallback in thin-safe paths

Deliverable:
1. Fallback debt eliminated from non-debugger paths.

## Phase 4: Debugger Surface Containment
1. Keep remaining direct `CurrentSession` reads behind one debugger facade/service.
2. Restrict direct access call-sites to runtime-state tab and debugger action handlers only.
3. Add guard tests to fail if non-debugger areas read debugger session graph.

Deliverable:
1. `CurrentSession` usage is intentionally contained and auditable.

## Validation and Tests
1. Existing regression gates:
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`
2. Focused simulator tests to add/update:
- Thin mode disables runtime-state tab and debugger mutation commands.
- Thick mode retains current debugger behavior.
- Non-debugger rendering and command flows succeed without `CurrentSession` reads.

## Done Criteria
1. All `CurrentSession` references are classified and justified.
2. No non-debugger simulator path directly reads `CurrentSession`.
3. Runtime-state tab is the only direct graph consumer in thick mode.
4. Thin mode runs with runtime-state tab disabled and no debugger graph dependency.
5. Required thin-safe contract fields are additive and covered by tests.

## Open Questions
1. Should render surface dimensions live in every presentation payload or only attach/baseline payloads with caching?
2. Should narrative path labels be computed server-side to avoid graph fallback entirely?
3. Should debug room click intent remain debugger-only for thin mode v1, or receive thin-safe command equivalents later?

## Discovery Details To Collect (Phase 0 Inputs)
1. Full CurrentSession usage inventory in `Storyboard.Simulator/ViewModels/SimulatorViewModel.cs` with:
- line number
- calling method
- feature area (runtime-state tab, command UX, render preview, narrative, assets, playback, diagnostics)
- provisional classification (Contract Gap, Debugger-Only Allowed, Fallback Debt)
2. Thin-safe payload availability matrix from shared contracts:
- `HostRuntimePresentationResult`
- `HostSessionDataEnvelope`
- `HostAttachSessionResult.InitialPresentation`
- `HostSessionDescriptor`
3. Explicit missing data matrix for non-debugger UX:
- required field
- current source (`CurrentSession` vs host payload)
- candidate additive contract target
4. Startup/composition map for thick vs thin mode in simulator:
- mode selector source
- capability flags
- UI command/tab gating points
5. Fallback precedence map for current behavior:
- where simulator currently prefers host payload
- where simulator falls back to `CurrentSession`
- where fallback should be removed entirely
6. Test impact map:
- tests that assume direct `CurrentSession` availability
- tests that should become mode-aware
- guardrail tests needed to block future bleed-over

## Phase 0 First-Pass Inventory (Prefilled)

Notes:
1. This is a first-pass classification to accelerate lock decisions.
2. Some rows are marked `Contract Gap or Fallback Debt` where final classification depends on lock answers.
3. Direct debugger method usage (`TryMovePlayerToRoom`, `TrySetVariable`, `TryHandleRoomPointIntent`, `LoadGameRuntimeProject`) is tracked in this table only when paired with `CurrentSession` reads.

| Line | Method | Usage Summary | Feature Area | First-Pass Classification |
|---|---|---|---|---|
| 299, 316-319 | `SimulatorViewModel` constructor | Command `CanExecute` gates require `CurrentSession` non-null | Command/playback controls | Fallback Debt |
| 302 | `SimulatorViewModel` constructor | Submit command gate requires `CurrentSession` | Command input | Fallback Debt |
| 347 | `SimulatorViewModel` constructor | Startup branch checks `CurrentSession` before rebuild/preview/polling | Startup hydration | Fallback Debt |
| 1188 | `TryLoadRuntimeProject` | Requires `CurrentSession` after local file-path load | Local runtime load | Debugger-Only Allowed |
| 2097 | `AppendCurrentRoomInitializationSummary` | Reads `CurrentRoom` from `CurrentSession` for startup console lines | Local runtime load summary | Debugger-Only Allowed |
| 2122 | `ApplyRenderSurfaceFromSession` | Reads authored render width/height from `CurrentSession` | Render surface sizing | Contract Gap |
| 2150, 2165, 2166 | `MovePlayerToRoomExecute` | Precondition and status messaging read from `CurrentSession` | Debug room move | Debugger-Only Allowed |
| 2175 | `TryHandleRenderSurfaceClick` | Waypoint mode precondition checks `CurrentSession` | Input gating | Fallback Debt |
| 2211 | `TryHandleRenderSurfaceClick` | Click-select precondition checks `CurrentSession` before debugger intent call | Debug room-object click select | Debugger-Only Allowed |
| 2266 | `TrySetGameStateVariable` | Mutation precondition checks `CurrentSession` | Debug variable mutation | Debugger-Only Allowed |
| 2434 | `SubmitWaypointsExecute` | Waypoint submit precondition checks `CurrentSession` | Waypoint command flow | Fallback Debt |
| 2471 | `SubmitWaypointsExecute` | Reads `CurrentRoom` from `CurrentSession` for waypoint context | Waypoint command flow | Contract Gap or Fallback Debt |
| 2616 | `ExecuteGameStateCommand` | Requires `_gameManager.HasActiveRuntime` and `CurrentSession` | Core command path | Fallback Debt |
| 2766, 2850 | `TryRequestImmediateSessionDeltaSynchronization`, `TrySynchronizeSessionDeltaStateFromPolling` | Polling sync requires `CurrentSession` | Session delta sync | Fallback Debt |
| 3982 | `EnsureNarrativePresentationFromCurrentSession` | Reads narrative tiers from `CurrentSession` phase nodes | Narrative presentation | Contract Gap or Fallback Debt |
| 4258, 4275 | `CanSaveRuntimeGameState`, `SaveRuntimeGameStateExecute` | Save-state flow requires `CurrentSession` | Session persistence UX | Fallback Debt |
| 4617, 4683, 4972 | `PlayGameStatePlaybackExecute`, `StepGameStatePlaybackExecute`, `ShiftToManualMode` | Playback controls require `CurrentSession` | Playback UX | Fallback Debt |
| 5350 | `RebuildGameStateTree` | Builds runtime-state tree from `CurrentSession` | Runtime state tab | Debugger-Only Allowed |
| 5384 | `RefreshCurrentRoomCommandOptions` | Reads command phrases from `CurrentRoom` on `CurrentSession` | Command suggestion UX | Contract Gap or Fallback Debt |
| 7146 | `RefreshCurrentRoomPreviewFromSession` | Full room/object preview rebuild from `CurrentSession` graph | Room preview rendering | Contract Gap or Fallback Debt |
| 7256 | `TriggerRoomEntryDiscoveryWarmupIfNeeded` | Reads `CurrentRoom.ScopeNodeId` from `CurrentSession` | Asset warmup scope | Fallback Debt |
| 7815 | `ResolveHostAssetGameKeyFallback` | Reads `ProjectName` from `CurrentSession` for asset identity fallback | Asset identity fallback | Fallback Debt |
| 8020 | `ResolveCurrentRoomObjectNode` | Resolves objects from `CurrentSession.CurrentRoom` | Object lookup fallback | Contract Gap or Fallback Debt |
| 8049 | `ResolveObjectDrawPlacement` | Reads current room for shared draw recompute fallback | Render placement parity | Contract Gap or Fallback Debt |
| 8308 | `BuildSharedPlacementParitySegment` | Reads current room grid sizing for trace parity computation | High-diagnostics render trace | Fallback Debt |

## First-Pass Classification Counts
Historical snapshot retained for lock-question traceability:
1. Total known `CurrentSession` call sites: 35.
2. Debugger-Only Allowed: 10 call sites.
3. Fallback Debt: 16 call sites.
4. Contract Gap: 1 call site.
5. Contract Gap or Fallback Debt (needs lock decision): 8 call sites.

## Current Inventory Snapshot (2026-08-25)
1. Total `CurrentSession` call sites in `Storyboard.Simulator/ViewModels/SimulatorViewModel.cs`: 1.
2. Approved direct usage:

| Line | Method | Usage Summary | Feature Area | Classification |
|---|---|---|---|---|
| 5350 | `RebuildGameStateTree` | Builds runtime-state tree from `CurrentSession` | Runtime state tab | Debugger-Only Allowed |

## Design Lock Questions
1. Should the strict boundary be "runtime-state tab only" for direct `CurrentSession`, or "runtime-state tab plus debugger action handlers"?
2. For thin mode v1, do we explicitly disable all debugger mutations (`TryMovePlayerToRoom`, `TrySetVariable`, `TryHandleRoomPointIntent`) even when command-level alternatives might exist?
3. Should render surface dimensions be added to `HostRuntimePresentationResult` and `HostSessionDataEnvelope`, or only to attach/baseline payloads with client-side caching?
4. Should current room display name and stable room id be first-class fields in thin-safe presentation payloads to eliminate room-label fallback reads?
5. Should narrative path display data (book/chapter/page names and optional title/prologue/narrative text) be emitted as host payload fields instead of inferred from `CurrentSession` graph fallback?
6. For room/object preview hydration, do we commit to host payload as authoritative and remove runtime-graph fallback in thin-safe flows?
7. Should game identity fallback from `CurrentSession.ProjectName` be removed and replaced with session/discovery identity only (`GameId`, `GameKey`, descriptor metadata)?
8. When thin-safe payload data is missing unexpectedly, should simulator fail closed (warning + skip UI update) instead of reading `CurrentSession` fallback?
9. Do we want a dedicated simulator-facing facade (for example runtime state reader service) as the only component allowed to read `CurrentSession`?
10. Should we add CI guardrails that fail when non-approved files/methods add new `.CurrentSession` reads?
11. Should capability gating live in a single runtime capability descriptor model shared by viewmodel commands and tabs, rather than ad hoc checks?
12. Are we locking additive-only contract changes for this effort, with no renames/removals until post-cutover cleanup?
13. Should this isolation effort complete before any `Storyboard.GameHost`/`Storyboard.GameClient` implementation starts, or allow overlap after Phase 1 capability gating?
14. Do we want a temporary compatibility mode switch that allows legacy fallback behavior for debugging while strict isolation is being rolled out?

## Question Count
1. Total design lock questions: 14.

## Locked Decisions (Q1-Q14)
1. Strict boundary: only Runtime State tab debugger workflows may use debugger surfaces and `CurrentSession`.
2. Thin mode v1 disables Runtime State tab and all debugger mutations.
3. Render surface dimensions are required in thin-safe payloads, emitted on attach/baseline and on delta only when changed (or full resync).
4. No duplicate top-level current-room identity fields are added now.
5. Attach and baseline must include `RoomChange.NewRoom` for current-room identity and render hydration.
6. Narrative path and tier content reuse existing phase DTOs (`HostCommandPhaseSummary` and `HostCommandPhaseNodeSummary`) with no new narrative DTO shape.
7. Attach and baseline must include `PhaseChange.NewPhase`; deltas include phase payload only when changed.
8. Non-debugger room/object preview hydration is host-payload authoritative; no runtime-graph fallback in thin-safe paths.
9. Non-debugger asset/game identity flow removes `CurrentSession.ProjectName` fallback and uses session/discovery identity precedence only.
10. Thin mode fails closed on missing thin-safe payload data (warning + skip affected update).
11. No new wrapper interface is required now; enforce boundary through composition ownership and allowlist guard tests using existing debugger interface.
12. Add CI guardrails that fail on new non-approved `CurrentSession` reads.
13. Use a single shared runtime capability descriptor for tab visibility and command gating.
14. Contract changes for this effort are additive-only, and contract delta packet agreement is required before implementation starts.

## Sequencing Lock
1. CurrentSession isolation and parity validation complete before any client/server implementation work begins.

## Temporary Compatibility Fallback Policy
1. Temporary fallback behavior is allowed only as a transition aid.
2. Fallback behavior is thick-mode-only, developer-only, and disabled by default.
3. Every fallback use must emit explicit diagnostics.
4. Hard gate: all temporary fallback logic must be fully removed before moving to client/server implementation.
