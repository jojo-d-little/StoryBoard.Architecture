# Navigation Traversal Enhancement Plan

Status: In Progress (Phases 1-4 partially implemented)
Owner: StoryboardDesigner authoring + Storyboard.Shared runtime
Last updated: 2026-07-05

Review note (2026-07-05):

1. This remains an active plan with substantial remaining work in phases 4-6.
2. Near-term sequencing and cross-plan prioritization are tracked in plans/future/CONSOLIDATED_OUTSTANDING_PLAN.md.

## 0. Current Implementation Status Snapshot

Status legend:

1. Completed: implemented and validated in current branch history.
2. In Progress: implemented in part, with follow-up work still open.
3. Not Started: planned but not yet implemented.

| Phase | Status | Progress Notes |
| --- | --- | --- |
| Phase 0: Planning Lock and Acceptance Baseline | Completed | Decision ledger is filled with approved baseline decisions for model, traversal semantics, outcome contract, and export policy. |
| Phase 1: Scoped Defaults and Canonical Traversal Model Scaffolding | In Progress | Canonical traversal connection model and related enum scaffolding are in use; core runtime-safe shapes are present. Remaining: explicit completion pass for full scope-precedence validation coverage and legacy-normalization diagnostics parity. |
| Phase 2: Runtime Traversal Evaluator and State Synchronization Core | In Progress | Navigate traversal execution and direction gating are active with stable outcome/result code behavior and room-context token support. Remaining: complete shared paired-openable synchronization behavior to full acceptance criteria (idempotent two-way sync, deterministic loop guard diagnostics). |
| Phase 3: Runtime Command Integration and Persistence Wiring | In Progress | Command processing integration for navigation outcomes is active; session behavior now physically moves the player object in scope-tree state. Remaining: finish traversal dynamic-state persistence lock/open continuity checks and replay-focused verification matrix. |
| Phase 4: Designer Editor Migration to Traversal Connections | In Progress | Navigation authoring UX significantly advanced: NavigateDirection editor completion, per-result echoes, and directional mapping correction/removal workflow. Remaining: full traversal-connection migration finish, scoped override UX completeness, and broad shared-property UX/diagnostics acceptance matrix. |
| Phase 5: Canonical Export Contract Finalization | Not Started | Canonical export finalization lock (schema/version transition record, migration note bundle, and final snapshot normalization pass) still pending as a dedicated phase gate. |
| Phase 6: Hardening, Migration Completion, and Cleanup | Not Started | Legacy-path removal and final migration cleanup/hardening are intentionally deferred until post-Phase-5 stabilization. |

### 0.1 Completed Highlights To Date

1. NavigateDirection action/editor/runtime pipeline implemented with configurable result-specific echo behavior.
2. Runtime now supports `currentRoom` and `priorRoom` context values for navigation outcomes and producer-authored messaging.
3. Player movement semantics updated so the player object is physically re-parented in the runtime scope tree during room transitions.
4. Directional mapping authoring flow now supports correcting and removing existing mappings.
5. Regression coverage expanded for navigation processing and player scope movement behavior.

### 0.2 Remaining Priority Work (Near-Term)

1. Complete shared paired-openable synchronization contract behavior and diagnostics guardrails.
2. Complete Phase 4 shared-property UX lock criteria (indicator, drill-in, deterministic ordering, terminology and accessibility guarantees).
3. Close Phase 5 export contract finalization with explicit schema/version migration notes and deterministic snapshot review.
4. Execute full Phase 6 cleanup once Phase 5 artifacts are locked.

## 1. Purpose

Define a durable navigation model that:

1. Supports static traversal intent (TwoWay, OneWayAtoB, OneWayBtoA).
2. Supports dynamic lock and unlock during gameplay.
3. Supports side-specific openable object association (one object per room side).
4. Supports optional shared open state between paired openable objects across both sides.
5. Prevents invalid reciprocal traversal direction configurations by construction.
6. Supports scoped traversal mode defaults with explicit overrides (project -> area -> room -> traversal leg).

This plan remains the implementation authority and is updated as phases are completed.

## 2. Problem Statement

Current directed link records allow redundant and inconsistent states for the same room pair. That makes authoring and runtime reasoning harder, and it permits invalid reciprocal movement definitions.

We need a canonical traversal connection model where traversal directionality and dynamic traversability can be expressed without duplicate link rows.

## 3. Goals

1. Eliminate duplicate traversal-direction records for the same room pair in authored data.
2. Preserve gameplay flexibility with asymmetric dynamic access.
3. Bind traversability to openable game objects from each side.
4. Support explicit two-object open-state synchronization for paired doors.
5. Keep designer and runtime boundaries clean.
6. Provide a controlled migration path to canonical traversal export shape with explicit contract updates.
7. Encourage consistency with defaults while allowing explicit exceptions per traversal leg.

## 4. Non-Goals (Initial Scope)

