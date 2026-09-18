# Web Runtime Session Architecture Direction Guide

Status: Future Direction
Date: 2026-08-10

## Intent

Define a durable architecture direction for browser-hosted game runtime sessions that supports:

1. Authoritative server-side game state.
2. Long-lived bidirectional client connections.
3. Second-client join to an existing running session.
4. Client reconnect to an existing running session.
5. Future multiplayer evolution without replacing the core runtime model.

This guide is directional. It is not an implementation checklist.

## Non-Goals

1. No framework lock decision in this document.
2. No service-by-service delivery schedule.
3. No infrastructure sizing commitments.
4. No transport protocol optimization details.

## Core Direction

Use a two-plane architecture:

1. Stateless control plane:
- Identity, account/profile, project/runtime metadata, entitlement, matchmaking, and administration APIs.
- Conventional horizontally scaled web services and database patterns.

2. Stateful runtime plane:
- Authoritative in-memory game session execution.
- Timed and event-driven world progression.
- Server-to-client push for deltas/events.

Only the runtime plane is intentionally stateful.

## Session Identity And Authority Model

1. Every running session has a stable `SessionId`.
2. Session authority is singular: exactly one runtime owner may apply commands for a session at a time.
3. Client sockets are ephemeral transport handles, not ownership anchors.
4. Session state continuity is keyed by `SessionId` and authenticated player identity, not by specific connection id.

## Connection Routing Direction

The platform must support routing a new socket to the existing session authority.

Required logical capabilities:

1. Session directory lookup:
- Resolve `SessionId -> OwnerRuntimeNode`.

2. Attach flow:
- Validate player access to session.
- Bind new connection to session audience.
- Emit bootstrap snapshot and begin live stream.

3. Resume flow:
- Validate resume token and player identity.
- Replace stale connection binding for that player.
- Replay missed updates when possible, otherwise send full snapshot.

Implementation styles that satisfy direction:

1. Sticky affinity by session key.
2. Gateway forwarding to runtime owner.
3. Virtual actor routing (for example Orleans grain identity routing).

No single style is mandated yet.

## Runtime State Model

Use authoritative server state with command-based mutation.

1. Clients send intents/commands only.
2. Runtime validates command against authoritative state.
3. Runtime applies deterministic transition.
4. Runtime emits versioned updates to subscribed clients.

Recommended durability posture:

1. Hot state in memory for active sessions.
2. Periodic snapshots for recovery performance.
3. Append-only event log for replay/audit/recovery.

## Join And Reconnect Contract Direction

Define protocol contracts around the following message semantics:

1. `JoinSession`:
- Input: `SessionId`, player auth context.
- Output: accepted/rejected plus snapshot baseline and stream start cursor.

2. `ResumeSession`:
- Input: `SessionId`, player identity, resume credential, `LastAckSequence`.
- Output: replay window or full snapshot fallback.

3. `SubmitCommand`:
- Input includes idempotency key (`CommandId`).
- Server acks acceptance/rejection and applies once.

4. `StateDelta` and `ServerEvent`:
- Outbound updates carry monotonic `SequenceNumber` per session stream.

5. `ResyncRequired`:
- Explicit instruction for client to request fresh snapshot when gap cannot be replayed safely.

## Multiplayer Readiness Direction

Design single-player runtime so it is already multiplayer-safe.

1. Multiple authenticated connections can attach to one authoritative session.
2. Command ordering is serialized per session authority.
3. Fan-out publishes identical authoritative updates to all participants.
4. Access policy governs role-specific command rights.

This avoids architectural rewrite when multiplayer is activated.

## Time-Driven World Events

Treat time as a first-class simulation input.

1. Runtime supports scheduled actions and timed transitions independent of inbound requests.
2. Server emits resulting events/deltas proactively.
3. Timer execution remains within session authority boundary for consistency.

## Reliability And Failure Direction

1. Runtime ownership must use leases or equivalent guard to prevent split-brain session authority.
2. On owner failure, replacement runtime restores from snapshot + event tail.
3. Reconnecting clients receive deterministic resync behavior.
4. Session lifecycle states are explicit (`Creating`, `Active`, `Suspended`, `Recovering`, `Ended`).

## Security Direction

1. Every join/resume is authenticated and authorized.
2. Resume credentials are scoped, expiring, and revocable.
3. Server enforces command authorization per player role/context.
4. Runtime is authoritative; client-provided state is never trusted as truth.

## Observability Direction

Capture operational telemetry at minimum:

1. Session count and per-node ownership distribution.
2. Join/resume success and failure rates.
3. Reconnect latency and replay-vs-snapshot resync ratio.
4. Command rejection categories (auth, validation, ordering, stale client).
5. Session recovery outcomes after owner failover.

## Orleans Fit Assessment (Directional)

Orleans is a strong candidate for runtime-plane orchestration because:

1. Grain identity naturally maps to `SessionId` authority.
2. Location transparency reduces custom routing complexity.
3. Single-threaded grain execution simplifies per-session concurrency.
4. Built-in timers/reminders and persistence hooks align with session lifecycle needs.

Adoption remains a deliberate decision point, not a lock in this guide.

## Decision Gates Before Implementation Plan

Before converting this direction into an execution plan, lock decisions for:

1. Runtime hosting model (custom session hosts vs Orleans).
2. Transport stack (SignalR vs raw WebSocket protocol layer).
3. Persistence model (snapshot cadence, event retention, store technology).
4. Session directory and lease strategy.
5. Reconnect replay window and fallback policy.

## Alignment With Existing Repository Guardrails

1. Keep runtime/domain logic in Storyboard.Shared where host-agnostic.
2. Keep host-specific transport/composition details in host projects.
3. Preserve simulator/designer separation boundaries.
4. Treat exported runtime contracts as versioned external interfaces.

## Next Artifact Suggestion

When ready, produce a dedicated implementation plan that maps this direction to:

1. Proposed contract DTO additions.
2. Session manager abstractions.
3. Host composition boundaries.
4. Validation and regression gates.
