# GameHost and GameClient Client-Server Architecture Plan

## Goal
Introduce remote runtime hosting with HTTP JSON while preserving simulator usability through a dual-mode composition model.

## Dependencies
This plan assumes the CurrentSession isolation plan is executed first:
1. `plans/archived/SIMULATOR_CURRENTSESSION_ISOLATION_PLAN.md`

## Target Projects
1. `Storyboard.GameHost`
- Server process hosting game runtime and host contract endpoints.

2. `Storyboard.GameClient`
- C# client adapters implementing host interfaces via HTTP JSON.

## Design Principles
1. Keep `Storyboard.GameEngine` system-neutral and reusable.
2. Keep simulator host-specific and mode-selectable.
3. Preserve additive contract evolution in shared host interfaces.
4. Avoid hidden fallback semantics in transport mapping.
5. Minimize maintenance by generating transport glue where possible.

## Runtime Modes
1. Thick Local Mode
- Simulator composes in-process runtime via existing factory.

2. Thin Remote Mode
- Simulator composes `Storyboard.GameClient` implementations.
- Runtime-state tab and debugger-only actions disabled initially.

## API Surface (Initial Thin-Safe Set)
1. `IHostRuntimeCommandProcessorClient`
2. `IHostSessionManagementClient`
3. `IHostSessionPersistenceClient`
4. `ISessionDeltaPolling`
5. `IHostGameDiscoveryClient`
6. `IHostAssetManagementClient`
7. `IHostIdentityManagementClient`

Excluded from thin v1:
1. `IHostRuntimeGameDebugger`
- No direct file-path load support in thin mode.
- No remote `CurrentSession` object graph transfer in thin mode v1.

## Transport Approach
1. ASP.NET Core Minimal API in `Storyboard.GameHost`.
2. System.Text.Json with shared serializer options for request/response parity.
3. Explicit endpoint per interface method and DTO-first contracts.
4. Support for cancellation tokens and request correlation IDs.

## Endpoint Conventions
1. Route prefix by interface area:
- `/api/runtime/*`
- `/api/session/*`
- `/api/discovery/*`
- `/api/assets/*`
- `/api/identity/*`
2. POST for command/state-changing operations.
3. GET or POST for query operations based on payload complexity.
4. Structured error envelope aligned with existing host result models.

## Code Generation and Maintenance Strategy
1. Add development-time transport generator utility.
2. Source of truth:
- Host interface signatures in shared contracts.
- Optional per-method metadata for route name and HTTP verb.
3. Generate:
- Server endpoint mapping stubs.
- Client proxy method bodies.
- Compile-time mismatch diagnostics if interface changed without regen.

Fallback if generator is deferred:
1. Implement manual vertical slices first.
2. Add test checks that guard endpoint-client parity.

## Phases

## Status Snapshot (2026-08-26)
1. Phase 0 (Foundation and Contracts): Completed.
- Thin-safe scope and contract-lock decisions are captured in Design Lock decisions 1-16.

2. Phase 1 (GameHost Skeleton): Completed.
- Host project exists, composes runtime host clients, and exposes health/meta plus full thin-safe endpoint groups.

3. Phase 2 (GameClient Skeleton): Completed.
- GameClient HTTP adapters exist for runtime, session/delta/persistence, and identity/discovery/assets seams.

4. Phase 3 (Simulator Dual-Mode Composition): Completed.
- Simulator supports thick and thin startup modes, explicit runtime connection settings, and thin-unavailable debugger behavior.

5. Phase 4 (Expand Endpoint Coverage): Completed for thin-safe surface.
- Endpoint and client coverage for all listed thin-safe interfaces is in place.
- Transport parity verification and guardrail tests are wired into CI.

6. Phase 5 (Hardening and Packaging): In progress.
- Cross-platform host publish matrix workflow exists.
- Structured request logging and correlation are in place.
- Transport trace controls with profile/field filtering are implemented.

