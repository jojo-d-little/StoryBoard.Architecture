# Base Object Ownership Lock-Off

Status: Active - Pending Closure (Tentative)
Owner: Team
Last updated: 2026-07-26

## 0. Tentative Close State

This plan is tentatively closed for implementation and remains in `plans/active` as pending closure.

Closure intent:

1. Keep this plan active during the next development window.
2. If no new issues are raised in this lock area, move to archive at the next planning checkpoint.
3. If issues are raised, reopen implementation tasks under this same plan before archival.

Exit criteria to archive:

1. No new ownership/override/effective-read regressions reported in ongoing development.
2. Required validation gates remain green when related changes are touched.
3. Team confirms no additional lock decisions are needed for v1 scope.

## 1. Purpose

Define an explicit, durable ownership contract for base objects versus linked room instances so editor/runtime behavior is predictable and drift-resistant.

This lock-off is the authoritative reference for:

1. Which fields are base-definition-owned.
2. Which fields are instance-owned.
3. Which fields can be locally overridden on an instance.
4. How effective values are resolved.
5. How legacy copied values are treated during migration.

## 2. Core Rule

Linked instances inherit definition-owned fields by dereferencing `DefinitionId`.

Linking is single-level in v1: base -> instance only. Linked instances must reference a non-linked base definition.

No broad push-sync propagation from base object edits to copied instance fields is used as the long-term model.

## 3. Ownership Matrix (Lock Proposal v1)

Legend:

1. Owner = canonical authoring owner.
2. Override Allowed = explicit local instance override is permitted.

| Field Group | Owner | Override Allowed | Notes |
|---|---|---|---|
| Definition identity (`ObjectType`) | Definition | No | Canonical type identity; non-overridable. |
| Display name (`Name`) | Definition | No | Stable type identity for linked instances; non-overridable. |
| In-game name (`NameInGame`) | Definition | Yes | Optional local narrative alias. |
| Name synonyms (`NameSynonyms`) | Definition | Yes | Optional local parser phrasing alias for linked instances. |
| Description | Definition | Yes | Optional local flavor text override. |
| Producer notes | Definition | No | Authoring metadata belongs to definition only. |
| Capability toggles/defaults (inventoriable/container/open/lock/activate/hide/composite/quantifiable) | Definition | No | Capability contract is type-level. |
| Action metadata (`Commands`, `AdditionalVerbs`, `AdditionalDirectionals`, traversal mappings) | Definition | No | Linked instances execute definition behavior unless explicitly detached. |
| Available actions | Definition | No | Linked instances do not own independent action lists. |
| Image variants (`ImageVariants`) | Definition | No | Canonical visual variant set belongs to base object. |
| Variant chooser script (`ImageVariantChooserScript`) | Definition | No | Chooser logic belongs to definition. |
| Visual/appearance feature fields (`ObjectHeightUnits`, `StackGroup`, footprint, heading defaults, stack scaling defaults) | Definition | No | Type-level visual behavior contract. |
| Placement transform (position, room placement, containment placement context) | Instance | No | Placement is instance context, not definition data. |
| Render order in room | Instance | No | Authoring order within room is instance context. |
| Quantity value (for quantifiable placement) | Instance | No | Count at placement point is instance context. |
| Definition link (`DefinitionId`/legacy `LinkedBaseObjectId`) | Instance | No | Mandatory for linked-instance projection. |
| Runtime/session mutable values (`isOpen`, `isLocked`, etc.) | Runtime session | N/A | Not canonical authoring ownership. |

## 4. Explicit Override Allowlist

Only these local instance overrides are allowed in v1:

1. `NameInGame`
2. `NameSynonyms`
3. `Description`

All other fields are inherited from the linked base definition.

## 5. Effective Value Resolution

For any linked instance and definition-owned field:

1. If field is in override allowlist and explicit override exists on instance, use instance value.
2. Otherwise resolve from direct base definition by `DefinitionId` (single-hop).
3. If definition cannot be resolved, design-time validation blocks write-through and offers Convert To Standalone repair.
4. Runtime remains non-throwing and emits diagnostics for unresolved base references; if compatible local fallback values exist, runtime may use them for safety.

## 6. Runtime and Export Behavior

1. Runtime uses effective values after definition dereference for definition-owned fields.
2. Clean export should not treat copied instance values as independent source-of-truth for definition-owned fields.
3. Linked instances may still persist placement data and explicit override payload.
4. Linked-instance payload minimization is required:
	1. Project JSON keeps link metadata, instance-owned fields, explicit overrides, and identity fields (`Name`, `ObjectType`) for compatibility.
	2. Project JSON omits other definition-owned copied payload on linked instances.
	3. Clean export JSON keeps `name` and link metadata, omits copied definition-owned linked-instance payload, and relies on runtime base dereference.
	4. Clean export schema v1 does not carry `objectType`.