1. No immediate redesign of all room interaction systems.
2. No unversioned clean export breaking change; schema changes must be explicit and documented.
3. No hard requirement that both sides reference the same logical door object unless shared mode is explicitly enabled.

## 5. Proposed Canonical Model

## 5.0 Traversal Granularity Scope Model

Traversal mode supports hierarchical defaults and local overrides.

Scope model:

1. Project sets DefaultTraversalMode (FourDirectional or EightDirectional).
2. Area can optionally override project default.
3. Room can optionally override effective area setting.
4. Traversal leg can optionally override effective room setting.

Resolution rule (top-down):

1. EffectiveTraversalLegTraversalMode = TraversalLegOverride
2. else RoomOverride
3. else AreaOverride
4. else ProjectDefault

Expected behavior:

1. Editor dropdowns and auto-link generation should follow effective defaults at the current scope to encourage consistency.
2. Author can still explicitly create a traversal leg that exceeds default strictness, such as a diagonal leg inside a FourDirectional default context.
3. Override is explicit and visible so exceptional links are intentional, not accidental.
4. Applying room override scope to both inbound and outbound context enables authoring of either direction, but does not mandate reciprocal traversal leg definitions.

## 5.1 Connection Identity

Introduce a canonical traversal connection entity with exactly one record per unordered room pair.

Suggested shape:

- TraversalConnectionId: Guid
- RoomAId: Guid
- RoomBId: Guid
- BaseTraversalDirectionFromA: Direction
- TraversalModeOverride: TraversalMode? (optional per-traversal-leg exception)
- TraversalAccessMode: TwoWay | OneWayAtoB | OneWayBtoA
- TraversalStateFromA: DirectionTraversalState
- TraversalStateFromB: DirectionTraversalState
- OpenStateBindingMode: Independent | SharedWithPairedOpenable

DirectionTraversalState:

- IsLocked: bool
- LockReasonTag: string? (optional, future use)
- OpenableObjectId: Guid? (room-local object on that side)
- OpenStatePolicy: OpenablePolicy

OpenablePolicy:

- IgnoreOpenableState
- RequireOpen
- RequireClosed (optional advanced gameplay mode)

## 5.2 Invariants

1. RoomAId != RoomBId.
2. Only one connection record for the pair {RoomAId, RoomBId}.
3. BaseTraversalDirectionFromA implies BaseTraversalDirectionFromB = Opposite(BaseTraversalDirectionFromA).
4. TraversalAccessMode constrains static permissibility only.
5. Dynamic state further gates static permissibility and does not expand denied static traversal by default.
6. SharedWithPairedOpenable is valid only when both sides are bound to openable-capable objects.
7. Traversal-leg-level diagonal traversal direction is allowed when TraversalModeOverride resolves to EightDirectional, even if higher scopes resolve to FourDirectional.
8. No implicit traversal-leg-level override should be created by auto-linking; overrides must be explicit author intent.

## 5.3 Traversability Evaluation

For attempt from source room S to destination room D:

1. Resolve connection and oriented side (A->B or B->A).
2. Evaluate static allow from TraversalAccessMode.
3. If static denied, traversal denied.
4. Evaluate dynamic state on oriented side:
   - Deny if IsLocked is true.
   - If OpenableObjectId exists, evaluate OpenablePolicy against object open/closed runtime state.
5. Allow traversal only if all checks pass.

## 6. Openable Association Design

## 6.1 Side-Specific Binding

Each side of a traversal connection can bind to one openable object in that side's room:

- A side binds to object in RoomA.
- B side binds to object in RoomB.

This enables gameplay patterns such as:

1. A->B passable only if RoomA-side object is open.
2. B->A passable regardless of RoomB-side object.
3. Asymmetric lock controls for puzzle design.
4. Synchronized paired doors where opening either side updates both sides.

## 6.2 Validation Rules

1. Bound object must exist in the corresponding room.
2. Bound object must advertise openable capability.
3. Orphaned object references are auto-cleared with warning during validation.
4. If policy requires openable state and no object is bound, traversal defaults to denied and emits diagnostic.
5. SharedWithPairedOpenable requires both sides to be bound.
6. SharedWithPairedOpenable cannot bind both sides to the same room object identity.

## 6.3 Shared Open-State Pairing

Add a configuration option to treat side A and side B openable references as one logical open-state pair.

Behavior intent:

1. Opening A-side openable sets B-side openable open state to match.
2. Closing either side updates the paired side to closed.
3. Runtime synchronization is authoritative in Storyboard.Shared traversal/openable coordination logic.

Initial release can limit synchronization scope to the single connection where SharedWithPairedOpenable is configured.

## 7. Runtime Contract Placement

Place runtime-safe connection and traversal evaluation contracts in Storyboard.Shared.

Keep designer-only editing conveniences and UI-specific composition in StoryboardDesigner.App.

Do not introduce dependency from Storyboard.Simulator to StoryboardDesigner.App.

When shared open-state mode is enabled, synchronization logic must live in shared runtime services so both hosts honor identical behavior.

