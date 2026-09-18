# Host Identity, Discovery, and Session Management Plan

Status: In Progress (core scope largely implemented; hardening and multi-session follow-up pending)
Owner: Host contract and runtime boundary design
Last updated: 2026-08-13

## 1. Purpose

Define a new host-facing management contract surface for:

1. User identity (authentication and principal context).
2. Poor-man authorization (roles/permissions).
3. Game discovery by id/key rather than project file path.
4. Session management by game/session identifiers.

This plan intentionally precedes deeper asset management implementation.

## 2. Why This Plan Exists

Current host integration successfully shields gameplay internals, but still assumes local filesystem coupling at the runtime boundary.

Examples of current coupling:

1. `LoadedProjectFilePath` on thin client contract.
2. `LoadGameRuntimeProject(string projectFilePath)` entrypoint.

Desired direction:

1. Host discovers games via contract-level identity metadata.
2. Host starts/joins sessions via ids, not file paths.
3. Authorization governs visibility/start/join operations.
4. Runtime-specific storage/path details remain internal.

## 3. Scope

In scope:

1. New host management DTO families and naming conventions.
2. New host management interface seams.
3. Minimal auth/authz model sufficient for local + hosted evolution.
4. Compatibility strategy with existing gameplay command contracts.

Out of scope (for this first planning cycle):

1. Full production identity provider implementation.
2. Token signing infrastructure and external IAM integration.
3. Full asset resolution protocol and transport abstraction.
4. Removal of existing gameplay command DTO contracts.

## 4. Folder and Naming Direction

New DTO folders under HostContracts:

1. `Storyboard.Shared.Contracts/HostContracts/UserManagementDtos`
2. `Storyboard.Shared.Contracts/HostContracts/GameDiscoveryDtos`
3. `Storyboard.Shared.Contracts/HostContracts/SessionManagementDtos`

Existing gameplay command DTO folder remains unchanged:

1. `Storyboard.Shared.Contracts/HostContracts/HostContractDtos`

Naming convention:

1. Keep `Host` prefix.
2. Keep `_contract` suffix for schema-emitted DTO artifacts.
3. Use operation-oriented names for request/result shapes.

## 5. Proposed Contract Families (Draft)

User management:

1. HostUserLoginRequest
2. HostUserLoginResult
3. HostUserPrincipal
4. HostUserRole
5. HostUserPermission
6. HostUserLogoutRequest
7. HostUserLogoutResult

Game discovery:

1. HostDiscoverGamesRequest
2. HostDiscoverGamesResult
3. HostGameDescriptor
4. HostGameAccessDescriptor

Session management:

1. HostStartSessionRequest
2. HostStartSessionResult
3. HostListSessionsRequest
4. HostListSessionsResult
5. HostSessionDescriptor
6. HostJoinSessionRequest
7. HostJoinSessionResult
8. HostLeaveSessionRequest
9. HostLeaveSessionResult

Common envelope (pattern reused by results):

1. Success
2. Code
3. Message
4. Diagnostics
5. CorrelationId
6. OccurredUtc
7. Retryable

Common request context (pattern reused by management requests):

1. RequestId
2. CorrelationId
3. PrincipalId or credential handle
4. TenantId or OrgId (when applicable)
5. ClientSentUtc
6. IdempotencyKey (for create/start style operations)
7. Optional diagnostics/trace flags

## 6. Interface Direction (Draft)

Candidate new method groups:

1. Identity/auth interface (authenticate, refresh/revalidate, logout, whoami).
2. Discovery interface (list games, get game details by id/key).
3. Session interface (start/list/join/leave/get status).

Compatibility note:

1. Existing `IHostRuntimeGameManagerThinClient` remains for gameplay command path during migration.
2. New management interfaces are additive first; path-based loading is deprecated later.
3. Direct file-path and snapshot loading are considered debugger-thick concerns and may live on `IHostRuntimeGameDebugger` (or a debugger-adjacent interface), while thin host startup moves to id/key-based session start.

## 7. Migration Phases (Draft)

Phase 1: Contracts and stubs

1. Add DTO schemas for user/discovery/session families.
2. Add generated DTO artifacts and lock manifests.
3. Add skeleton interfaces with no-op or config-backed stubs.

Phase 2: Poor-man auth/authz

1. File/JSON-backed users, roles, permissions.
2. Visibility filtering for discovery and sessions.
3. Permission checks for start/join actions.

Phase 3: Host integration

