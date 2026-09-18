# Events Foundation Plan

Date: 2026-08-15
Status: Completed (Closed 2026-08-17)

## Status Refresh (2026-08-24)

1. Closure remains valid; no reopening blockers identified.
2. Focused phase/timer follow-on work has not invalidated event-foundation lock decisions.
3. This plan remains closed; only downstream integration scenarios continue under umbrella/phase plans.

## Current Snapshot (2026-08-17)

Overall assessment:

1. Event + session-delta foundation is complete for this plan scope.
2. Runtime command/event journaling and watermark progression paths are in place and repeatedly exercised by focused regression gates.
3. Host-facing command response compatibility is preserved while session-delta continuity remains active.

Validated this session:

1. Focused runtime regression gate passes:
- `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`
2. Smoke replay gate passes:
- `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`

Closeout summary:

1. Foundation hardening Sections A-F are complete with evidence captured below.
2. Focused runtime gate and smoke replay gate passed repeatedly in this closure pass.
3. Legacy presentation overlap decision is recorded as `narrow` with bounded usage rules.
4. Remaining non-blocking enhancements have been intentionally deferred to follow-on plans.

## Foundation Hardening Gate (Pre-Use-Case)

Decision lock for current priority:

1. Defer additional event-specific use cases beyond room/inventory foundation for now.
2. Do not start new event-specific features (including room-visit-count tracking) until this hardening gate is complete.
3. Defer direct gameSession anchor implementation until a concrete payload/filter use case requires it.

### A. Core Semantics Hardening

- [x] Complete anchor nesting + lifetime invariants:
	- room selection under `currentRoom.*`
	- command/action lifetime reset correctness
	- keep gameSession as reserved manifest root only (no implementation requirement in this closure pass)
- [x] Complete read-only anchor provider implementation.
- [x] Complete deterministic source-path resolver behavior:
	- missing-path diagnostics
	- `required` significance behavior
	- `defaultValue` fallback behavior
- [x] Wire event payload mapping through anchor provider/resolver only (single canonical path).

### B. Dispatch Determinism Hardening

- [x] Implement and verify subscription disposition behavior:
	- `bubble`
	- `consume`
- [x] Verify deterministic subscriber order across repeated runs.
- [x] Verify FIFO event chain behavior through quiescence.
- [x] Verify runaway-chain circuit breaker behavior and diagnostics.

### C. Delta Contract Integrity Hardening

- [x] Verify safe-boundary-only watermark advancement (`commandComplete`, `eventDispatchChainComplete`).
- [x] Verify no mid-boundary partial state emission.
- [x] Verify duplicate visibility tolerance (command response + later delta entries).
- [x] Verify expired watermark path returns resync-required behavior.
- [x] Verify baseline + incremental continuation for reconnect flows.

### D. Diagnostics and Operability Hardening

- [x] Add/verify medium/high diagnostics for anchor/path resolution outcomes.
- [x] Add/verify medium/high diagnostics for dispatch decisions (invoke, bubble, consume, stop reason).
- [x] Add/verify medium/high diagnostics for delta coalescing/pruning/watermark progression.
- [x] Keep low-noise mode clean (no verbose internals by default).

### E. Test Closure Hardening

- [x] Add unit tests for lifetime scopes and transition resets.
- [x] Add unit tests for resolver success/failure/default paths.
- [x] Add integration tests for MVP events and mapped payload/action invocation.
- [x] Repeat runtime-focused gate until stable green signal across repeated runs.
- [x] Repeat smoke replay gate until stable green signal across repeated runs.

### F. Overlap and Retirement Decision

- [x] Record final keep/narrow/retire decision for legacy `GetCurrentPresentation(...)` after delta baseline flows are validated.

## Push-Through Execution Order

1. Complete Section A (core semantics) first.
2. Then complete Section B (dispatch determinism).
3. Then complete Section C (delta contract integrity).
4. Then complete Section D (diagnostics) and Section E (test closure) in parallel where practical.
5. Finish with Section F overlap decision and mark plan status as complete-ready.

## Foundation Completion Definition

The event system foundation is considered complete for this plan when:

1. Sections A through F are checked complete.
2. Runtime-focused gate and smoke replay gate are passing reliably.
3. No unresolved lock questions remain for general event pipeline behavior.
4. New event-specific use case work can proceed (for example room-visit-count tracking) without reopening core foundation semantics.

### Completion Verdict (2026-08-17)

1. Criteria 1 met: Sections A-F are checked complete.
2. Criteria 2 met: runtime-focused gate and smoke replay gate passed with repeated green runs.
3. Criteria 3 met: lock answers captured in this plan.
4. Criteria 4 met: event-specific use case work may proceed under this locked foundation.

## Initial Hardening Audit (2026-08-17)

Observed implementation state (pre-closeout):

