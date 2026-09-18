# Multi-Session Starter Plan (StartSession-First Design)

## Goal
Enable the runtime engine to run more than one game session at a time in a single host process, with `StartSession` as the primary lifecycle entry point and session authority.

## Why StartSession Must Be First-Class
Current runtime behavior is active-context based. To support multi-session safely, `StartSession` must become the canonical creator of a complete runtime session context, not just a registration record.

`StartSession` should own:
1. Session identity and metadata.
2. Runtime snapshot binding for that session.
3. Mutable game state root (`GameStateSession`) for that session.
4. Session-scoped command/correlation/clarification state.
5. Session-scoped attach/readiness state.

## Current Gaps (Summary)
1. Single-slot runtime fields (`_runtimeLoadedGame`, `_currentSession`, `_registeredSession`).
2. StartSession blocks second session (`Session.Start.AlreadyExists`).
3. Command execution APIs rely on one implicit active context.
4. Clarification/correlation state is global, not per session.
5. Command events/results do not carry session identity.

## Target Architecture (v1)
Introduce a per-session runtime container and treat all runtime state as session-owned.

### New Internal Model
1. `RuntimeSessionRecord`
- Session descriptor metadata (session id, game id/key, owner, policy, timestamps, state).
- `RuntimeLoadedGame` snapshot binding.
- `GameStateSession` mutable state.
- Session command state:
  - Completed correlation IDs set.
  - Pending clarification correlation ID.
  - Pending clarification raw command text.
  - Pending clarification payload.
- Attach/join state.

2. `Dictionary<Guid, RuntimeSessionRecord> _sessions`
- Replaces single `_registeredSession` and `_currentSession` ownership model.

3. Optional host-attachment cursor
- Keep a minimal host-local attached-session pointer for compatibility where APIs are still active-context based.

## Implementation Phases

## Phase 0: Guardrails and Preparation
1. Add focused tests that describe intended multi-session behavior:
- StartSession can create session A and session B.
- ListSessions returns both.
- Leave/Join/End target the requested session only.
- Ending one session does not mutate the other.
2. Preserve existing behavior through compatibility tests while refactoring internals.

## Phase 1: StartSession-First Storage Refactor
1. Add `RuntimeSessionRecord` internal type.
2. Replace single-session fields with `_sessions` map.
3. Update `StartSession`:
- Resolve runtime path for request game.
- Load runtime snapshot for that game.
- Create independent `GameStateSession`.
- Initialize session command state.
- Insert new record into map.
4. Remove `Session.Start.AlreadyExists` single-session gate.

Deliverable: multiple sessions can be created and listed.

## Phase 2: Session Lifecycle Operations
1. Update `GetSession`, `JoinSession`, `LeaveSession`, `EndSession` to operate by `SessionId` map lookup.
2. Keep ownership/auth rules per session record.
3. Ensure `EndSession` tears down only target session.

Deliverable: independent lifecycle actions per session.

## Phase 3: Thin-Client Runtime Routing (Compatibility First)
1. Keep existing thin-client method signatures initially.
2. Route command/presentation calls to an attached session context (host-local pointer set by `AttachSession`).
3. Make `AttachSession` select target record and bind active cursor without mutating unrelated sessions.

Deliverable: existing host APIs continue to work while engine supports multiple sessions.

## Phase 4: Contract Evolution (Recommended)
1. Add explicit `SessionId` to command/presentation request surfaces.
2. Add `SessionId` to command result/event payloads.
3. Keep compatibility shims temporarily:
- If `SessionId` missing, fall back to attached session.

### Phase 4A: Interface/DTO Surfaces That Must Carry SessionId
1. `HostProcessCommandRequest` should include `SessionId`.
2. `HostMoveByWaypointsRequest` should include `SessionId`.
3. `GetCurrentPresentation` should be session-addressable (parameter or request DTO carrying `SessionId`).
4. `HostProcessCommandResult` should include `SessionId` so consumers can disambiguate multi-session command streams.
5. `HostCommandProcessedEventArgs` should include `SessionId` (directly or via result payload).
6. Optional compatibility path:
- Treat `SessionId` as optional initially and route to attached-session cursor when omitted.
- Emit diagnostics warning when command execution occurs without explicit `SessionId`.

Deliverable: explicit, robust session-targeted command execution.

## Phase 5: Persistence and Diagnostics Hardening
1. Allow capture/load by explicit `SessionId` (or explicit active-attach semantics if deferred).
2. Ensure logging/diagnostics include session id on all command/lifecycle paths.
3. Add regression tests for cross-session isolation.

## Recommended First Slice (Smallest Useful Milestone)
1. Complete Phases 1 and 2 only.
2. Keep Phase 3 in compatibility mode.
3. Defer contract changes (Phase 4) until after internal stability.

This gets real multi-session lifecycle support quickly while limiting blast radius.

## Risks
1. Hidden coupling to single active session in non-lifecycle methods.
2. Clarification/correlation leaks if any global path remains.
3. Event consumers assuming one session stream.
4. UI host assumptions around one attached session.

## Risk Mitigations
1. Add explicit tests for session isolation before and during refactor.
2. Require all command-state fields to live in `RuntimeSessionRecord`.
3. Add session id to diagnostics immediately, even before full contract updates.
4. Keep feature behind staged rollout path in simulator host.

## Done Criteria (for multi-session v1)
1. StartSession can create at least two concurrent sessions for same or different games.
2. List/Get/Join/Leave/End all operate correctly per `SessionId`.
3. Commands for session A never mutate session B.
4. Ending session A does not end session B.
5. Existing playback regression and session lifecycle tests pass.
6. Command/presentation/event contracts are session-addressable (or documented compatibility fallback is active with warnings).

## Suggested Follow-Up Work Items
1. Create `RuntimeSessionRecord` and `_sessions` map.
2. Refactor `StartSession` to construct full session-owned context.
3. Refactor lifecycle methods by session map lookup.
4. Refactor command-state fields into session record.
5. Add session-id diagnostics and isolation tests.