## Phase 0: Foundation and Contracts
1. Lock thin-safe contract scope.
2. Confirm additive fields from CurrentSession isolation prerequisites.
3. Lock host capability source and mode-aware UX behavior per accepted decisions.

Schema/contract gate for Phase 0:
1. Perform explicit schema-impact review before implementation begins.
2. If schema changes are needed, keep them additive, approve first, and land them in an isolated contract-first change.

Deliverable:
1. Stable thin-safe contract baseline.

## Phase 1: Storyboard.GameHost Skeleton
1. Create project and composition root.
2. Compose runtime using existing runtime host factory.
3. Add health endpoint.
4. Add one vertical slice endpoint:
- Attach session + process command + get baseline.

Deliverable:
1. Runnable server with one end-to-end gameplay flow.

## Phase 2: Storyboard.GameClient Skeleton
1. Create project with typed HTTP client infrastructure.
2. Implement the same vertical slice interface methods.
3. Add retry/timeouts policy for safe idempotent reads only.
4. Preserve deterministic error translation to host result envelopes.

Deliverable:
1. In-process simulator can call remote runtime for vertical slice.

## Phase 3: Simulator Dual-Mode Composition
1. Add simulator runtime mode settings and startup selection.
2. Thick mode:
- existing in-process composition.
3. Thin mode:
- compose `Storyboard.GameClient` interfaces.
- disable runtime-state tab and debugger-only commands.
4. Add status/UX indicators of active mode.

Deliverable:
1. Simulator launches and operates in both thick and thin modes.

## Phase 4: Expand Endpoint Coverage
1. Implement remaining thin-safe interface endpoints and client methods.
2. Add session delta polling parity tests.
3. Add identity/discovery/asset integration tests.

Deliverable:
1. Thin mode reaches practical feature parity excluding debugger-only features.

## Phase 5: Hardening and Packaging
1. Cross-platform publish matrix for `Storyboard.GameHost`:
- Framework-dependent publish.
- Self-contained publish for selected runtimes.
2. Add structured logs and request correlation.
3. Add auth boundary hardening if needed for non-local deployments.

Deliverable:
1. Portable host runtime package and validated deployment guidance.

## Testing Strategy
1. Unit tests:
- Endpoint handler behavior and result mapping.
- Client proxy serialization/deserialization and error mapping.
2. Integration tests:
- Host in-memory test server plus real `Storyboard.GameClient` calls.
3. Simulator mode tests:
- Thin-mode capability gating and command flow.
- Thick-mode regression parity.

## Risks
1. Contract drift between interfaces and endpoint/client implementations.
2. Hidden thick-mode assumptions in simulator viewmodel.
3. Session consistency under concurrent clients.
4. Transport overhead for payload-heavy updates.

## Mitigations
1. Generator or parity tests as hard gate in CI.
2. Complete CurrentSession isolation prerequisites first.
3. Session-id-first routing discipline in all thin-safe requests.
4. Prefer delta/baseline payloads over heavy graph serialization.

## Done Criteria
1. `Storyboard.GameHost` serves all thin-safe host interface operations over HTTP JSON.
2. `Storyboard.GameClient` implements thin-safe host interfaces and is simulator-compatible.
3. Simulator supports both modes with minimal composition-only switch impact.
4. Debugger and runtime-state tab are intentionally unavailable in thin mode v1.
5. Build and targeted regression tests pass.

## Deferred Follow-Up
1. Separate design for remote debugger semantics and `CurrentSession` replacement/projection model.
2. Optional push-based session updates (`ISessionDeltaSubscriber`) once pull flow is stable.

## Design Lock Questions
1. Capability Source of Truth
- Should host capability support remain interface-driven through `SupportsCapability(...)`, or move to a dedicated capability endpoint/DTO as the authoritative source?