1. Host login flow and principal context.
2. Discovery-first launch flow.
3. Session list/start/join UX wiring.

Phase 4: Path-decoupling hardening

1. Minimize direct host dependence on runtime project file paths.
2. Introduce explicit asset root/asset resolver contracts.
3. Deprecate path-centric members after stability period.

## 7A. Recommended AppRoles (Draft)

These roles are a starting set for the shared hardened enumeration registry and the AppRole -> Group -> User model.

Discovery and catalog:

1. `DiscoverPublicGames`
2. `DiscoverActionableGames`
3. `ViewGameDetails`

Session lifecycle:

1. `StartNewGameSession`
2. `JoinExistingGameSession`
3. `LeaveOwnSession`
4. `ViewOwnSessions`

Administrative session visibility and control:

1. `ViewAllSessions`
2. `ManageAnySession`
3. `TerminateAnySession`

User and security administration:

1. `ViewUserDirectory`
2. `ManageUsers`
3. `ManageGroups`
4. `ManageRoleAssignments`

Operational governance:

1. `ViewAuditTrail`
2. `ManageGameCatalog`
3. `ManageAuthorizationPolicy`

Notes:

1. AppRole strings should be stable, case-sensitive, and centrally registered.
2. Groups map to sets of AppRoles.
3. Users receive effective AppRoles through group membership.
4. Per-game overrides can be modeled as scoped role grants (for example `StartNewGameSession` scoped to one game key).

## 7B. Simulator Dual-Mode Host Abstraction Pattern

Goal:

1. Keep simulator code stable while enabling a runtime switch between in-process C# service calls and future remote REST calls.

Pattern:

1. Add simulator-local client abstractions for management and gameplay interactions.
2. Provide two implementation families:
3. InProcess adapters that call C# interfaces directly.
4. Remote adapters that call REST endpoints and map responses to the same contracts.
5. Select adapter family through startup configuration and DI at composition root.

Recommended simulator client seams:

1. `ISimulatorIdentityClient`
2. `ISimulatorDiscoveryClient`
3. `ISimulatorSessionClient`
4. `ISimulatorGameplayClient`
5. `ISimulatorDebuggerClient` (optional capability for thick/debug workflows)

DI/config requirements:

1. Config key: host transport mode (`InProcess` or `Remote`).
2. Remote settings: base URL, timeout, retry policy, and auth bootstrap settings.
3. Capability profile from provider so simulator UI can hide unsupported actions.

Guardrails:

1. Simulator view model does not depend on transport details.
2. Thin gameplay and management contracts remain path-agnostic.
3. Debugger-thick features stay explicitly optional and capability-gated.

## 8. Design Lock-Off Questions

Decision status key:

1. Open: needs explicit decision.
2. Locked: approved and implementation-ready.