## 8. Persistence and Export Strategy

## 8.1 Authoring Persistence

Persist canonical connections in project model as a single list.

Example concept:

- Project.DefaultTraversalMode: TraversalMode
- Area.TraversalModeOverride: TraversalMode?
- Room.TraversalModeOverride: TraversalMode?
- Area.TraversalConnections: List<TraversalConnection>

During transition, support legacy Area.Links read path and normalize to connections.

## 8.2 Clean Export Contract Strategy

Locked direction:

1. Move clean export to canonical traversal connection shape now.
2. Do not preserve legacy directional-row export shape in the new contract.
3. Publish explicit schema/version update in the same change as shape transition.
4. Provide migration notes and snapshot updates in the same change set.

## 9. Migration and Normalization Plan

## 9.1 Legacy Link Ingestion

When loading legacy directed links:

1. Group by unordered pair {FromRoomId, ToRoomId}.
2. Derive BaseTraversalDirectionFromA from the A->B directed record if available.
3. Derive TraversalAccessMode:
   - both directions present -> TwoWay
   - only A->B present -> OneWayAtoB
   - only B->A present -> OneWayBtoA
4. If conflicting directions are detected for same pair, mark validation issue and repair deterministically.
5. If a diagonal link exists under an effective FourDirectional context, preserve the link and set traversal-leg-level TraversalModeOverride to EightDirectional.

## 9.2 Repair Strategy for Conflicts

Initial deterministic strategy:

1. Prefer direction implied by room placement vector when available.
2. Else prefer earliest record order.
3. Record warning diagnostics for author review.

## 10. Authoring UX Plan

## 10.1 Connection Editor

1. Select two rooms to create one connection row.
2. Edit TraversalAccessMode via explicit selector.
3. Configure per-side dynamic controls:
   - Locked toggle
   - Openable object picker scoped to side room
   - Openable policy selector
4. Configure open-state binding mode:
   - Independent
   - Shared between side A and side B openables
5. Show effective traversal mode and override controls:
   - Project default (global setting)
   - Optional area override
   - Optional room override
   - Optional traversal leg override for exceptional links

## 10.2 Canvas Rendering

1. Render single visual connection baseline between rooms.
2. Render directional indicators from TraversalAccessMode.
3. Render lock/open icons per side when configured.
4. Render pairing indicator when shared-open-state mode is active.

## 10.3 Diagnostics

Show non-blocking warnings for:

1. Missing openable target on policy requiring one.
2. Bound object removed or not openable.
3. Shared mode enabled with missing opposite-side openable binding.
4. Legacy conflict repairs applied.
5. Traversal leg override conflicts with selected traversal direction (for example FourDirectional override with diagonal direction).

Validation report usability requirements:

1. Error diagnostics must include exact affected entity path/scope (project, area, room, traversal connection, traversal leg).
2. Error diagnostics must state why the rule is violated in plain language.
3. Error diagnostics must provide a concrete next-step fix hint.
4. Blocking errors at save/export must be grouped in a clear report panel so authors can resolve issues quickly.

## 11. Implementation Phases

## 11.1 Phase Sequence Overview

1. Phase 0: Planning lock and acceptance baseline.
2. Phase 1: Scoped defaults and canonical traversal model scaffolding.
3. Phase 2: Runtime traversal evaluator and state synchronization core.
4. Phase 3: Runtime command integration and persistence wiring.
5. Phase 4: Designer editor migration to traversal connections and override UX.
6. Phase 5: Canonical export contract finalization.
7. Phase 6: Hardening, migration completion, and cleanup.

## 11.2 Phase Details

## Phase 0: Planning Lock and Acceptance Baseline

Scope:

1. Lock field names and invariants for TraversalConnection and DirectionTraversalState.
2. Lock open-state pairing semantics for SharedWithPairedOpenable.
3. Lock precedence rules for project, area, room, and traversal-leg traversal mode overrides.
4. Finalize acceptance scenarios and regression list.

Deliverables:

1. Finalized field list and naming glossary.
2. Acceptance test matrix for traversal and synchronization.
3. Decision record for export schema transition approach.
4. Decision record for traversal mode override persistence shape.

Validation gate:

1. Team signoff on model and behavior matrix.
2. No code changes required in this phase.

## Phase 0 Decision Ledger

Status key:

1. Approved: locked for implementation.
2. Open: pending explicit decision in Phase 0.
3. Deferred: intentionally postponed beyond initial implementation.