1. Resolver pipeline components exist in runtime code (`RuntimeObjectPathResolverPipeline`, start/projection/structural stages, object-id projection resolver, leaf resolver) and are already used by runtime event payload builder.
2. Event dispatch disposition behavior appears implemented in dispatcher code paths (`bubble` / `consume` handling present), but closeout tests and diagnostics assertions should still be treated as required before check-off.
3. Safe-boundary journal markers are present (`commandComplete`, `eventDispatchChainComplete`) and are being appended in manager delta journaling paths.
4. Inventory-history projection source paths and projection rules are present for event payload mappings.

Execution implication:

1. Remaining effort is likely weighted toward closeout verification, lifetime edge-case tests, and diagnostics hardening rather than brand-new subsystem creation.
2. Hardening sections A-F remain authoritative and should be closed in order before enabling new event-specific feature work.

Section A/E evidence update (2026-08-17):

1. Added `RuntimeEventPayloadBuilderTests` coverage for resolver semantics:
- `player_room_entered` applies `defaultValue` fallback (`isFirstVisit=false`) and omits unresolved optional `visitCount`.
- `inventory_item_added` reports required-missing diagnostic when `activePlayer.inventoryHistory.lastAddedObject.id` is unavailable.
- `inventory_item_added` succeeds with required `itemId` while omitting optional `itemNameInGame` when resolved object has blank `nameInGame`.
2. Focused validation gate passed:
- `dotnet test .\\Storyboard.GameEngine.Tests\\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~RuntimeEventPayloadBuilderTests|FullyQualifiedName~GameCommandContextLifecycleTests|FullyQualifiedName~RuntimeObjectPathResolverPipelineTests|FullyQualifiedName~RuntimeReferenceValueResolverPipelineTests"`

Section A closeout update (2026-08-17):

1. `ScopeChainReferenceValueResolver` now defaults to pipeline-backed `RuntimeObjectPathAnchorObjectResolver` (retiring default dependence on legacy projected-anchor resolver behavior for current-room/command/action alias expansion).
2. `RuntimeSessionAnchorValueProvider.Resolve(...)` now returns an immutable read-only dictionary instance.
3. Added/updated tests proving Section A closure behaviors:
- command/action alias visibility during command and absence after `EndCommandContext` (`GameCommandContextLifecycleTests.RuntimeSessionAnchorValueProvider_CommandAndActionAliases_AreAbsentAfterCommandContextEnds`).
- read-only provider mutation attempts throw (`RuntimeReferenceValueResolverPipelineTests.RuntimeSessionAnchorValueProvider_ReturnsReadOnlyDictionary`).
4. Focused Section A validation rerun passed after wiring changes:
- `dotnet test .\\Storyboard.GameEngine.Tests\\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~RuntimeEventPayloadBuilderTests|FullyQualifiedName~GameCommandContextLifecycleTests|FullyQualifiedName~RuntimeObjectPathResolverPipelineTests|FullyQualifiedName~RuntimeReferenceValueResolverPipelineTests"`
5. Full solution build passed:
- `dotnet build .\\StoryboardDesigner.slnx`

Section B closeout update (2026-08-17):

1. Dispatch disposition behavior is covered with explicit tests for `bubble` and `consume` in `RuntimeEventSubscriptionDispatcherTests`.
2. Added deterministic subscriber-order regression coverage across repeated dispatches:
- `RuntimeEventSubscriptionDispatcherTests.Dispatch_BubbleDisposition_RunsOriginThenAncestorSubscriptions_InDeterministicOrderAcrossRuns`
3. Added think-loop event-chain determinism coverage:
- FIFO chain processing through quiescence: `GameThinkLoopTests.ProcessWake_DrainsEventChainInFifoOrderThroughQuiescence`
- runaway-chain cap behavior: `GameThinkLoopTests.ProcessWake_EventChainExceedsWorkChainCap_StopsAndLeavesPendingWork`
4. Focused Section B validation gate passed:
- `dotnet test .\\Storyboard.GameEngine.Tests\\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~RuntimeEventSubscriptionDispatcherTests|FullyQualifiedName~GameThinkLoopTests"`

Section C decision + closeout update (2026-08-17):

1. Contract decision lock:
- Session-delta polling is the only authoritative synchronization path.
- Context host command APIs return state-thin acknowledgement envelopes (meta/diagnostic oriented), not authoritative session-state payloads.
- Command-boundary sequence snapshots may appear in diagnostics as non-authoritative observability text only and must not be consumed as sync control data.
2. Context command ack behavior implemented:
- `IHostRuntimeCommandProcessorClient.ProcessCommand(...)` and `ProcessMoveByWaypoints(...)` context paths now project immediate command `SessionData` to ack-only shape (room/object/output/sound state stripped; diagnostics retained).
3. Verification evidence (`SessionDeltaPollingContractBehaviorTests`):
- state-thin command ack with retained watermark continuity.
- non-authoritative boundary snapshot diagnostics (`CMD-BOUNDARY-SNAPSHOT`).
- immediate post-command poll returns authoritative state deltas.
- malformed watermark invalid-request behavior.
- expired watermark resync-required behavior.
- baseline + resume continuation without replay until next boundary.