2. Thin-Mode Bootstrap Contract
- Which minimum startup payload fields are mandatory in thin mode (render dimensions, game label, narrative path, active room label), and which are optional with warnings?

3. Session Attachment Ownership
- Is session attach in thin mode always explicit (user/session picker), or can startup options auto-attach to a last-known session?

4. Correlation and Idempotency Rules
- What is the required correlation/idempotency policy per operation type, and which requests must be retriable vs single-attempt only?

5. Error Envelope Standardization
- Should all HTTP failures be normalized into host result envelopes, including transport failures/timeouts, or should some failures bubble as client exceptions?

6. Endpoint Versioning Strategy
- Do we version by URL segment, header, or contract assembly version alignment only, and what is the non-breaking change policy for thin-safe endpoints?

7. Authentication Boundary for v1
- Is thin-mode v1 limited to local trusted development, or does v1 require authenticated caller context and role checks for session/asset operations?

8. Authorization Scope Model
- Are access checks performed solely on session ownership/membership, or must game-level discovery and asset scopes enforce additional authorization predicates?

9. Session Consistency Model
- Under concurrent clients, what consistency guarantees are required for baseline plus delta polling (strict ordering, monotonic sequence, eventual consistency tolerances)?

10. Polling Cadence and Backpressure
- What default polling interval/window should thin client use, and what server-side throttling/backpressure behavior should be contractually defined?

11. Code Generation Contract
- Is transport generation required as a merge gate from phase 1 onward, or can manual endpoint/client slices continue until a later hard switch?

12. Thin-Mode UX Contract
- Beyond disabling runtime-state/debugger actions, what explicit user-facing indicators and warnings are required when thin mode lacks thick-only diagnostics/features?

## Design Lock Decisions
1. Lock 1 (2026-08-25): Host capability truth is interface-owned in v1 through `SupportsCapability(...)`.
- Accepted.
- Current placement on `IHostRuntimeCommandProcessorClient` is provisional and may be relocated to a more ownership-aligned host interface in a future refactor.
- A dedicated capability endpoint is deferred unless capabilities must vary dynamically per request, session, or tenant.

2. Lock 2 (2026-08-25): Runtime bootstrap required fields and mode parity.
- Accepted.
- Required bootstrap fields: render surface width/height, stable session/game identity, current room id, and initial session-delta watermark seed.
- Requirement parity is enforced across thick and thin simulator modes: host/simulator runtime data requirements must be the same regardless of transport mode.
- UX labels (game label, narrative path display label, current room display name, asset-cache hint metadata) are optional; when absent, simulator must emit explicit diagnostics and use placeholders rather than runtime-graph fallback.

3. Lock 3 (2026-08-25): Startup ownership, mode selection, and session initiation semantics.
- Accepted.
- Full IHostSessionManagementClient support is required in both modes, including StartSession and JoinSession.
- User-direct simulator launch requires explicit session intent after startup: user must choose Start Session or Join Session.
- Mode selection is explicit at startup (Thick vs Thin).
- Thin mode requires explicit remote host endpoint configuration (address/URL and related connection settings) from startup settings or launch context; simulator must not silently guess endpoint values.
- Designer-launched simulator test flow defaults to Thick mode and may auto-start and auto-join a session for the current test run.
- Implicit reattach to a last-known session is disallowed unless an explicit startup session id is provided by launch context.

4. Lock 4 (2026-08-25): Correlation, idempotency, and retry discipline.
- Accepted.
- HostRequestContext correlation fields are mandatory for all operations.
- Mutating operations must carry idempotency keys when retry is possible; server deduplicates by principal plus session plus idempotency key within a bounded TTL and returns deterministic replay-safe outcomes.
- Automatic client retries are read-only by default unless a mutating call is explicitly idempotent and retry-safe.