| Decision Area | Decision | Status | Notes |
| --- | --- | --- | --- |
| Terminology | Use Traversal Connection, Traversal Leg, Traversal Direction, Traversal Mode, Traversal Access Mode | Approved | Glossary aligned in TERMINOLOGY.md. |
| Plan authority | Single canonical source is NAVIGATION_TRAVERSAL_ENHANCEMENT_PLAN.md | Approved | Secondary phased plan removed to prevent drift. |
| Traversal mode hierarchy | Precedence is project -> area -> room -> traversal leg | Approved | Applies to effective traversal mode resolution. |
| Exception behavior | Traversal-leg override may allow diagonal leg under inherited FourDirectional mode | Approved | Override must be explicit author intent. |
| Auto-link behavior | Auto-linking follows effective traversal mode and does not implicitly create traversal-leg overrides | Approved | Keeps exceptions intentional. |
| Traversal access baseline | Static state defines what is possible; dynamic state can only allow/disallow within static possibilities | Approved | Locked rule: dynamic can never make traversal possible if TraversalAccessMode does not allow it. |
| Room override scope | Room override applies to both outbound and inbound editing/evaluation context, without mandating reciprocal traversal leg definitions | Approved | Locked caveat: bidirectional context enables authoring both ways, but one-way traversal remains valid and common at transition boundaries (for example indoor-to-outdoor expansion). |
| Shared open-state sync | SharedWithPairedOpenable syncs open/close from either side to both sides | Approved | Implement with idempotent loop protection. |
| Shared open-state runtime seam | Shared paired-openable sync executes with bounded, deterministic runtime behavior | Approved | Locked rule: connection-local only; side changes propagate to paired side in same command tick; updates are idempotent; propagation carries source marker to prevent rebroadcast loops; missing side binding at runtime skips sync with deterministic warning diagnostic (no crash). |
| Shared open-state grouping | Shared pairing remains connection-local only in initial release | Approved | Cross-connection grouping deferred. |
| Cross-connection shared groups | Reusable group IDs spanning multiple connections | Deferred | Revisit after initial release stability. |
| Dynamic state persistence | Mutable traversal lock/open state is runtime-only; designer persists linkage and static configuration only | Approved | Locked rule: simulator/runtime session owns current lock/open outcomes; designer stores traversal leg definitions, object bindings, traversal modes, and policies. |
| Navigation command model | Use one runtime Navigate action type with parsed traversal direction intent; allow optional direction-qualified Navigate bindings for authoring specificity | Approved | Locked rule: no 8 separate runtime action types; direction-specific bindings may override generic binding selection while sharing the same Navigate executor. Direction-specific action instances per room are supported and expected. |
| Navigate echo messaging | Each direction-qualified Navigate action instance has one success echo and one failure echo | Approved | Locked rule: use a single success/failure echo pair per action instance; no per-condition echo matrix required in initial release. |
| Navigate no-direction inference | Commands without explicit traversal direction (for example, "leave room") resolve deterministically by exit count | Approved | Locked rule: if exactly one traversable traversal leg exists from current room, infer and use it; if zero or more than one traversable leg exists, fail with disambiguation-style feedback. |
| Directional mapping authoring UX | Directionals editor supports optional mapping from producer-defined directional tokens to fixed eight-way traversal directions | Approved | Locked rule: mapping is optional per token and remains producer-controlled. Navigate explicit-direction resolution uses these mappings when present. |
| Action outcome property contract | Standardize action outcome properties across most/all actions | Approved | Locked rule: actionProperties.Success is always present and boolean (true on success, false on failure). When a typed failure/specialized outcome exists, actionProperties.ResultCode is present with a readable stable token (for example, Locked). |
| Shared-property terminology | Use "shared" terminology for cross-endpoint value synchronization in designer UX (avoid "linked" in this context) | Approved | Locked rule: keep "linked" reserved for quantifiable/link-actions language to reduce producer confusion. |
| Export contract policy | Adopt canonical traversal connection export shape now with explicit schema/version update | Approved | Locked rule: no backward-compatibility adapter for legacy directional-row export in new contract. Set canonical export schema to 2.0 for this break, preserve deterministic ordering, include migration notes, and update snapshot baselines in the same implementation change. |
| Legacy normalization | Preserve directional links deterministically, including diagonal exception repair | Approved | Repair diagnostics required for conflicts. |
| Validation severity | Strict validation by default: invariant/contract violations are errors that block save/export; deterministic repairs are warnings | Approved | Locked caveat: validation reports must be highly actionable with clear issue text, affected scope/path, reason, and concrete fix guidance for the author. |

Decision update rules:

1. Update this ledger in the same change where a decision is locked.
2. Move unresolved items to Deferred only with explicit rationale.
3. Do not start Phase 1 implementation for Open items that alter model shape or export contract semantics.

## Phase 1: Scoped Defaults and Canonical Traversal Model Scaffolding

Scope:

1. Add project default traversal mode and optional area and room overrides.
2. Add canonical traversal connection model with optional traversal-leg-level traversal mode override.
3. Add legacy directed link normalization to connection list.
4. Add invariant validation, including traversal mode override consistency diagnostics.

Project touch points:

1. StoryboardDesigner.App models and load paths.
2. Storyboard.Shared contracts for traversal connection shape where runtime-safe.
3. Scope-resolution utility for effective traversal mode.
4. Test project for normalization and validation.

Deliverables:

1. Scoped traversal mode defaults and override fields introduced.
2. Canonical traversal connection types introduced.
3. Legacy load path produces canonical records.
4. Duplicate and conflict diagnostics surfaced.

Validation gate:

1. Build succeeds.
2. New normalization tests pass.
3. Effective traversal mode resolution tests pass across all scopes.
4. Existing link-dependent tests remain green.

Rollback strategy:

1. Keep legacy link read path intact behind adapter.
2. Preserve existing area-level default behavior if scoped traversal mode overrides regress.
3. Do not remove existing directed list usage yet.

## Phase 2: Runtime Traversal Evaluator and State Synchronization Core

Scope:

1. Implement traversal evaluator in shared runtime logic.
2. Apply static traversal access mode, scoped effective traversal mode, and dynamic lock checks.
3. Implement shared open-state synchronization between side A and side B openables when enabled.

Project touch points:

1. Storyboard.Shared traversal services and contracts.
2. Storyboard.Shared openable state coordination paths.
3. Runtime-focused tests.

Deliverables:

1. Deterministic evaluator API for traversal checks.
2. Pair synchronization mechanism with loop protection and idempotent updates.
3. Traversal direction gating honors effective mode unless explicit traversal leg override is present.
4. Diagnostics for invalid shared-mode configurations.

Validation gate:

1. Focused runtime suite passes.
2. Synchronization tests pass for open and close events initiated from either side.
3. Override precedence tests pass for runtime traversal checks.
4. No event-loop regressions.

Rollback strategy:

1. Keep synchronization behind explicit connection mode check.
2. Disable shared mode behavior if instability is detected.

## Phase 3: Runtime Command Integration and Persistence Wiring

Scope:

1. Integrate lock and unlock plus open-state updates into command processing flow.
2. Ensure runtime session state captures dynamic changes.
3. Ensure save and load retain canonical traversal state.

Project touch points:

1. Storyboard.Shared command processing and manager orchestration.
2. Host adapters in StoryboardDesigner.App and Storyboard.Simulator as needed.
3. Test fixtures around command effects.

Deliverables:

1. Runtime command handlers for traversal state changes.
2. Persisted dynamic state behavior aligned with agreed policy.
3. Replay-safe behavior through save and restore.

Validation gate:

1. Command processor regression tests pass.
2. Save and reload tests prove state continuity.
3. Existing gameplay playback regressions remain green.

Rollback strategy:

1. Keep compatibility path for static traversal if dynamic state handling fails.
2. Gate new traversal command pathways by feature capability checks.

## Phase 4: Designer Editor Migration to Traversal Connections

Scope:

1. Replace directed link editing workflows with traversal connection editor workflows.
2. Add per-side openable selection and open-state policy controls.
3. Add shared-open-state mode toggle and validation UX.
4. Add project, area, room, and traversal leg override controls with effective-mode display.
5. Add shared-property visual affordances and relationship inspection UX.

Project touch points:

1. StoryboardDesigner.App area navigation editor viewmodels and views.
2. Validation and diagnostics display paths.
3. Authoring serialization adapters.

Deliverables:

1. Traversal-connection-centric editor UX.
2. Shared pairing configuration UX with constrained selectors.
3. Scoped default and override UX that encourages defaults while allowing explicit exceptions.
4. No editor path can create duplicate reciprocal rows.
5. Shared game-property indicator icon on affected properties with a clear accessible label (for example, "Shared").
6. Drill-in relationship view from a shared property showing all current share endpoints, directionality, and effective sync mode.

Validation gate:

1. UI interaction regression tests pass.
2. Manual authoring workflow test pass for symmetric and asymmetric examples.
3. Manual workflow validation confirms diagonal exception path under FourDirectional defaults.
4. Build and full test project pass.

Shared-property UX acceptance criteria (Phase 4 lock):

1. Property grid row indicator:
   - Any game property participating in one or more shared-property relationships must show a consistent shared indicator icon in the same visual column across all property grids.
   - Indicator must include an accessible text label "Shared" for screen readers and automation.
2. Terminology:
   - UI copy for this feature must use "Shared" and "Share" language.
   - UI copy must avoid "Linked" for this feature to prevent overlap with quantifiable/link-action terminology.
3. Drill-in entry point:
   - Producer can open relationship details from the property row via icon click and context menu action (for keyboard parity).
   - Interaction should require at most one click from the property row to open the relationship panel/dialog.
4. Relationship detail payload:
   - Drill-in view must list all share relationships touching the selected endpoint.
   - Each row must show: counterpart endpoint path, directionality, transform mode, enabled state, and conflict/sync mode summary.
5. Deterministic ordering:
   - Relationship rows must be sorted by Priority ascending, then RelationshipId ascending as tie-breaker.
   - Sorting must be stable across reloads and independent of creation order in UI collections.