Section D closeout update (2026-08-17):

1. Event payload diagnostics-level plumbing is now end-to-end:
- `RuntimeEventPublicationRequest.DiagnosticsLevel` flows through publication service, payload build request, publish request, and persisted `RuntimePublishedEvent.DiagnosticsLevel`.
2. Payload-resolution diagnostics are now verbosity-aware:
- medium/high emits resolver/default/optional-skip diagnostics while preserving required-missing failure semantics.
3. Dispatch diagnostics are now verbosity-aware:
- event-bound action execution request diagnostics level now follows `RuntimePublishedEvent.DiagnosticsLevel`.
- medium emits invoke/completion/chain-stop diagnostics; high retains additional detail paths.
4. Delta-poll operability diagnostics are now profile-gated without contract expansion:
- `HostSessionDeltaBatchProfile.Mild` remains low-noise (no verbose internals).
- `HostSessionDeltaBatchProfile.Medium` emits merged/no-update/resync diagnostics.
- `HostSessionDeltaBatchProfile.Hot` additionally emits retention trim window details.
5. Added/updated coverage:
- `RuntimeEventPayloadBuilderTests` medium diagnostics assertions.
- `RuntimeEventPublicationFlowTests` diagnostics-level propagation assertion.
- `RuntimeEventSubscriptionDispatcherTests` medium dispatch diagnostics assertions.
- `SessionDeltaPollingContractBehaviorTests` batch-profile diagnostics assertions.
6. Validation evidence:
- `dotnet build .\\StoryboardDesigner.slnx`
- `dotnet test .\\Storyboard.GameEngine.Tests\\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~SessionDeltaPollingContractBehaviorTests|FullyQualifiedName~RuntimeEventSubscriptionDispatcherTests|FullyQualifiedName~RuntimeEventPayloadBuilderTests|FullyQualifiedName~RuntimeEventPublicationFlowTests"`
- `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`
- `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`

Section E closeout update (2026-08-17):

1. Added lifetime/transition reset unit coverage:
- `RuntimeScopeLifetimeTransitionTests.MoveToRoom_RoomTransientVariables_DeactivateAndResetOnReentry`
- `RuntimeScopeLifetimeTransitionTests.MoveToRoom_CurrentRoomPrimarySelectionAnchor_DoesNotLeakAcrossRooms`
2. Added deterministic MVP integration coverage for mapped payload + dispatch invocation:
- `RuntimeEventMvpIntegrationTests.PublishAndDispatch_RoomEnteredAndExited_InvokesMappedActions`
- `RuntimeEventMvpIntegrationTests.PublishAndDispatch_InventoryAddedAndRemoved_InvokesMappedActions`
3. Stability evidence (repeated green runs):
- Focused runtime gate executed 2 additional consecutive runs after baseline; all green.
- Smoke replay gate executed 2 additional consecutive runs after baseline; all green.
4. Validation commands used for E closeout evidence:
- `dotnet test .\\Storyboard.GameEngine.Tests\\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~RuntimeScopeLifetimeTransitionTests|FullyQualifiedName~RuntimeEventMvpIntegrationTests"`
- `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`
- `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`

Section F decision update (2026-08-17):

1. Decision: `narrow` (not retire).
2. Keep `GetCurrentPresentation(...)` as a bounded snapshot API for:
- session attach/bootstrap initial hydration
- explicit/manual resync/recovery reads when hosts need full current snapshot semantics
3. Do not use `GetCurrentPresentation(...)` as command-result synchronization path.
4. Authoritative incremental synchronization remains `ISessionDeltaPolling.GetSessionBaseline(...)` + `GetSessionDeltas(...)` watermarks.
5. Call-site review evidence:
- `Storyboard.Simulator/ViewModels/SimulatorViewModel.cs` uses polling as the continuous path and calls `GetCurrentPresentation(...)` for bootstrap/manual presentation fetch paths.
6. Contract cleanup decision (2026-08-17): remove `scopeSearchMode` and `onAmbiguousAction` from event binding target contracts; runtime already resolves action targets using canonical nearest-scope behavior and does not require ambiguity mode controls.

## Goal

Deliver the first production-ready event foundation with deterministic dispatch, manifest-driven payload mapping, and lifetime-aware anchor resolution.

## Scope (this plan)

1. Event dispatch model and subscriber behavior.
2. Event payload mapping model and manifest semantics.
3. Anchor model and lifetime policy.
4. Event-to-action linkage and context exposure.
5. Event-focused implementation checklist and validation gates.

## Out Of Scope (this plan)

1. Tick scheduler implementation details.
2. Milestone authoring UX.
3. Full phase/schedule composition beyond event dependencies.

## Event Foundation (v1)