5. Lock 5 (2026-08-25): Error standardization and retry ownership split.
- Accepted.
- GameClient must translate transport-level failures into existing host result envelopes; no new error envelope contract is introduced.
- Exceptions are reserved for programming or configuration defects, not normal runtime transport outcomes.
- Retry ownership split:
	- GameClient owns bounded automatic retries for read/query operations under transient transport failure conditions.
	- Mutating operations are single-attempt by default, unless explicitly idempotent and retry-safe.
	- Simulator/host UX owns user-facing recovery behavior (prompting, retry intent, degraded-state messaging, reconnect flows).

6. Lock 6 (2026-08-25): Transport endpoint versioning policy.
- Accepted.
- GameHost and GameClient transport APIs use explicit URL versioning (`/api/v1/*`).
- Shared contract assembly version changes do not implicitly alter transport API version.
- Within v1, additive-only evolution is allowed; breaking changes require a new API version.

7. Lock 7 (2026-08-25): Authentication boundary and hardening scope for v1.
- Accepted.
- The existing identity seam is in scope and used as the MVP authentication pattern through IHostIdentityManagementClient.
- Thin-mode v1 proceeds with this current identity model for authenticated caller context and role-aware session and discovery flows.
- Additional network-edge hardening (front-door web server, infrastructure-grade auth gateway, and related deployment hardening) is deferred to a later phase and is explicitly out of scope for this plan.

8. Lock 8 (2026-08-25): Authorization scope model with mode parity.
- Accepted.
- Authorization policy is the same in thick and thin modes; transport mode does not change access rules.
- Session APIs enforce membership and ownership rules.
- Discovery and asset APIs enforce game-scope authorization in addition to authenticated caller context.
- Access is deny-by-default when scope resolution is missing or ambiguous.

9. Lock 9 (2026-08-25): Session consistency model with client-owned delta progression.
- Accepted.
- Consistency guarantees are session-scoped and mode-parity applies (same semantics in thick and thin).
- Server maintains an append-only session delta journal with monotonic sequence and watermark semantics.
- Client owns progression through that journal: it must not issue overlapping or out-of-order delta requests for the same session stream.
- Server does not maintain per-client expected-next-sequence state.
- On client-detected gaps, stale watermarks, or ordering uncertainty, client must request re-baseline/resynchronization from a known watermark instead of speculative merge.

10. Lock 10 (2026-08-25): High-frequency delta polling cadence and tuning ownership.
- Accepted.
- Delta polling remains frequent (faster than once per second) so background runtime changes (for example timers) are reflected quickly.
- No adaptive polling backoff is introduced in this phase.
- Simulator owns polling cadence configuration (tunable setting) so the team can tune interval behavior during validation.
- Client continues to enforce non-overlapping per-session poll progression (single in-flight progression semantics), advancing watermark only after response processing.
- Long-poll and websocket push models are explicitly deferred; this phase must not preclude either follow-on transport strategy.

11. Lock 11 (2026-08-25): Contract-first change control and drift prevention.
- Accepted.
- Contract review and lock happen early in the phase before broad endpoint/client expansion.
- Default posture is minimal/no contract change; additive changes require explicit approval before implementation.
- Once approved, contract shape is treated as locked for the phase unless a new lock decision is recorded.
- Endpoint-client parity validation remains a mandatory CI guard to prevent post-lock implementation drift.

12. Lock 12 (2026-08-25): Thin-mode UX visibility and diagnostics contract.
- Accepted.
- Simulator UX must always expose runtime mode (Thick vs Thin), connection context, and feature capability state.
- Thin-unavailable features are shown as disabled with concise reason labels rather than being silently removed.
- Direct-launch startup UX must keep explicit Start Session vs Join Session intent choices.
- Error states must surface concise actionable messages with access to diagnostics details.

13. Lock 13 (2026-08-25): GameHost bind and listen configuration contract.
- Accepted.
- GameHost bind URL and port are explicit configuration inputs with deterministic precedence (command line, environment, appsettings, defaults).
- Safe local defaults are loopback-oriented.
- Non-loopback exposure requires explicit opt-in configuration and remains aligned with the authentication boundary lock.