## 7. Migration Rules (Legacy Copied Data)

1. Existing projects may contain copied values on linked instances for definition-owned fields.
2. During migration cutover, canonical ownership is enforced immediately: definition-owned truth is base-only.
3. One-time in-place migrations for samples/fixtures and ad hoc project data should remove redundant copied definition-owned payload from linked instances.
4. Do not preserve or continue emitting erroneous copied definition-owned payload after migration.

## 8. Non-Goals

1. No broad push-propagation mechanism for base edits.
2. No runtime host coupling to Designer types.
3. No export contract break without explicit version decision.

## 9. Signoff Checklist

Lock decisions are complete when all are marked accepted:

1. Accept ownership matrix in Section 3.
2. Accept override allowlist in Section 4.
3. Accept dereference-first resolution in Section 5.
4. Accept migration handling in Section 7.

## 10. Initial Validation Expectations

1. Base edit to `ImageVariants` updates all linked instances at runtime without instance edits.
2. Base edit to appearance fields (for example `ObjectHeightUnits`) updates linked-instance effective behavior.
3. Local instance overrides for `NameInGame`, `NameSynonyms`, and `Description` remain local while definition `Name` flows through to all linked instances.
4. Linked instance action editing remains definition-authoritative.

## 11. Runtime Read Architecture (Lock Proposal)

### 11.1 Goal

Keep base-awareness encapsulated inside runtime object instance reads so the rest of runtime code can read object properties without repeatedly handling base-object corner cases.

### 11.2 Design Shape

For linked instances in runtime:

1. Instance keeps its own placement/session state fields.
2. Instance keeps `DefinitionId` reference.
3. Instance resolves and stores direct base definition reference during session bootstrap.
4. Definition-owned fields are read through instance-level effective accessors.

This means call sites read effective values from the instance, not from ad hoc link-resolution logic spread across processors/executors.

### 11.3 Runtime Object Model Contract

Runtime object node provides two conceptual layers:

1. Local layer: placement-owned and local-override payload values.
2. Effective layer: definition-owned read surface with override precedence.

Resolution precedence for effective layer:

1. Allowed instance override (if present).
2. Direct base definition value.
3. Local fallback value (compatibility safety only).

### 11.4 Encapsulation Rule

Only runtime object instance internals and dedicated resolver helpers may follow definition links.

Runtime systems such as command processing, movement/stacking, variant selection, and rendering should consume effective object reads and remain unaware of base-link traversal.

### 11.5 Performance and Stability

1. Resolve direct base references once during bootstrap/session load.
2. Cache effective values per field group where appropriate.
3. Avoid repeated dictionary graph lookups in hot command paths.

### 11.6 Suggested Shared Runtime Components

1. `RuntimeObjectDefinitionResolver`: resolves and validates direct base references and single-level safety.
2. `RuntimeObjectEffectiveReadAccessor`: centralized effective value read logic for definition-owned fields.
3. `RuntimeObjectLinkDiagnostics`: optional diagnostics for missing-definition fallback events.

### 11.7 Field Groups for First Effective-Read Cutover

Phase R1 scope (highest drift risk first):

1. `ImageVariants`
2. `ImageVariantChooserScript`
3. Appearance feature fields used by movement/stacking/render prep

After R1 parity is proven, expand to remaining definition-owned fields.

## 12. Designer Write Architecture Boundary

Runtime read encapsulation does not imply designer write-through on linked instances.

Designer remains explicit about write intent:

1. Editing definition-owned field from linked instance route writes to owning base object.
2. Editing allowlisted local override writes to instance override payload.
3. Editing instance-owned placement fields writes to instance.

This keeps authoring intent clear while runtime reads stay simple and uniform.

## 13. Runtime-Dereference Locked Outcomes

1. Bootstrap-time base resolution is locked with single-level direct references only.
2. Runtime processors must not traverse base links directly after cutover.
3. R1 cutover scope is locked to `ImageVariants`, chooser script, and appearance fields used by movement/stacking/render prep.
4. Missing definition behavior is locked to deterministic diagnostics and non-throwing runtime behavior.
5. Design-time validation must catch missing base references and block write-through edits until repaired.

## 14. Designer Write-Through Locked Outcomes

1. Definition-owned edits from linked-instance surfaces write through to the owning base object.
2. Instance-owned fields always write local and never write-through.
3. Non-allowlisted fields cannot create local overrides.
4. UI remains lightweight: a dialog-level banner communicates linked/base impact.
5. No per-field ownership badges.
6. No per-session confirmation prompts.
7. Impact count and scope context are shown in the banner text.
8. Override behavior is implicit for allowlisted fields:
	1. If local value equals base, clear override.
	2. If local value differs from base, store override.