1. Add runtime event bus abstraction with deterministic in-process dispatch.
2. Event keys and emit points are engine-owned only.
3. Event handlers invoke actions through existing action execution pipeline.

## Event MVP Scope

1. `player.room.entered`
2. `player.room.exited`
3. `inventory.item.added`
4. `inventory.item.removed`

MVP lock refresh (2026-08-17):

1. This closure pass treats the four events above as the complete MVP event set to unblock Section E integration coverage.
2. Procedure events (`procedure_started`, `procedure_completed`, `procedure_failed`) remain explicitly deferred and out of this MVP closure slice.
3. Implementation status update:
- `player_room_entered`: implemented and published from `GameManager` room-transition command path.
- `player_room_exited`: implemented and published from `GameManager` room-transition command path using old-room context.
- `inventory_item_added`: implemented and published from `GameManager` command path when inventory-history transfer evidence is present.
- `inventory_item_removed`: implemented and published from `GameManager` command path when inventory-history transfer evidence is present.
4. Integration verification added for MVP events:
- `RuntimeEventMvpIntegrationTests` now validates publication + payload mapping + dispatch-loop action invocation for all four MVP event keys.

## Payload Mapping Model

1. Engine remains source of truth for event names and emit points.
2. Payload keys are configured in runtime manifest.
3. Mapping uses anchored runtime source paths.
4. Missing/invalid mappings are non-fatal and diagnostic.

### Mapping fields

1. `payloadKey`
2. `sourcePath`
3. `required`
4. `defaultValue` (optional)

### Missing-value policy

1. Missing mappings never crash runtime.
2. `required=true` raises diagnostic significance only.
3. `defaultValue` applies when provided.
4. Without default, unresolved mapping is omitted.

## Anchor Model (locked direction)

Top-level roots:

1. `currentPlanet`
2. `currentCountry`
3. `currentArea`
4. `currentRoom`
5. `activePlayer`
6. `currentCommand`
7. `currentAction`
8. `gameSession`

Scoping direction:

1. Room-selection anchors are nested under room context:
- `currentRoom.primaryRoomObject`
- `currentRoom.secondaryRoomObject`
2. Inventory-history anchors are active-player scoped.

## Anchor Lifetime Policy

1. All anchor access routes through `GameStateSession` entry point.
2. Anchor values are lifetime-scoped, not universally durable.
3. Room-scoped anchors reset on room transition.
4. Command-scoped anchors initialize at command start and clear at command end.
5. Action-scoped anchors track the currently executing action frame (including linked actions).
6. Active-player-scoped inventory history swaps projection when active player changes.

## Event Dispatch Behavior

1. Dispatch is serialized and deterministic.
2. Event emissions are queued and processed FIFO.
3. Runtime processes queued work to quiescence before returning.
4. Event-triggered actions may emit additional events through same queue.
5. Per-turn processing cap acts as circuit breaker for runaway chains.

## Host Delta Delivery Model (locked direction)

1. Host-facing contracts should communicate session state changes, not event internals.
2. Delta payload shape should reuse existing concise host-facing change structures where practical (for example room change and room object change payload families already used by command results).
3. Delivery transport is separate from payload shape:
- Push model for callback/event-capable hosts.
- Pull model for polling-capable hosts.
4. Runtime should support both models from one canonical session-delta stream.
5. Command response payloads include an additive session-delta watermark token so healthy clients can keep watermark continuity in sync during normal command flows (including room changes).
6. Client consumers must tolerate duplicate visibility between immediate command responses and subsequent delta-stream entries.
7. Session-delta stream is runtime-owned and source-agnostic: command, event, scheduler/tick, and future mutation sources all flow through the same channel.
8. Runtime should buffer per-session deltas to support pull consumers and reconnect/recovery for push consumers.
9. Session-delta delivery is current-state oriented, not a per-mutation change log.
10. If the same logical value is updated multiple times before delivery, runtime should coalesce and ship only the latest effective value.
11. Presentation cues carry author-defined delivery/freshness policy metadata.
12. Runtime evaluates cue policy server-side at payload-build time (against session timing/watermark context) and may drop stale/non-applicable cues before delivery.
13. Clients do not make stale/drop policy decisions; clients render only cues included in the delivered payload.
14. Initial authored cue freshness fields are `lateDeliveryPolicy` and optional `maxLateMs`.
15. Runtime applies these fields when constructing outbound session-delta payloads.
16. Session-delta journal entries for game-property updates record changed logical property identity/path, not property value snapshots.
17. Delta payload values for game properties are resolved from authoritative live session scope state at payload-build time.
18. Runtime must not source game-property values from journal snapshots, preventing stale-value replay from journal storage.
19. Polling batch intent requests (for example advisory `Mild`/`Medium`/`Hot` profiles) are hints interpreted by runtime, not arbitrary hard slice points.
20. Runtime truncates outbound batches only at safe execution boundaries; never emit mid-execution partial state.
21. Safe truncation boundaries include completed command execution and fully completed event dispatch chains (including bubbling/propagation completion).
22. Runtime mutation processing remains single-threaded and serialized so command/event journal ordering is deterministic and non-interleaved at execution boundaries.
23. Journal entries carry room identity captured at boundary write time.
24. Room relevance pruning occurs before profile-driven volume decisions: entries outside the current room are not eligible for outbound delta consideration.
25. Room transition is a journal-scope boundary: prior-room entries are purgeable after transition capture.
26. If requested watermark predates retained room-scoped journal, runtime returns resync-required and caller must hydrate via baseline for current room.