| ID | Question | Options | Recommended | Status |
| --- | --- | --- | --- | --- |
| HDSM-01 | Should auth be mandatory before discovery? | A) yes; B) allow anonymous discovery subset; C) split public anonymous discovery and authenticated actionable discovery | C | Locked |
| HDSM-02 | Principal identifier format? | A) GUID; B) stable string key; C) both | C | Locked |
| HDSM-03 | Canonical game identifier? | A) GUID; B) key string; C) GUID + key | C | Locked |
| HDSM-04 | Session identifier format? | A) GUID; B) host-generated key; C) both | A | Locked |
| HDSM-05 | Result envelope shape strategy? | A) single reusable result fields across all results; B) per-DTO custom fields; C) reusable envelope + operation-specific payload fields | C | Locked |
| HDSM-06 | Error code taxonomy ownership? | A) shared hardened enumeration registry; B) freeform per operation | A | Locked |
| HDSM-07 | Poor-man auth backend source? | A) JSON config via provider pattern (disk file or injected JSON string for tests); B) in-memory only; C) mixed with fallback | A | Locked |
| HDSM-08 | Password handling for MVP? | A) plaintext (temporary); B) hashed with salt; C) delegated external auth only | B | Locked |
| HDSM-09 | Authorization model shape? | A) roles only; B) permissions only; C) three-tier AppRole -> Group -> User (roles + permissions) | C | Locked |
| HDSM-10 | Discovery visibility rule? | A) filter invisible games out; B) include with denied flag | A | Locked |
| HDSM-11 | Start-session permission granularity? | A) global permission only; B) per-game permission overrides | B | Locked |
| HDSM-12 | Session visibility rule? | A) owner-only by default with admin `ViewAllSessions` override on the same interface; B) global view-all | A | Locked |
| HDSM-13 | Join-session authorization baseline (coarse authz layer)? | A) open join if visible; B) explicit `JoinExistingGameSession` permission required | B | Locked |
| HDSM-13A | Join-session admission policy (session-level access layer)? | A) open admission for visible sessions; B) invite-code or owner approval required; C) host-config selectable by session | C | Locked |
| HDSM-14 | Session ownership transfer needed in MVP? | A) yes; B) no | B | Locked |
| HDSM-15 | Correlation id generation ownership? | A) host-generated; B) engine-generated; C) host optional, engine fallback | C | Locked |
| HDSM-16 | Token/session credential lifetime policy? | A) no expiry in MVP; B) short fixed expiry; C) configurable expiry | C | Locked |
| HDSM-17 | Re-auth behavior after expiry? | A) hard fail requiring login; B) silent refresh path; C) host policy selectable | A | Locked |
| HDSM-18 | Backward compatibility for path load API? | A) keep indefinitely on thin interface; B) deprecate on thin interface, keep path/snapshot load in debugger-thick interface for diagnostics, and remove from thin later; C) remove immediately everywhere | B | Locked |
| HDSM-19 | Should discovery expose runtime file paths at all? | A) never on discovery/thin contracts; debugger-path exposure is separate and optional only if later required | A | Locked |
| HDSM-20 | Multi-tenant readiness marker in contracts? | A) include tenant/org fields now; B) defer until needed | A | Locked |
| HDSM-21 | Audit trail requirements in result envelope? | A) include actor/action/time minimal fields now; B) defer | A | Locked |
| HDSM-22 | Interface split strategy? | A) single management interface; B) three focused interfaces (identity/discovery/session) | B | Locked |
| HDSM-23 | Common request context payload strategy? | A) shared request context on all management requests; B) optional per-request custom metadata only | A | Locked |

## 9. Suggested Lock Sequence

1. HDSM-03 game identifier strategy.
2. HDSM-04 session identifier strategy.
3. HDSM-22 interface split strategy.
4. HDSM-05 result envelope strategy.
5. HDSM-06 error code taxonomy ownership.
6. HDSM-07 poor-man auth backend source.
7. HDSM-08 password handling baseline.
8. HDSM-09 authorization model.
9. HDSM-10 through HDSM-13 visibility and join policy rules.
10. HDSM-18 compatibility/deprecation timeline.
11. HDSM-19 path exposure policy.
12. HDSM-20 and HDSM-21 future-hosting guardrails.

## 10. Exit Criteria

1. HDSM-01 through HDSM-23 plus HDSM-13A locked or explicitly deferred.
2. DTO schema touch-list approved.
3. Interface signature draft approved.
4. Backward compatibility policy approved.
5. Initial stub implementation plan approved.

## 11. Initial Implementation Slice (After Lock-Off)

1. Add UserManagementDtos, GameDiscoveryDtos, SessionManagementDtos schema files.
2. Generate and lock first batch of additive DTO contracts.
3. Add new host management interfaces (no behavior change to gameplay command flows).
4. Implement config-backed identity/discovery/session stubs.
5. Add focused tests for auth, visibility filtering, and session permission checks.

## 12. Step-Wise Phased Execution Plan (Interface-First)

Execution principle:

1. Move slowly on C# interface definitions.
2. Require deep review before implementation classes.
3. Treat JSON stand-ins as first-class contract fixtures and test assets.

Phase 0: Baseline and safety rails

1. Freeze and document current simulator composition behavior.
2. Add a checklist for non-breaking migration constraints.
3. Confirm regression gate command set for each phase.

Phase 1: Interface draft package only

1. Draft new C# management interfaces in contracts project.
2. Draft request context and result envelope DTO shells.
3. No implementation classes in this phase.

Review gate 1:

1. Interface signature review with explicit approval before any concrete implementation.
2. Validate naming, scope boundaries, and optional field semantics.

Phase 2: Schema-first DTO authoring

1. Author schema files for UserManagementDtos, GameDiscoveryDtos, SessionManagementDtos.
2. Encode shared request-context and result-envelope fields consistently.
3. Define hardened code registry vocabulary and AppRole string baselines.

Review gate 2:

1. Schema review for completeness, additive compatibility, and naming consistency.
2. No implementation class creation until schema review is approved.

Phase 3: Generate and lock contracts

1. Generate schema-emitted DTO contracts.
2. Lock generated artifacts and update guardrail manifests.
3. Add or update contract drift tests for new folders.