6. Empty-state behavior:
   - If a property is not shared, drill-in view must show a clear empty-state message with a direct action to create a share relationship.
   - If relationships are invalid or stale, drill-in view must show non-blocking warnings with endpoint-specific repair guidance.
7. Hover/help affordance:
   - Indicator hover text must summarize share count (for example, "Shared with 2 endpoints").
   - Hover text must remain deterministic and update immediately after create/delete/enable/disable changes.
8. Navigation and focus:
   - Opening drill-in must retain originating property context and support one-step return focus to the same property row.
   - Keyboard navigation must fully support opening, traversing rows, and closing drill-in without mouse-only affordances.
9. Performance envelope:
   - Indicator rendering and drill-in open must remain responsive for at least 500 relationships in the active project without blocking UI thread interactions.
10. Testability:
   - Add UI/viewmodel tests covering icon visibility, drill-in population, deterministic sort order, empty-state copy, and terminology guardrails.

Shared-property result-code and diagnostics acceptance criteria (Phase 4/Runtime integration lock):

1. Runtime outcome contract:
   - Any action that triggers shared-property propagation must always emit actionProperties.Success (true or false).
   - actionProperties.ResultCode must be present for any non-success or specialized non-failure outcome.
2. Canonical shared-property result codes:
   - Initial stable code set: SharedPropagationApplied, SharedNoOp, SharedConflictIgnored, SharedCycleStopped, SharedStepLimitReached, SharedInvalidEndpoint, SharedTypeMismatch, SharedTransformInvalid.
   - Codes are contract strings and must be treated as stable tokens in tests and authored scripts.
3. Deterministic diagnostics ordering:
   - For one command tick, diagnostics related to shared propagation must be ordered by processing sequence using relationship sort order (Priority then RelationshipId).
   - Diagnostic ordering must be deterministic across repeated runs.
4. Endpoint-specific diagnostics:
   - Failure and warning diagnostics must include both source endpoint path and target endpoint path.
   - Diagnostics must include a concise fix hint when the issue is author-actionable.
5. Conflict visibility:
   - When conflict policy ignores a write, runtime must set ResultCode=SharedConflictIgnored and include a diagnostic indicating the winning endpoint/write source for the tick.
6. Guardrail visibility:
   - On cycle stop or step-limit stop, runtime must set Success=false, set the matching ResultCode, and stop further shared propagation for the tick.
   - Partial mutations that occurred before guard trigger remain committed and must be reported in diagnostics summary.
7. Designer drill-in alignment:
   - Drill-in view must surface last-known runtime issue category mapping for relationships when available (for example type mismatch, invalid endpoint, cycle stop).
   - If runtime telemetry is unavailable, drill-in must show "No runtime diagnostics yet" rather than implying healthy state.
8. Logging granularity:
   - Low/normal diagnostics should include only summary lines.
   - High diagnostics may include per-hop propagation traces with endpoint/value snapshots, still respecting deterministic order.
9. Regression coverage:
   - Add focused tests proving each canonical ResultCode can be produced deterministically.
   - Add regression tests proving deterministic diagnostic ordering for multi-hop propagation scenarios.

Shared-property implementation checklist (execution map):

1. Contracts and model scaffolding:
   - Add shared-relationship DTO/model primitives with RelationshipId, Priority, endpoint descriptors, directionality, transform mode, conflict policy, enabled flag.
   - Add endpoint descriptor support for object-property endpoints and traversal virtual endpoints.
   - Acceptance mapping: UX-4, UX-5, RT-2.
2. Deterministic ordering utility:
   - Implement one reusable ordering helper (Priority asc, RelationshipId asc) used by runtime propagation and designer drill-in list rendering.
   - Acceptance mapping: UX-5, RT-3.
3. Runtime propagation core:
   - Implement in-tick shared propagation service with deterministic per-hop evaluation, endpoint write de-duplication, and no-op detection.
   - Emit canonical result codes and actionProperties.Success/actionProperties.ResultCode outcomes.
   - Acceptance mapping: RT-1, RT-2, RT-8.
4. Runtime guardrails and conflict policy:
   - Add cycle detection and step-limit guard behavior with deterministic stop semantics.
   - Implement FirstWriterWinsThisTick conflict behavior and SharedConflictIgnored diagnostics.
   - Acceptance mapping: RT-5, RT-6.
5. Runtime diagnostics payloads:
   - Ensure endpoint-specific diagnostics include source path, target path, issue summary, and fix hint where applicable.
   - Add diagnostics-level switch between summary-only and per-hop trace output.
   - Acceptance mapping: RT-4, RT-8.
6. Designer property-grid indicator:
   - Add shared indicator icon column and accessible label "Shared" for any property participating in one or more shared relationships.
   - Add deterministic hover summary text with share count.
   - Acceptance mapping: UX-1, UX-7.
7. Drill-in relationship view:
   - Add one-click (plus keyboard/context-menu) entry from property row to relationship panel/dialog.
   - Display counterpart endpoint, directionality, transform mode, enabled state, conflict/sync mode summary, and deterministic row ordering.
   - Acceptance mapping: UX-3, UX-4, UX-5, UX-8.