### Subscription disposition

1. Subscription-level dispatch disposition is explicit:
- `bubble`
- `consume`
2. This controls whether dispatch continues to later subscribers.
3. This is separate from per-binding `stopChainOnFailure` behavior.

## Event-to-Action Linkage

1. Default linkage is action-targeted invocation.
2. Reuse existing scope-resolution mechanics where practical.
3. Command-template invocation remains future extension, not v1 default.

## Action Context Extension

1. Expose trigger metadata in unified action execution context:
- `action.trigger.kind`
- `action.trigger.isEventFired`
- `action.trigger.eventKey`
- `action.trigger.eventSequenceNumber`
2. Project event payload into `action.trigger.gameProperty.*`.
3. Keep behavior deterministic and additive-only.

## Reusable Predicate Direction

1. Implement one reusable predicate concept for event binding filters now.
2. Wire event condition/filter contracts to shared predicate contracts.
3. Defer lock/recipe migration to later milestone.

## Pre-Implementation Lock-Off Questions

- [x] Q1. Watermark cursor semantics: is watermark exclusive (`after watermark`) for all delta reads, and what is behavior for malformed watermark tokens?
- [x] Q2. Expired watermark behavior: when requested watermark falls outside retention, should runtime return a resync-required outcome with baseline payload + fresh watermark?
- [x] Q3. Safe boundary marker set: confirm the initial journal boundary markers are `commandComplete` and `eventDispatchChainComplete` only.
- [x] Q4. Batch profile interpretation: what concrete runtime limits map from `Mild`, `Medium`, and `Hot` (count, bytes, processing budget, or hybrid)?
- [x] Q5. Cue freshness defaults: when `lateDeliveryPolicy` is omitted, should default be `PlayIfStateApplied`, and when `maxLateMs` is omitted should no explicit time cutoff apply?
- [x] Q6. Cue staleness clock basis: should freshness be evaluated against deterministic runtime ticks rather than wall-clock time?
- [x] Q7. Journal retention policy: what minimum retained execution boundaries per session are required before compaction/trimming?
- [x] Q8. Host diagnostics contract: should dropped cue diagnostics be included in payload diagnostics by default or only at elevated diagnostics levels?
- [x] Q9. Engine loop pacing: how is loop wake cadence controlled, and which pacing knobs are externally configurable?

### Lock Answers

1. Q1 locked: watermark is exclusive. `GetSessionDeltas(watermark)` returns deltas strictly after the supplied watermark.
2. Q1 locked: null/empty watermark is valid and requests initial sync from the earliest retained safe boundary.
3. Q1 locked: malformed watermark tokens return invalid-request behavior; runtime does not silently reinterpret malformed tokens.
4. Q2 locked: `GetSessionDeltas(...)` remains delta-only and does not return level-set baseline payloads.
5. Q2 locked: when watermark is outside retention, runtime returns resync-required behavior and client must call a separate baseline method on the session-delta polling seam.
6. Q2 locked: baseline method scope is full current presentation hydration for the active room view, including all host-facing data required to reestablish room presentation from scratch.
7. Q2 locked: baseline response includes a fresh session-delta watermark so incremental polling can resume immediately after baseline apply.
8. Q3 locked: initial safe boundary marker set is exactly `commandComplete` and `eventDispatchChainComplete`.
9. Q3 locked: runtime truncation and emission boundaries must align to these markers; no mid-boundary partial emission is allowed.
10. Q4 locked: profile handling is evaluated only after room-relevance pruning; entries from prior rooms are excluded before any profile-based pacing decision.
11. Q4 locked: under consolidated single-delta polling, `Mild`, `Medium`, and `Hot` do not change outbound payload shape (still zero-or-one consolidated delta); profile semantics are advisory pacing controls only.
12. Q4 locked: if future pacing limits are applied, they advance watermark only through validated room-relevant safe boundaries and must not reintroduce partial/mid-boundary snapshots.
13. Q5 locked: near-term cue freshness enforcement is lateness-window based; cues are not played after their allowed lateness window expires.
14. Q5 locked: boundary-safe journal batching is expected to keep related activity packaged together, reducing the need for strict cue-to-state correlation in v1.
15. Q5 locked: until explicit cue/state correlation metadata exists, `PlayIfStateApplied` is treated as `PlayIfLate` and still subject to lateness-window expiry.
16. Q6 locked: producer-facing freshness configuration uses natural time units (milliseconds), not tick counts.
17. Q6 locked: runtime evaluates freshness against deterministic internal session time, with session time advanced by configured tick progression.
18. Q6 locked: producer-authored millisecond windows are converted/evaluated in runtime session-time math without exposing tick semantics to content authors.
19. Q7 locked: journal retention floor settings are externally configurable via a new general runtime engine tuning file rather than hard-coded constants.
20. Q7 locked: initial tuning settings are `sessionDeltaMinRetainedSafeBoundaryCount` and `sessionDeltaMinRetainedWindowMs`.
21. Q7 locked: defaults are provisional and can be tuned per game/host environment without code change.
22. Q7 locked: room-transition pruning is primary relevance compaction; count/time retention floors are secondary within retained current-room scope.
23. Q8 locked: dropped-cue diagnostics are emitted in host payload diagnostics only at elevated diagnostics levels (for example `Medium` and above), not by default low-noise mode.
24. Q9 locked: engine loop uses fixed-step timer pacing with immediate signal wake for queued command/event work; commands are queued and never executed inline on caller threads.
25. Q9 locked: external pacing controls are `engineLoopTickIntervalMs` and `engineLoopMaxCatchUpTicksPerWake` in runtime engine tuning config.
26. Q9 locked: loop remains single-threaded/serialized for state mutation ordering, with catch-up processing capped per wake to prevent runaway latency bursts.