Review gate 3:

1. Verify generated output is review-accepted and lock files are stable.
2. Validate no direct runtime file-path leakage in thin/discovery contracts.

Phase 4: JSON stand-in provider design

1. Define provider interfaces for auth, game catalog, and session metadata.
2. Implement JSON provider that supports:
3. disk-backed JSON initialization
4. injected JSON string initialization for tests
5. Document JSON shape examples and minimal sample datasets.

Review gate 4:

1. JSON model review for readability, override behavior, and deterministic tests.
2. Approval required before wiring runtime services.

Phase 5: In-process adapter implementation

1. Implement simulator-facing in-process adapters for identity/discovery/session.
2. Keep gameplay path on existing in-process interfaces.
3. Add capability profile reporting.

Review gate 5:

1. Simulator UX and API boundary review with no remote dependencies introduced.
2. Validate all focused regression tests pass.

Phase 6: Composition root transport switch

1. Add DI/config transport selection in simulator startup.
2. Register in-process adapters by default.
3. Add remote adapter stubs (contract-only, optional not-yet-active behavior).

Review gate 6:

1. Confirm simulator runs unchanged in default mode.
2. Confirm transport switch path is isolated and low-risk.

Phase 7: Remote adapter activation (later)

1. Implement REST-backed adapters behind the same simulator client interfaces.
2. Add targeted integration tests for auth/discovery/session flows.
3. Keep debugger-thick operations local or explicitly remote-capability gated.

Review gate 7:

1. Security and authorization behavior review.
2. Rollout readiness review for mixed local/remote hosting.

## 13. Preferred Execution Mode: Vertical Slice Per Interface

To get full-process confidence early, execute one interface completely end-to-end before starting the next interface.

Execution order:

1. Identity interface slice first.
2. Discovery interface slice second.
3. Session interface slice third.

Per-interface end-to-end checklist (repeat for each slice):

1. Draft C# interface signatures.
2. Deep review and lock signatures before implementation.
3. Author schema files for that interface's DTO set.
4. Deep review and lock schema semantics.
5. Generate and lock DTO artifacts.
6. Create JSON stand-in samples and provider mappings.
7. Implement in-process adapter path.
8. Wire simulator through abstraction layer (no transport leakage).
9. Add focused tests for contract, authz, and simulator behavior.
10. Validate build plus focused regression gates.

Review gates per slice:

1. Interface gate.
2. Schema gate.

## 14. Implementation Status Snapshot (2026-08-13)

Status update (2026-08-13):

1. Thin-host path-decoupling hardening is complete and no longer a closeout blocker for this plan.

Completed (implemented and validated):

1. Identity, discovery, and session management interfaces exist and are wired:
- `IHostIdentityManagementClient`
- `IHostGameDiscoveryClient`
- `IHostSessionManagementClient`
2. Session persistence seam exists and is wired:
- `IHostSessionPersistenceClient`
3. DTO families are present under HostContracts:
- `UserManagementDtos`
- `GameDiscoveryDtos`
- `SessionManagementDtos`
4. Config/JSON-backed providers and factories are present:
- `JsonRuntimeIdentityAuthProvider` + `RuntimeIdentityProviderFactory`
- `JsonRuntimeGameDiscoveryProvider` + `RuntimeGameDiscoveryProviderFactory`
5. Simulator host composition is discovery/session/identity first:
- Startup uses identity/discovery/session clients.
- Discovery pane flow starts sessions by game id/key.
- Sessions tab exists with list/join/leave/end interactions.
6. Runtime session lifecycle supports `Start/List/Get/Join/Leave/End` and attach flow.
7. Guardrail and focused regression tests are in place and passing in recent runs:
- `SessionManagementLifecycleTests`
- `SimulatorSessionManagementStartFlowTests`
- required playback regression gate.

Partially complete / changed since original draft intent:

1. `LeaveSession` and `EndSession` semantics are now split:
- `LeaveSession` keeps session active for rejoin.
- `EndSession` terminates session/runtime state.
2. Thin gameplay remains active-context based (single attached runtime context per manager instance).

## 15. Remaining To Call This Plan Complete

Scope note:

1. Multi-session runtime expansion work is tracked in [MULTI_SESSION_STARTSESSION_PLAN.md](MULTI_SESSION_STARTSESSION_PLAN.md).
2. The previous remaining items 2 through 4 in this section are now owned by that plan and are excluded from this plan's closure gate.

Completion note:

1. Path-decoupling hardening for thin-host startup is complete:
- Path-based runtime load is retained on debugger-thick contract surfaces.
- Thin runtime gameplay contract excludes path-based loading and uses session/discovery-driven startup flow.
- Remaining path-based usage in simulator is debug/thick workflow only.

Must complete:

1. Complete phase-6/7 transport goals from this plan when scheduled:
- Formal transport switch verification.
- Remote adapter activation and integration/security review.

Nice-to-have before closure:

1. Add focused simulator UI tests for session command enablement and session switching semantics.
2. Add explicit deprecation notes/timeline for thin path-centric members once migration is complete.
3. Generated contract/lock gate.
4. Adapter and simulator wiring gate.
5. Test and regression gate.

Definition of done per slice:

1. Interface approved.
2. Schemas approved and locked outputs stable.
3. JSON stand-ins reviewed and deterministic for tests.
4. Simulator runs in in-process mode through abstraction.
5. No regressions in required test filters.

Initial recommended first slice:

1. Identity interface, because auth context and request/result envelope behavior influence all later discovery/session flows.

## 14. Slice 1 Start: Identity Interface (Concrete Work Plan)

Status:

1. Active
2. Scope limited to identity/auth only
3. No discovery/session interface implementation in this slice
4. Draft signatures captured in [SLICE1_IDENTITY_INTERFACE_DRAFT.md](SLICE1_IDENTITY_INTERFACE_DRAFT.md) for pre-implementation review

Primary goal:

1. Deliver identity/auth contracts, schema-emitted DTOs, JSON stand-ins, and in-process simulator wiring behind abstraction with focused tests.

Step 1: Interface draft and lock (no implementation classes yet)

Target files:

1. [Storyboard.Shared.Contracts/HostContracts/HostContractMethods](Storyboard.Shared.Contracts/HostContracts/HostContractMethods) (new identity interface file)
2. [Storyboard.Shared.Contracts/HostContracts/HostContractMethods/IHostRuntimeGameManagerThinClient.cs](Storyboard.Shared.Contracts/HostContracts/HostContractMethods/IHostRuntimeGameManagerThinClient.cs) (compatibility notes only, no breaking removal yet)

Draft interface candidates:

1. IHostIdentityManagementClient
2. Methods:
3. Authenticate
4. Logout
5. GetCurrentPrincipal

Interface review gate:

1. Signature review and explicit approval before any implementation classes.
2. Verify request-context and result-envelope usage is consistent.

Step 2: Schema authoring for identity DTOs

Target schema folder:

1. [Storyboard.Shared.Contracts/HostContracts/Schemas/UserManagementDtos](Storyboard.Shared.Contracts/HostContracts/Schemas/UserManagementDtos)

Initial schema set:

1. HostRequestContext_contract.schema.json
2. HostResultEnvelope_contract.schema.json
3. HostAuthenticateRequest_contract.schema.json
4. HostAuthenticateResult_contract.schema.json
5. HostLogoutRequest_contract.schema.json
6. HostLogoutResult_contract.schema.json
7. HostGetCurrentPrincipalRequest_contract.schema.json
8. HostGetCurrentPrincipalResult_contract.schema.json
9. HostPrincipalDescriptor_contract.schema.json
10. HostPrincipalRoleDescriptor_contract.schema.json
11. HostPrincipalPermissionDescriptor_contract.schema.json

Schema review gate:

1. Confirm no filesystem path leakage.
2. Confirm tenant/org and audit-ready metadata fields are present where expected.
3. Confirm hardened result code usage points are explicit.

Step 3: Generate and lock identity DTO contracts

Target generated folder:

1. [Storyboard.Shared.Contracts/HostContracts/UserManagementDtos](Storyboard.Shared.Contracts/HostContracts/UserManagementDtos)

Lock artifacts:

1. Add identity DTO lock manifest entries and guardrail coverage.

Generation/lock review gate:

1. Generated output reviewed and accepted before progressing.
2. Guardrail tests updated to enforce schema-emitted parity for new identity DTO set.

Step 4: JSON stand-ins and provider contracts

Target runtime-side provider abstractions:

1. New provider interfaces under shared/runtime service area for identity source access.

Target JSON stand-ins:

1. Users
2. Groups

## 15. Simulator Discovery UX Lock-Off (Pre-Implementation)

Purpose:

1. Lock simulator discovery UX decisions before implementing simulator-side discovery controls.
2. Keep discovery UI isolated from existing simulator main view/model bloat.
3. Preserve gameplay screen real estate while making game switching easy.