8. Empty-state and stale-state UX:
   - Add create-share call-to-action empty state and non-blocking stale/invalid relationship warnings with repair guidance.
   - Surface "No runtime diagnostics yet" when telemetry snapshot is absent.
   - Acceptance mapping: UX-6, RT-7.
9. Terminology and copy pass:
   - Replace/guard all UI copy for this feature to use "Shared" / "Share" terms and avoid "Linked" in this context.
   - Add regression checks for copy terms in key views/viewmodels.
   - Acceptance mapping: UX-2.
10. Test matrix and validation gate:
   - Add/extend tests for indicator visibility, drill-in population, deterministic sorting, empty state, accessible labels, each canonical ResultCode, deterministic diagnostic ordering, and guardrail behavior.
   - Run build plus focused runtime and UX tests as phase gate.
   - Acceptance mapping: UX-10, RT-9.

Rollback strategy:

1. Keep legacy visualization read-only path available during migration window.
2. Do not remove old serializer fields yet.

## Phase 5: Canonical Export Contract Finalization

Scope:

1. Replace directional-row clean export with canonical traversal connection shape.
2. Finalize schema/version update and migration notes for the new contract.
3. Include traversal mode overrides and shared-open-state metadata in canonical contract fields.
4. Update snapshots and export validation.

Project touch points:

1. StoryboardDesigner.App export service.
2. Clean export DTOs and snapshot tests.

Deliverables:

1. Stable canonical export output generated from traversal connection model.
2. Documented schema/version update and migration notes.
3. Updated deterministic ordering guarantees.

Validation gate:

1. Clean export tests pass.
2. Snapshot changes reviewed and intentional.
3. Scoped traversal mode override serialization behavior is deterministic.
4. Contract break is intentional, documented, and versioned.

Rollback strategy:

1. If release risk is discovered, pause rollout at branch level before merge rather than dual-shape runtime branching.
2. Do not ship partially migrated contract states.

## Phase 6: Hardening, Migration Completion, and Cleanup

Scope:

1. Remove obsolete directed-link authoring write paths.
2. Harden diagnostics and repair tooling for legacy edge cases.
3. Perform final guardrail and end-to-end validation.

Project touch points:

1. StoryboardDesigner.App migration cleanup.
2. Storyboard.Shared guardrail tests.
3. StoryboardDesigner.App.Tests regression suites.

Deliverables:

1. Deprecated pathways removed.
2. Documentation and migration notes finalized.
3. Release readiness checklist complete.

Validation gate:

1. Full build and full test pass.
2. Focused runtime guardrails pass.
3. Sample project migration verification complete.

Rollback strategy:

1. Keep migration utility available for one release window.
2. Provide fallback import diagnostics for legacy projects.

## 11.3 Cross-Phase Quality Gates

Run after each coding phase:

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj

Run after runtime-touching phases:

1. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"

## 11.4 Proposed Execution Order by Workstream

1. Data model and normalization first.
2. Runtime evaluator second.
3. Runtime command and persistence third.
4. Designer UX migration fourth.
5. Export decision and snapshot stabilization fifth.
6. Cleanup and removal last.

## 11.5 Acceptance Criteria Summary

1. Exactly one canonical connection per room pair in authored data.
2. Traversal decision honors static traversal access mode, effective scoped traversal mode, lock state, and openable policy.
3. Shared mode guarantees two-way open-state sync between paired side openables.
4. No event-loop behavior during synchronization.
5. Scoped defaults resolve in precedence order: project -> area -> room -> traversal leg.
6. Clean export uses canonical traversal connection shape with explicit schema/version update.
7. Full regression and focused runtime suites pass.

## 12. Test Strategy

1. Unit tests for normalization:
   - paired links -> TwoWay
   - single direction -> OneWay
   - conflict repair deterministic behavior
2. Unit tests for evaluator:
   - lock and unlock transitions
   - openable policy combinations
   - asymmetric side behavior
   - paired openables sync when either side changes state
   - validation failure when shared mode enabled with missing side binding
   - effective traversal mode resolution precedence
   - diagonal traversal leg allowed only when effective mode is EightDirectional
3. Designer tests:
   - side openable picker validation
   - shared mode enable/disable constraints
   - scoped default and override UX behavior (project, area, room, traversal leg)
   - duplicate prevention guardrails
4. Export tests:
   - canonical traversal connection output shape and field coverage
   - deterministic canonical serialization behavior

## 13. Risks and Mitigations

1. Risk: Contract drift between authoring and runtime.
   - Mitigation: shared contract types and focused runtime guardrail tests.
2. Risk: Legacy project ambiguity during normalization.
   - Mitigation: deterministic repair + diagnostics surfaced in editor.
3. Risk: Scope creep into full interaction framework redesign.
   - Mitigation: enforce phased milestones and non-goals.