## Execution Checklist (Historical Working List - Superseded)

Closure note:

1. This checklist reflects earlier implementation tracking and is retained as historical context.
2. For closure authority, rely on hardening Sections A-F and the completion verdict above.

- [ ] Confirm contract/manifest lock snapshot is current (anchors, path semantics, dispatch disposition, and no `valueHint`).
- [ ] Add `GameStateSession` anchor state for room selection (`PrimaryRoomObject`, `SecondaryRoomObject`) and room-transition reset behavior.
- [x] Introduce command-scoped runtime context (`currentCommand`) and initialize/clear it across command lifecycle.
- [x] Introduce action-scoped runtime context (`currentAction`) and update it for each action execution frame (including linked actions).
- [ ] Keep room selection anchors under room context (`currentRoom.primaryRoomObject`, `currentRoom.secondaryRoomObject`) rather than top-level duplicates.
- [ ] Implement active-player-scoped inventory-history anchor behavior (project through active player; avoid cross-player bleed).
- [ ] Implement read-only anchor provider over session + command/action context.
- [ ] Implement deterministic source-path resolver with soft diagnostics and optional `defaultValue` fallback.
- [ ] Wire event payload mapping resolution through the new anchor provider/resolver.
- [ ] Implement subscriber dispatch disposition behavior (`bubble` / `consume`) at subscription level.
- [ ] Lock host session-delta contract seam with dual delivery interfaces (push + pull) using shared payload structures.
- [x] Implement per-session delta buffering and cursor/sequence progression for push/pull parity.
- [x] Preserve current command response contract while enabling duplicate visibility through session-delta delivery.
- [ ] Add startup/load-time manifest validation diagnostics for anchor keys and paths.
- [ ] Add unit tests for lifetime scope behavior (room, command, action, active-player projections).
- [ ] Add unit tests for path resolution, missing-value diagnostics, and default fallback behavior.
- [ ] Add integration tests for MVP events using manifest mappings.
- [x] Run focused regression suite: `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`.
- [x] Run smoke replay regression gate: `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`.
- [ ] Validate in simulator host flows and capture follow-up lock decisions.

## Phase-Ordered Implementation Checklist (Historical - Superseded)

Closure note:

1. This phase list is preserved for traceability and is no longer the closure authority for this plan.
2. Closure authority is the hardening gate completion evidence and recorded lock decisions.

- [ ] Phase 1 - Designer authoring: complete event/subscription/payload/anchor authoring surfaces and designer-time validation.
Done criteria: at least one full event-definition set can be authored and exported as runtime-ready test fixture data.

- [x] Phase 2 - Shared contract stabilization: finalize and lock event/payload/delta contract terms, including watermark naming.
Done criteria: schema and generated contract outputs are locked with no open naming or shape decisions.

- [x] Phase 3 - Runtime inventory-history prerequisite: implement active-player inventory-history data modeling and persistence before event runtime mapping/dispatch work.
Done criteria: inventory add/remove mutations update active-player inventory-history (`lastAddedObject`, `lastRemovedObject`) with object references; save/load round-trip preserves inventory-history state; runtime can resolve inventory-history source paths used by `inventory.item.added` and `inventory.item.removed` payload mappings.

Review note for this phase:

1. Before moving to Phase 4, capture and inspect a representative save-game/session-state payload and confirm exactly how active-player inventory-history is represented at rest (including both `lastAddedObject` and `lastRemovedObject`).
2. Record whether persisted shape is variable-backed, structured object-backed, or hybrid, and verify it round-trips without lossy transformations.
3. If persisted representation is unclear or brittle for anchor resolution, stop and lock a clarification note/decision before proceeding to runtime anchor/resolver implementation.

Phase 3 review outcome (2026-08-16):

1. Representation at rest is variable-backed on object scope state (`SaveScopeNodeStateDto.VariableStates`) for participating container objects.
2. Persisted keys verified in save envelope include:
- `inventoryHistory.lastAddedObject.id`
- `inventoryHistory.lastAddedObject.nameInGame`
- `inventoryHistory.lastRemovedObject.id`
- `inventoryHistory.lastRemovedObject.nameInGame`
3. Save/load round-trip preserves these values without lossy transformation in focused regression coverage.

### Locked Resolver Pipeline Plan (Approval-Gated)

Scope lock for current runtime resolver effort:

1. Implement the new object-path pipeline using only the components listed below.
2. Do not add, remove, merge, or materially repurpose these components without explicit producer approval.
3. Any proposed deviation must be recorded in this plan first as "Approval Required" and left unimplemented until approved.

Locked implementation set:

1. `RuntimeObjectPathResolverPipeline`
- Interface: `IRuntimeObjectPathResolver`
- Responsibility: coordinator/orchestrator only.

2. `RuntimeObjectPathStartContextStage`
- Interface: `IRuntimeObjectPathResolutionStage`
- Responsibility: choose starting node context (anchor-rooted via `::` when present; otherwise caller-provided current node context).

3. `RuntimeObjectPathProjectionStage`
- Interface: `IRuntimeObjectPathResolutionStage`
- Responsibility: evaluate projection resolvers at current cursor before structural traversal.

4. `RuntimeObjectPathStructuralTraversalStage`
- Interface: `IRuntimeObjectPathResolutionStage`
- Responsibility: resolve real hierarchy hops when no projection applies.

5. `ObjectIdProjectionResolver`
- Interface: `IRuntimeObjectProjectionResolver`
- Responsibility: single generalized projection-by-object-id resolver.
- Required rule parameters: projection token/path pattern, object-id source variable name, required source node kind.

6. `RuntimeObjectNodeLeafVariableResolver`
- Interface: `IRuntimeObjectPathLeafValueResolver`
- Responsibility: resolve final leaf variable on the resolved object node.

Locked initial projection rules for `ObjectIdProjectionResolver`:

1. `primaryRoomObject` -> id source `activeRoomObjectId`, source node kind `Room`.
2. `secondaryRoomObject` -> id source `secondaryActiveRoomObjectId`, source node kind `Room`.
3. `inventoryHistory.lastAddedObject` -> id source `inventoryHistory.lastAddedObject.id`, source node kind `GameObject`.
4. `inventoryHistory.lastRemovedObject` -> id source `inventoryHistory.lastRemovedObject.id`, source node kind `GameObject`.

Explicit non-goals for this locked slice:

1. No broad replacement of existing legacy reference-value pipelines.
2. No runtime wiring expansion beyond the isolated target entry path for this effort.
3. No additional projection implementation classes unless explicitly approved.

- [ ] Phase 4 - Runtime anchor and resolver core: implement anchor lifetime behavior and deterministic source-path resolution with diagnostics/default fallback.
Done criteria: unit tests verify lifetime transitions/reset behavior and path-resolution outcomes.

- [ ] Phase 5 - Runtime event dispatch: implement FIFO dispatch, subscription disposition handling, and event-to-action invocation.
Done criteria: integration tests verify dispatch ordering, bubble/consume behavior, and action invocation chaining.

- [x] Phase 6 - Canonical session-delta stream: implement per-session buffering and watermark progression while preserving existing command response behavior.
Done criteria: polling with and without watermark behaves as expected for initial sync and incremental sync.

- [ ] Phase 7 - Host/simulator delivery wiring: wire polling/subscription consumption and watermark persistence/reuse across reconnects.
Done criteria: simulator host validates end-to-end delta consumption, watermark advancement, and recovery behavior.

- [ ] Phase 8 - Regression and overlap review: run focused gates and perform post-implementation overlap review for GetCurrentPresentation.
Done criteria: regression gates pass and keep/narrow/retire decision is recorded for legacy GetCurrentPresentation usage.

## Implementation Order (Designer -> Runtime -> Simulator)

1. Designer first: implement authoring/validation and produce stable event/cue fixtures.
2. Contract sync: finalize any last shared contract seams required by authored output and runtime consumption.
3. Runtime prerequisite: implement active-player inventory-history mutation tracking and save/load persistence; update runtime inventory event payload paths to inventory-history sources.
4. Runtime core: implement loop pacing, journal boundaries, watermark progression, and baseline/delta plumbing.
5. Runtime policy pass: enforce coalescing and cue freshness/drop rules server-side.
6. Simulator host wiring: implement baseline then incremental polling with watermark persistence/resync flow.
7. End-to-end verification: run designer fixtures through runtime + simulator and validate deterministic behavior.
8. Overlap decision: re-evaluate legacy `GetCurrentPresentation(...)` role after baseline endpoint behavior is proven.