14. Lock 14 (2026-08-25): Simulator thin endpoint configuration contract.
- Accepted.
- Thin mode requires explicit remote endpoint configuration (base URL and connection settings) from startup context or simulator settings.
- Simulator must fail fast with actionable diagnostics when thin mode is selected but endpoint configuration is missing or invalid.
- Endpoint guessing or implicit discovery by heuristic is disallowed.

15. Lock 15 (2026-08-25): Transport generation utility ownership and workflow.
- Accepted.
- A new dedicated utility is introduced for interface-to-transport generation; this is separate from Storyboard.SchemaCodegen.
- Scope boundary: Storyboard.SchemaCodegen remains focused on JSON schema to C# DTO generation.
- New utility input: host C# interfaces and transport metadata.
- New utility output: server endpoint registration assets, client proxy assets, and parity metadata.
- Workflow: manual generation run, human review of generated diffs, then commit accepted output.
- CI verify mode is required and fails when generated artifacts are stale relative to interface source.

16. Lock 16 (2026-08-25): Lightweight host API discovery and documentation strategy.
- Accepted.
- Host exposes a generated lightweight API manifest endpoint family for interface, method, route, request type, response type, and capability/discovery metadata.
- Manifest output is generated by the same transport utility to avoid duplicate truth.

## Development Readiness
All design locks required for phase-start are accepted, including:
1. Host bind/listen configuration contract.
2. Simulator thin endpoint configuration contract.
3. Dedicated transport generation utility (separate from SchemaCodegen).
4. Lightweight generated API discovery manifest approach.

Schema impact status:
1. Current assessment: no schema changes are required to start development for this effort.
2. Primary work is transport/composition generation and host-client wiring over existing interfaces.
3. Any newly discovered schema/contract delta must be proposed and approved before implementation of that delta.

## Completed Development-Start Prereqs (Superseded)
1. Implementation has progressed beyond kickoff gates; this section is now informational only.
2. Configuration keys and startup validation behavior are implemented for GameHost bind policy and simulator thin endpoint settings.
3. Transport generation workflow exists with generate/verify commands and CI stale-artifact enforcement.
4. Vertical-slice kickoff scope (attach session, baseline/current presentation, process command) is implemented and expanded.
5. Build/parity guardrails are active in CI.

## Remaining Work (Updated 2026-08-26)
1. Complete Phase 5 auth-boundary hardening for non-local deployments (beyond current MVP identity seam).
2. Expand deployment/runbook documentation for production-style host hosting (publish artifacts, network exposure, and operator guidance).
3. Resolve or intentionally baseline outstanding nullable/analyzer warnings in runtime and test projects to tighten quality gates.

## Item 3 Closure: Delta Polling Hardening (Locked 2026-08-26)
This item is considered closed for this plan scope.

Delivered in-scope outcomes:
1. Simulator delta background polling cadence is now user-tunable through runtime connection settings UI and persisted settings.
2. Poll interval is normalized and bounded (100ms to 5000ms) to prevent unsafe values.
3. Background polling loop uses configured base cadence and bounded exponential backoff derived from that base interval.
4. Focused regression tests cover normalization bounds, persisted settings clamp behavior, and backoff math.
5. Required build and playback regression gates passed after changes.

Out-of-scope for this plan closure (deferred follow-up):
1. Multi-client stress, long-duration concurrency, and soak-style polling consistency suites.
2. Transport model upgrades (long-poll/websocket push) already deferred by existing lock decisions.

## Deferral Note (2026-08-26)
1. Remaining Work items 1 and 2 are intentionally deferred to a later follow-up phase.
2. Transport upgrade discussion (long-poll and websocket push) is intentionally deferred and will return as a new follow-up item.
3. Direction for that follow-up: target eventual support for both transport modes rather than a forced single-mode choice.