4. Risk: Event-loop or double-apply behavior when synchronizing paired open states.
   - Mitigation: idempotent state update semantics and guard against rebroadcast loops.

## 14. Open Questions

1. Should dynamic lock state be persisted in authoring data defaults, runtime save state, or both?
2. Should openable policy default to RequireOpen when object is bound?
3. Should one-way static mode still allow runtime unlock to override, or remain hard-denied?
4. Should side state include custom script conditions beyond lock/openable state in initial release?
5. Should shared open-state remain connection-local only, or support reusable cross-connection group IDs later?
6. Should room override apply to all outbound traversal legs only, or both inbound and outbound traversal leg editing affordances?

## 15. Immediate Next Planning Steps

1. Confirm final TraversalConnection field set and naming.
2. Confirm traversal evaluator precedence rules.
3. Confirm synchronization semantics for paired openables (authoritative side, conflict handling).
4. Confirm canonical export schema/version identifiers and migration note format.
5. Confirm traversal mode override persistence shape at project, area, room, and traversal leg scopes.
6. Draft a small vertical-slice implementation plan for Phase A only.
7. Define shared-property UX spec: iconography, placement in property grids, and drill-in relationship panel behavior.

## 16. Pre-Lock Command and Action Processing Checklist

Use this checklist to lock command and action processing behavior before implementation continues.

| Checklist Item | Proposed Rule | Status | Notes |
| --- | --- | --- | --- |
| Selection precedence | Resolve bindings in order: direction-qualified Navigate -> generic Navigate -> parser/system fallback failure | Approved | Locked rule: use resolved traversal direction (explicit or inferred) to select direction-qualified Navigate first; duplicates at the same scope are validation errors and runtime must not guess. |
| No-direction inference filter | For no-direction commands, treat traversable set as post-evaluation legs only (static + dynamic + openable policy); blocked legs do not count | Approved | Locked rule: infer only when exactly one traversable exit remains. Also expose actionProperties tokens traversalCanidates and traversalCanidates[0] for producer-authored echo hinting when no-direction navigation fails or needs guidance. |
| Failure taxonomy and routing | Use stable failure codes (NoExit, AmbiguousExit, Locked, Closed, StaticDenied, InvalidDirection) and route text via action failure echo first, then system fallback | Approved | Locked rule: set actionProperties.Success=false and actionProperties.ResultCode to the stable readable token on failure; on success set actionProperties.Success=true and omit ResultCode unless action-specific semantics require a non-failure outcome code. |
| Ambiguity ordering | Sort candidate exits in canonical traversal direction order before generating disambiguation feedback | Approved | Locked rule: use one canonical ordering for direction lists in actionProperties (including traversalCanidates), authored echo interpolation, and system fallback text to keep outputs deterministic. |
| Parser normalization contract | Use producer-defined directional vocabulary as authoritative input set; each directional may optionally map to a Traversal Direction | Approved | Locked rule: Navigate honors existing scope-tree directionals plus optional traversal-direction mapping metadata (for example north->North, forward->North), authored through the Directionals editor dialog. Parsing is case-insensitive with whitespace normalization. Tokens without mapping do not resolve explicit traversal direction. Preposition/object-target phrases (for example "go thru door") are deferred. |
| Side-effect ordering | Success path: evaluate -> move player -> emit success echo/events. Failure path: no move mutation -> emit failure echo/event | Approved | Locked rule: success emits actionProperties.Success=true after location mutation. Failure emits actionProperties.Success=false with ResultCode and must not mutate player location. |
| Re-entrancy guard | Allow bounded chained Navigate within a single command tick while preventing loops | Approved | Locked rule: allow up to 5 successful room transitions in one command tick. Track visited rooms per tick and stop with failure if a transition returns to a room already visited in that tick. Exceeding 5 successful transitions or cycle detection stops further transitions, emits actionProperties.Success=false, and sets ResultCode to a stable token (for example NavStepLimitExceeded or NavCycleDetected). |
| Evaluation snapshot consistency | Evaluate traversal against one command-tick-consistent dynamic state view | Approved | Locked rule: external/concurrent state changes must not interleave during a command tick. Internal state mutations from successful Navigate steps are committed between steps in the same tick and become the basis for subsequent step evaluation. |
| Conflicting-binding validation | Designer/runtime validation should error on duplicate direction-qualified Navigate bindings at same scope | Approved | Locked rule: verb/directional trigger combinations must be unique per scope. Existing scoped action authoring validation already enforces this; retain and cover with regression tests. |
| Minimum pre-go-forward test matrix | Require integration tests for explicit direction, no-direction single-exit infer, no-direction ambiguous fail, one-way behavior, lock/open transitions, and echo selection | Approved | Locked rule: matrix must include openable-bound traversal leg gating (closed blocks/open allows), shared paired-openable configuration validation (both sides required and distinct ids), and shared-state safety coverage (idempotent sync/no rebroadcast loop) in addition to command/action precedence and deterministic outcome assertions. |