## Designer POV Status (2026-08-16)

### Delivered in Designer

1. Event Subscription editor simplification landed: removed `scopeSearchMode` and `onAmbiguousAction` from visible UX while preserving contract compatibility.
2. Filter operator selection now uses explicit enum-backed options.
3. Event filter variable chooser is split into two explicit modes:
- Known event payload variables (left side).
- Anchor/subproperty/template-variable guidance (right side).
4. Right-side guidance flow is staged as explicit steps:
- select template anchor
- select subproperty
- select scope object (quick pick or search)
- select variable
5. Scope object search dialog exists and is reusable (not event-subscription-specific).
6. Quick pick is intentionally constrained to likely local/up-scope targets (`Self`, `Parent`, `Ancestor`).
7. Traversal-leg filter remains visible as disabled reminder with deferred follow-up.

### Remaining from Designer Perspective

1. Add explicit micro-copy for step guidance in the right-side pane (step labels or short helper text near each control).
2. Add small source-path visibility for selected subproperty (for example show `currentCommand.primaryCommandObject` read-only under subproperty selection).
3. Add a compact empty-state message in variable grid area when step prerequisites are not yet met.
4. Add regression tests for staged right-side flow:
- variables remain hidden/disabled until scope object is selected
- quick pick stays constrained to allowed relations
- search dialog relation toggles + refresh behavior are honored.
5. Decide whether scope-object text filters should be staged like checkbox filters (currently text filters apply immediately).

### Validation Rules to Add (Designer)

Current `PROJ-015 Event Subscription Integrity` covers only structural basics (`eventKey`, duplicate binding order, required `actionName`, required `variableName`).

Recommended additional rules:

1. `EVT-001` Event key known in manifest (Warning)
- Flag subscriptions whose `eventKey` is not present in `event-payload.manifest.json`.

2. `EVT-002` Filter variable matches known payload key or authored variable pattern (Warning)
- For event payload mode, prefer payload keys that exist for the selected event.
- For template mode, enforce `<subPropertyKey>.<variableName>` shape.

3. `EVT-003` Manifest anchor/subproperty reference validity (Warning)
- When using template mode, ensure selected anchor/subproperty pair exists in manifest `anchors[].supportedSubProperties`.

4. `EVT-004` expectedScopeType compatibility diagnostics (Warning)
- If manifest subproperty declares `expectedScopeType`, validate chosen scope object type compatibility.

5. `EVT-005` Duplicate filter condition normalization (Info/Warning)
- Detect repeated filter rows within the same binding (same `variableName`, operator, expected value).

6. `EVT-006` Event payload key collisions with template-mode paths (Info)
- Surface ambiguous naming when a payload key and template-mode reference can be confused in the same binding.

Rule placement suggestion:

1. Keep structural integrity in `PROJ-015`.
2. Add event-authoring semantic rules as focused rules in `Validation/Rules/Project` (one rule per concern) to keep deterministic diagnostics and simpler test coverage.

## Validation Gates

1. `dotnet build .\StoryboardDesigner.slnx`
2. Runtime-focused regression suite.
3. Smoke replay regression gate.

## Exit Criteria

1. Deterministic event dispatch behavior is implemented and tested.
2. Anchor lifetime model is implemented and tested.
3. Manifest path resolution is implemented with diagnostics/default behavior.
4. MVP events produce expected payloads and invoke actions correctly.
5. Subscriber disposition (`bubble`/`consume`) behavior is verified.

## Post-Implementation Follow-Up

1. Re-evaluate overlap between legacy presentation hydration and session-delta delivery after implementation lands.
2. Specifically review whether `IHostRuntimeCommandProcessorClient.GetCurrentPresentation(HostRequestContext context, GameDiagnosticsLevel diagnosticsLevel = GameDiagnosticsLevel.None)` remains necessary as-is, should be narrowed, or should be retired in favor of the session-delta model.
3. Completed (2026-08-17): enforce action catalog invariant that duplicate action names within the same scope are rejected via designer validation rule `PROJ-016 Scoped Action Name Uniqueness`.
4. Completed (2026-08-17): retain `quantityEvaluationMode` support for event binding conditions, but remove support for `SumOfResolvedCanPass` in event filtering; event-condition choices are constrained to supported deterministic modes.
5. Revisit whether an optional event history/journal structure is needed for playback and replay diagnostics, and if adopted, lock retention/shape boundaries so playback behavior stays deterministic.
6. Continue deterministic scheduler/replay lock decisions in the umbrella time/events plan, including explicit replay assertion mode policy (`strict` vs `semantic`) and CI/default usage guidance.