9. Missing-base behavior during edit uses a simple blocking warning with Convert To Standalone repair.
10. Write-through persistence is base-only; linked instances are not independently mutated in authoring data.
11. MVVM notifications must fan out to linked instances so UI redraws effective values immediately.
12. Rollback safety is required if validation fails.
13. Validation runs impacted-graph first, then full-project checkpoints.
14. Semantics must be identical across all entry points; any surface that cannot honor rules is read-only for linked base-owned fields.
15. Propagation model is full-definition impact for definition-owned fields only.
16. Propagation never overwrites instance-owned fields and never overwrites allowlisted local overrides.
17. Designer/runtime effective-read parity is a release gate.

## 15. Lock Meeting Worksheet

Use this worksheet during signoff sessions. Mark one status per row and capture owner/date/notes.

Status values:

1. `Accepted`
2. `Rejected`
3. `Deferred`

| Decision Item | Status | Owner | Date | Notes |
|---|---|---|---|---|
| Ownership matrix in Section 3 | Accepted | Team | 2026-07-26 | `Name` non-overridable; instance-owned placement/render/quantity locked. |
| Override allowlist in Section 4 | Accepted | Team | 2026-07-26 | `NameInGame`, `NameSynonyms`, `Description` only. |
| Effective resolution precedence in Section 5 | Accepted | Team | 2026-07-26 | Single-hop direct base resolution in v1. |
| Migration treatment for legacy copied values in Section 7 | Accepted | Team | 2026-07-26 | One-time cleanup; no ongoing preservation of erroneous copied payload. |
| Runtime bootstrap-time definition resolution (13.1) | Accepted | Team | 2026-07-26 | Single-level base->instance model. |
| Runtime no-direct-traversal rule after cutover (13.2) | Accepted | Team | 2026-07-26 | Processor code consumes effective reads only. |
| R1 runtime cutover scope (13.3) | Accepted | Team | 2026-07-26 | Visual/appearance R1 scope locked. |
| Missing-definition fallback plus diagnostics behavior (13.4) | Accepted | Team | 2026-07-26 | Design-time blocking + runtime diagnostics. |
| Designer write-through default for definition-owned fields (14.1.1) | Accepted | Team | 2026-07-26 | Base-owned fields write-through by default. |
| Mandatory write-target labeling in UI (14.1.2) | Accepted | Team | 2026-07-26 | Simple top banner, no heavy UI. |
| Impact count before write-through commit (14.2.1) | Accepted | Team | 2026-07-26 | Lightweight count text in banner. |
| One-time per-session confirmation policy (14.2.2) | Rejected | Team | 2026-07-26 | No confirmation prompts in v1. |
| Field ownership badges in editor UI (14.3.1) | Rejected | Team | 2026-07-26 | Avoid field-level UI complexity. |
| Inherit versus Override controls for allowlisted fields (14.3.2) | Rejected | Team | 2026-07-26 | Implicit override behavior only. |
| Atomic undo for write-through edits (14.4.1) | Accepted | Team | 2026-07-26 | Base-only persisted mutation; MVVM fan-out required. |
| Impacted-graph validation policy (14.5.2) | Accepted | Team | 2026-07-26 | Run impacted graph first, then full checkpoints. |
| Scope impact summary in confirmations (14.6.1) | Accepted | Team | 2026-07-26 | Banner-only scope summary; no extra confirmation UI. |
| Runtime/Designer effective-read parity gate (14.7.1) | Accepted | Team | 2026-07-26 | Required release gate. |

### 15.1 Final Lock Summary

1. Meeting date: 2026-07-26
2. Participants: Team design walkthrough
3. Accepted count: 15
4. Rejected count: 3
5. Deferred count: 0
6. Follow-up owner: Team
7. Follow-up due date: In implementation planning

### 15.2 Deferred Decisions Backlog

Track any deferred lock items here with explicit revisit dates.

| Deferred Item | Reason | Owner | Revisit Date | Exit Criteria |
|---|---|---|---|---|
| None | N/A | N/A | N/A | N/A |

## 16. Implementation Phases and Test Gates

### 16.1 Phased Sequence

1. Phase A: enforce ownership boundaries and single-level link validation.
2. Phase B: implement runtime/designer effective-read parity for R1 fields.
3. Phase C: implement write-through behavior, lightweight banner, and MVVM fan-out refresh.
4. Phase D: align clean export contract and apply one-time migration cleanup for redundant copied definition-owned payload.
5. Phase E: finalize guardrails and documentation polish.

### 16.2 Required Gate Between Every Phase

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`
4. Phase progression is blocked until all three pass.

### 16.3 Targeted New/Updated Test Coverage

1. Single-level link validation rejects chained links.
2. `Name` override is blocked for linked instances.
3. `NameInGame`, `NameSynonyms`, and `Description` override behavior is preserved.
4. Missing-base edit flow supports Convert To Standalone repair.
5. Runtime/designer parity for R1 effective reads.
6. Base edit MVVM fan-out refresh updates linked-instance UI projections.