Structural direction (locked):

1. Add a tabbed left pane where:
2. Tab A is Runtime State (existing tree).
3. Tab B is Discover Games (new isolated view + viewmodel).
4. Discovery should be implemented as its own control/viewmodel, not merged into existing monolithic surfaces.

Decision status key:

1. Open: needs explicit decision.
2. Locked: approved and implementation-ready.

| ID | Question | Options | Recommended | Status |
| --- | --- | --- | --- | --- |
| SDUX-01 | Preview image presentation style for selected game? | A) single large preview image; B) large preview + multi-image navigation; C) thumbnail preview only | C (thumbnail only; rotate every 10s when multiple images exist) | Locked |
| SDUX-02 | Preview image sizing/crop policy? | A) fixed 16:9 `UniformToFill`; B) `Uniform` no-crop; C) dynamic image-native size | B (fit thumbnail within fixed pane bounds, preserve full image) | Locked |
| SDUX-03 | Discovery list density model? | A) compact list rows; B) large card grid; C) toggle compact/card | A (single compact list; no card mode toggle in MVP) | Locked |
| SDUX-04 | Start action behavior? | A) select row + Start button; B) double-click starts; C) both | C (support both explicit Start and double-click launch) | Locked |
| SDUX-05 | After Start, should UI switch to Runtime State tab automatically? | A) yes always; B) no stay on Discover; C) user preference | A (always switch to Runtime State after successful Start) | Locked |
| SDUX-06 | Discovery refresh policy? | A) manual only; B) auto on first open + manual refresh; C) polling timer | B (auto load once on first tab open, plus explicit manual Refresh) | Locked |
| SDUX-07 | Invalid/broken registration entries visibility? | A) show with warning and disabled Start; B) hide invalid entries; C) show only in diagnostics mode | A (show with warning badges/reason text; disable Start) | Locked |
| SDUX-08 | Selection persistence scope? | A) in-memory for current simulator session only; B) persisted across restarts; C) none | A (remember selection for current simulator process session only) | Locked |

Lock-off order:

1. SDUX-01 preview presentation.
2. SDUX-02 preview sizing policy.
3. SDUX-03 list density.
4. SDUX-04 start behavior.
5. SDUX-05 post-start tab behavior.
6. SDUX-06 refresh policy.
7. SDUX-07 invalid entry behavior.
8. SDUX-08 selection persistence.

Compact-list visual guideline (locked for MVP):

1. Use one compact list only.
2. Keep preview thumbnails reasonably readable, not tiny (target approximately 160x90 logical pixels, with flexibility to adapt to pane width).
3. Prioritize readability over maximum row compression.

Exit criteria before simulator discovery implementation starts:

1. SDUX-01 through SDUX-08 all set to Locked.
2. Simulator UX ownership boundaries accepted (new discovery view/viewmodel/service seam).
3. Initial control layout and state transitions documented.
3. AppRoles
4. Group-role mapping
5. User-group mapping

Provider behavior requirements:

1. Disk JSON loading support.
2. Injected JSON string initialization support (tests).
3. Deterministic parse/validation diagnostics.

JSON/provider review gate:

1. Sample JSON reviewed for readability and maintainability.
2. Test bootstrap usage proven with injected JSON path.

Step 5: Simulator abstraction and in-process adapter wiring

Target files:

1. [Storyboard.Simulator/App.xaml.cs](Storyboard.Simulator/App.xaml.cs)
2. [Storyboard.Simulator/ViewModels/SimulatorViewModel.cs](Storyboard.Simulator/ViewModels/SimulatorViewModel.cs)
3. New simulator client abstraction files under simulator services layer.

Wiring goals:

1. Introduce identity client abstraction in simulator.
2. Register in-process adapter by default via config-aware composition.
3. Keep simulator behavior unchanged for non-identity flows.

Simulator review gate:

1. Verify no transport-specific logic leaks into view model.
2. Verify debugger-thick behavior remains capability-gated.

Step 6: Focused validation for Slice 1

Required validation:

1. Build solution.
2. Run focused runtime boundary regression suite.
3. Run updated guardrail tests for identity DTO schema parity.

Definition of done for Slice 1:

1. Identity interface signatures approved and committed.
2. Identity schemas approved, generated, and locked.
3. JSON stand-ins reviewed and used in tests.
4. Simulator resolves identity through abstraction in in-process mode.
5. Required tests are green.
