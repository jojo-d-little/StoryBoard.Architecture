# Improve Instance Object Handling Plan

Status: Near Complete (All non-item-6 work signed off; item 6 review intentionally deferred)
Owner: StoryboardDesigner.App model and editor workflows
Last updated: 2026-07-05

Review note (2026-07-05):

1. Plan remains open only for Section 22 item 6 review and final closeout signoff.
2. Active remainder tracking is mirrored in plans/future/CONSOLIDATED_OUTSTANDING_PLAN.md.

Recent progress (2026-07-05):

1. Added `Promote to Base Object` workflow for room objects, including transactional rollback behavior on failure.
2. Added regression tests covering action visibility, successful promotion conversion, rollback restoration, and area Base Objects tree discoverability after promotion.
3. Added scope-targeted promotion selection (Area/Country/Planet/Global) with catalog-aware insert and deterministic rollback coverage.
4. Expanded promotion test coverage for Country/Planet/Global target catalogs and selected-scope duplicate-name collision blocking.
5. Added promotion cancellation guardrail coverage to ensure no catalog mutation or instance conversion when scope selection is canceled.
6. Added hierarchy-level cancel-path assertion to ensure Base Objects tree nodes remain unchanged when scope selection is canceled.
7. Added scope picker contract coverage asserting Area -> Country -> Planet -> Global option order and labels.
8. Refreshed Birmingham playback snapshots via `UPDATE_PLAYBACK_SNAPSHOTS=1` and re-ran full test project successfully (382/382 passing).
9. Boundary Status: Designer-only (no Shared/Simulator impact).

## 1. Purpose

Improve how object instances are modeled so base-defined data has a single source of truth, linked-instance drift is eliminated, and editing behavior is predictable across designer, export, and runtime mapping.

## 2. Why This Plan Exists

Current linked-instance handling mixes relationship metadata with duplicated local fields.

Symptoms observed:

1. Rename propagation bugs when editing base definitions.
2. Risk of divergence between base and linked copies for base-owned fields.
3. Repeated one-off synchronization logic in viewmodel workflows.
4. Ambiguity about which fields are authoritative on linked instances.
5. Harder long-term maintenance for instanceable individual-instance workflows.

## 3. Terminology Workstream (Locked Outcome)

Terminology decisions are locked; this section captures final terms and historical options considered.

### 3.1 Terminology Goals

1. Avoid overloaded parent/child language (already used for scope tree structure).
2. Make ownership semantics obvious from names.
3. Make authoring and runtime concepts consistent.

### 3.2 Candidate Vocabulary Sets (Historical; Superseded by D-01)

Candidate Set A:

1. Object Type: canonical definition record.
2. Object Instance: placed/realized object record.
3. Type Reference: pointer from instance to type.
4. Instance Overrides: explicitly allowed per-instance deviations.

Candidate Set B:

1. Prototype Object: canonical definition record.
2. Realized Object: placed record.
3. Prototype Link: pointer from realized object to prototype.
4. Realized Overrides: explicit deviations.

Candidate Set C:

1. Base Object Definition: canonical definition record.
2. Referencing Instance: placed record.
3. Definition Reference: pointer from instance to base definition.
4. Instance Overrides: explicit deviations.

### 3.3 Locked Terminology for Implementation

Use these terms in implementation and docs:

1. Use Definition for canonical/base-owned fields.
2. Use Instance for placed records.
3. Use DefinitionId for link identity.
4. Use InstanceOverrides for allowed local differences.

### 3.4 Legacy-to-Target Terminology Mapping (Locked)

Target terminology for this initiative:

1. Object Type: canonical definition.
2. Base Object: the specific type record referenced by instances.
3. Object Instance: placed object record that references a base object.
4. Instanceable: object feature that allows multi-instance materialization/reuse semantics.
5. Quantifiable: complementary capability that controls quantity/count semantics.

Legacy compatibility mapping (current code/data names):

1. IsQuantifiable: retained as Quantifiable capability (no replacement by Instanceable).
2. QuantifiablePlacementDistributionMode: retained as Quantifiable-only behavior policy.
3. LinkedBaseObjectId -> DefinitionId (target name).

Note: keep legacy names readable only during time-bounded migration slices; final completion target removes legacy naming baggage after one-time sample-project migration.

## 4. Target Model Direction

### 4.1 Core Principle

Definition-owned fields are not duplicated as independent source-of-truth on linked instances.

### 4.2 Field Ownership Split (Locked Baseline)

Definition-owned (canonical):

1. Name and display name defaults.
2. Description defaults.
3. Capability toggles and defaults (inventoriable, container, instanceable, openable, lockable, activatable, hidable, composite-related defaults).
4. Action definitions and scope-level command metadata.
5. All object features are definition-owned.

Instance-owned:

1. Placement and containment context.
2. Definition reference id.
3. Explicit per-instance override payload (only allowlisted fields).
4. Runtime state variables (session-side, not authoring canonical data).
5. Game properties tied to the placed instance.
6. Room location and scope location context (exact room-location shape TBD).

## 5. Scope and Non-Goals

In scope:

1. Clarify model semantics for definition vs instance.
2. Reduce/retire drift-prone synchronization pathways.
3. Prioritize forward model clarity; provide one-time migration support for sample projects instead of broad long-term backward-compat complexity.
4. Keep simulator boundary intact (no Designer to Simulator coupling).

Out of scope (initial plan):

1. Full runtime command architecture redesign.
2. Unplanned clean-export contract version break.
3. UI redesign unrelated to definition/instance semantics.

## 6. Architecture Guardrails

1. Keep reusable logic in Storyboard.Shared only when host-agnostic.
2. Keep Designer-specific authoring model orchestration in StoryboardDesigner.App.
3. Keep Storyboard.Simulator independent of StoryboardDesigner.App.
4. Keep additive compatibility layers during migration; retire only after parity evidence.

### 6.1 Default Boundary for This Plan

This initiative is treated as Designer-authoring UX and model semantics work first.

Default implementation target:

1. StoryboardDesigner.App only.
2. No required code changes in Storyboard.Shared or Storyboard.Simulator unless explicitly approved.

### 6.2 Escalation and Signoff Gate for Shared/Simulator Impact

If implementation uncovers a need to modify Storyboard.Shared or Storyboard.Simulator, execution must pause for explicit visibility and signoff.

Required escalation artifact before any cross-boundary change:

1. Impact summary: what cannot be solved in Designer-only scope.
2. Alternatives considered: Designer-only options and why insufficient.
3. Proposed Shared/Simulator change scope: exact files, contracts, and behavior impact.
4. Risk assessment: runtime compatibility, simulator behavior, export contract risk.
5. Validation plan: focused tests and regression gates to run.

Approval rule:

1. No Shared/Simulator edits proceed until signoff is recorded in plan notes for that slice.

### 6.3 Required Visibility Note per Slice

Each implementation slice should include a one-line boundary status entry:

1. Boundary Status: Designer-only (no Shared/Simulator impact), or
2. Boundary Status: Escalated (Shared/Simulator signoff required and linked).

## 7. Migration Strategy

Slice status update:

1. Slice T1 is complete (decision lock finalized).
2. Slices T2-T5 are complete and validated.
3. Remaining work for final completion is limited to Section 22 addendum review (item 6), intentionally deferred until all other work is signed off.

### Slice T1: Terminology and Ownership Lock

1. Finalize naming set for definition/instance concepts.
2. Produce field ownership matrix for GameObject-related data.
3. Decide allowed override fields and override precedence.
4. Define invariants for linked/instance records.

Exit criteria:

1. Terminology locked.
2. Ownership matrix approved.
3. Override policy approved.

### Slice T2: Model Scaffolding (Additive)

1. Add explicit definition-reference and override model structures.
2. Add effective-value accessors that resolve definition plus overrides.
3. Keep existing fields readable for compatibility during transition.

Exit criteria:

1. Effective-value paths available.
2. Existing project load/save still functional.

### Slice T3: Editor Workflow Cutover

1. Update object settings and rename flows to use definition-authoritative updates.
2. Restrict or hide definition-owned field editing on instance records.
3. Ensure quantity/instanceable object creation flows create valid instance references.

Exit criteria:

1. No manual propagation hacks required for definition-owned fields.
2. Dialog and inline edit behavior consistent with ownership policy.

### Slice T4: Serialization and Mapping Alignment

1. Update project serialization mapping for explicit definition plus instance semantics.
2. Provide one-time migration tooling/notes for sample projects and keep compatibility handling minimal and time-bounded.
3. Ensure runtime snapshot mapping consumes effective values deterministically.

Exit criteria:

1. Old projects load with equivalent behavior.
2. New saves preserve intended definition/instance separation.

### Slice T5: Validation and Guardrails

1. Add validation rules enforcing illegal override and orphan-definition conditions.
2. Add architecture and behavior guardrail tests for definition-instance invariants.
3. Remove obsolete synchronization code paths.

Exit criteria:

1. Guardrails prevent reintroduction of drift behavior.
2. Regression suite green.

## 8. Key Decisions (Locked)

1. Final terminology set.
2. Exact field ownership list.
3. Allowed instance override fields.
4. ObjectType policy: first-class field, definition-authoritative, never instance-overridden.
5. Name policy: defaults from definition at creation, but may be locally overridden on instances.
6. Whether NameInGame can be an override.
7. How templates, player objects, and room objects share definition identity rules.
8. How instanceable individual instances map to definition plus instance records.

## 9. Risks and Mitigations

Risk: migration complexity and compatibility regressions.
Mitigation: additive rollout, compatibility readers, focused parity tests.

Risk: ambiguous partial ownership for some fields.
Mitigation: strict ownership matrix and validation rule enforcement.

Risk: accidental runtime/designer coupling.
Mitigation: keep model semantics host-local; preserve separation guardrail tests.

## 10. Validation Plan

Default gates per meaningful slice:

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"

Additional targeted tests to add/maintain:

1. Definition rename propagates to all referencing instances.
2. Definition-owned field edits are reflected through effective-value resolution without duplicated writes.
3. Illegal instance override attempts are blocked or validated.
4. Legacy linked-object project load maps to new model semantics without behavior drift.
5. Instanceable individual-instance workflows preserve expected authoring and runtime behavior.
6. ObjectType remains definition-authoritative on instances, while local instance Name overrides (when enabled) remain instance-scoped.
7. ObjectType is globally unique across the project and duplicate create/edit attempts are blocked with clear validation errors.
8. self.nameInGame token is available in echo message script editors and resolves to effective NameInGame (instance override first, then definition fallback).
9. self.description token resolves to effective Description (instance override first, then definition fallback).
10. Composite message script token/value resolution should use the same effective-value resolver so inherited/overridden data behavior comes nearly for free from the core model.

## 11. Deliverables

1. Terminology decision record.
2. Field ownership matrix document.
3. Model and accessor implementation slices.
4. Migration/compatibility mapping notes.
5. Regression and guardrail test additions.

## 12. Immediate Next Step

All non-item-6 scope has been completed and signed off with current validation evidence.

1. Keep this plan in near-complete state while Section 22 review remains deferred by request.
2. When ready, run Section 22 review as the final completion gate and record keep/refactor decisions for listed composite fields.

## 13. Field Ownership Matrix (Locked Baseline)

Legend:

1. Owner: Definition means canonical object type data; Instance means placement-specific data.
2. OverrideAllowed: Yes means instance may override definition value via explicit override payload; No means definition-authoritative only.
3. This matrix is draft input for Slice T1 lock.
4. Object features and action definitions are definition-owned by default.
5. Game properties and room/scope location context are instance-owned by default.

| Field | Owner (Draft) | OverrideAllowed (Draft) | Migration Notes |
|---|---|---|---|
| ObjectType | Definition | No | First-class canonical type identity; defaulted at creation and never instance-overridden. |
| Name | Definition | Yes (planned local override) | Create-time default mirrors ObjectType; instances may later set local Name override without changing definition ObjectType. |
| NameInGame | Definition | Yes (candidate) | Candidate override only if gameplay requires per-placement aliasing; keep open for T1 lock. |
| Description | Definition | Yes (candidate) | Candidate for environmental flavor overrides; otherwise keep definition-only for simplicity. |
| ProducerNotes | Definition | No | Treat as authoring metadata on definition, not instance state. |
| IsInventoriable | Definition | No | Capability belongs to definition contract. |
| InventoryPointsDefaultValue | Definition | No | Capability default should not drift by instance. |
| IsContainer | Definition | No | Capability belongs to definition. |
| ContainerPointsDefaultValue | Definition | No | Definition-owned default capacity setting. |
| IsCapacityPointShareDividerEnabled | Definition | No | Capability toggle tied to definition feature contract. |
| CapacityPointShareDividerDefaultValue | Definition | No | Definition-owned default behavior. |
| IsOpenable | Definition | No | Definition capability. |
| IsOpenDefaultValue | Definition | No | Definition default, runtime/session carries actual state. |
| IsLockable | Definition | No | Definition capability. |
| IsLockedDefaultValue | Definition | No | Definition default, runtime/session carries actual state. |
| IsActivatable | Definition | No | Definition capability. |
| IsActiveDefaultValue | Definition | No | Definition default, runtime/session carries actual state. |
| IsHidable | Definition | No | Definition capability. |
| IsHiddenDefaultValue | Definition | No | Definition default, runtime/session carries actual state. |
| IsInstanceable (new feature) | Definition | No | Definition feature for type reuse and multi-instance placement semantics. |
| IsQuantifiable (retained capability) | Definition | No | Complementary definition capability for quantity/count behavior. |
| Quantity | Instance | No (definition override path separate) | For grouped-stack authoring, instance quantity is placement count/value; for individual instances quantity is typically 1 each. |
| QuantifiablePlacementDistributionMode (retained) | Definition | No | Quantifiable-only distribution strategy should be definition policy to avoid mixed semantics. |
| IsCompositeTarget | Definition | No | Composite role belongs to definition. |
| IsCompositeReversible | Definition | No | Composite behavior policy belongs to definition. |
| CompositePartRequirementMode | Definition | No | Definition behavior policy. |
| CompositeMinimumRequiredPartCount | Definition | No | Definition behavior policy. |
| CompositeBuildSuccessMessage | Definition | Yes (candidate) | Candidate override if room-specific narrative is needed; otherwise definition-owned only. |
| CompositeBuildFailureMessage | Definition | Yes (candidate) | Candidate override if narrative variance is desired. |
| CompositeBreakSuccessMessage | Definition | Yes (candidate) | Candidate override if narrative variance is desired. |
| CompositeBreakFailureMessage | Definition | Yes (candidate) | Candidate override if narrative variance is desired. |
| Commands | Definition | No | Authoring command tokens should be type-level by default. |
| Variables (definition defaults/features) | Definition | No (for schema/defaults) | Split needed: definition variable schema/defaults vs runtime session values. |
| AdditionalVerbs | Definition | No | Definition command vocabulary surface. |
| AdditionalDirectionals | Definition | No | Definition directional vocabulary surface. |
| AdditionalDirectionalTraversalMappings | Definition | No | Definition command mapping surface. |
| AvailableActions | Definition | No | Action set should be authored at definition level. |
| CompositeRecipeId | Definition | No | Recipe identity belongs to definition. |
| CompositeRequiredParts | Definition | No | Recipe requirements belong to definition. |
| ContainedObjects | Instance | No | Containment is placement/runtime tree structure, not definition canonical data. |
| LinkedBaseObjectId (to-be-renamed DefinitionId) | Instance | No | Keep on instance as mandatory reference when instance derives from definition. |
| LinkActionsToBaseObject (legacy) | Instance | No | Legacy transitional flag; candidate deprecation once definition ownership is explicit. |
| ObjectId | Instance | No | Unique identity per record; definition and instance ids should be distinct concepts. |

### 13.1 Follow-up Split Needed for Variables

Current Variables carries multiple concerns at once. Draft target split:

1. DefinitionVariableSchema: variable names, restrictions, default values, feature-contract members.
2. InstanceVariableOverrides: optional, allowlisted authored deviations only (if enabled).
3. RuntimeSessionValues: game-state values at runtime, outside authoring canonical data.

### 13.2 Override Allowlist (Locked for Current Plan)

Locked allowlist for instance-level overrides:

1. Name (local instance override; ObjectType remains non-overridable).
2. NameInGame.
3. Description.
4. CompositeBuildSuccessMessage.
5. CompositeBuildFailureMessage.
6. CompositeBreakSuccessMessage.
7. CompositeBreakFailureMessage.

Everything else defaults to definition-authoritative unless explicitly approved by a later lock decision.

## 14. MVVM Design Pattern for Definition-Owned Fields

This section captures the preferred MVVM approach for definition-owned fields (including ObjectType) and Name display behavior from instance nodes in the tree.

### 14.1 Core MVVM Rule

1. Tree nodes bind to effective display properties, not raw storage fields on linked instances.
2. Linked instances do not own definition-owned fields such as ObjectType.
3. Name displayed in tree resolves from local instance Name override when present; otherwise from definition default Name.
4. Edit commands route by ownership policy: ObjectType edits route to definition owner; local Name edits stay on instance.

### 14.2 ViewModel Contract (Draft)

Each object tree node viewmodel should expose:

1. DisplayName: effective name used by tree rendering.
2. IsLinkedInstance: whether this node is an instance projection.
3. DefinitionId: resolved owning definition identity (when linked).
4. CanEditDefinitionOwnedFields: false for linked instance nodes.
5. CanEditObjectType: false for linked instance nodes.
6. RenameCommand: updates local instance Name when editing an instance node, and updates definition Name when editing a definition node.
7. CanEditActions: false for linked instance nodes.
8. GoToBaseObjectCommand: navigates from linked instance context to owning base object editor.

Optional clarity properties:

1. NameOwnershipLabel (Definition or Instance).
2. LinkBadgeText (for example Derived Instance).

### 14.3 Change Notification Strategy

When a node projects data from a definition:

1. Node subscribes to PropertyChanged events from owning definition (or receives domain event bus notifications).
2. On definition Name change, node raises PropertyChanged for DisplayName.
3. Subscriptions must be weak or explicitly disposed to avoid leaks during hierarchy rebuilds.
4. Rebuild/teardown paths must unregister subscriptions deterministically.

### 14.4 Command Routing and Edit Semantics

Rename flow for linked instance node:

1. User edits tree node name.
2. Node command writes local Name override on the instance (when local-name override is enabled).
3. ObjectType remains unchanged and non-editable on the instance.
4. Validation and uniqueness checks execute in instance owner scope for local-name updates.
5. Definition rename remains a separate definition-level edit path; subscribed nodes without local-name override refresh DisplayName automatically.

Direct instance writes to definition-owned fields (including ObjectType) are disallowed.

### 14.5 Service Boundary Recommendation

Introduce one resolver/service for effective values and ownership decisions so all UI surfaces stay consistent:

1. EffectiveObjectFieldResolver (definition plus overrides projection).
2. ObjectOwnershipPolicy (which fields are definition-owned vs instance-owned).
3. RenameRoutingService (local instance Name vs definition Name based on edit target).

Tree viewmodels should consume these services rather than duplicating ownership logic.

### 14.6 XAML Binding Guidance

1. Bind tree text to DisplayName (effective projection), not raw model Name.
2. Disable inline edit for ObjectType on linked instance nodes.
3. Hide or disable action editing surfaces on linked instance nodes; prefer a clear Go to Base Object affordance.
4. Keep tooltip/badge indicators for linked-instance context to reduce author confusion.

### 14.7 Test Expectations for MVVM Behavior

Add/maintain tests proving:

1. Linked instance tree nodes display local instance Name override when present; otherwise definition Name.
2. Definition rename triggers PropertyChanged and updates linked node DisplayName values that do not have local-name overrides.
3. Rename command invoked from a linked instance node updates instance local Name only; ObjectType remains unchanged.
4. ObjectType edits are blocked on linked instance nodes and route to definition editor path.
5. Action editing is unavailable on linked instance nodes and Go to Base Object navigation is available.
6. Subscription teardown during hierarchy rebuild does not leak or double-fire updates.

### 14.8 Migration Note

Current direct assignment patterns (for example setting node editable name directly on model instances) should be migrated to the effective-value projection and routed-command pattern above, slice by slice, to avoid broad disruptive rewrite.

## 15. Global Object Concept Model (Critical Design Clarification)

This section defines three distinct global-level concepts that must remain intentionally separate.

### 15.1 Global Game Objects (Existing; Keep)

Definition:

1. Full game objects that are active in the game world (for example player-scoped/global objects).
2. They participate directly in runtime scope/state.

Rules:

1. They are not mere design blueprints.
2. They can have actions, variables, and runtime behavior as active entities.

### 15.2 Object Templates (Existing; Keep)

Definition:

1. Designer scaffolding used to bootstrap newly created objects.
2. Not active game entities.

Rules:

1. Templates are authoring accelerators only.
2. Templates do not exist as runtime entities unless instantiated into actual objects by authoring workflows.
3. Templates must be available as bootstrap sources when creating new Base Objects.

### 15.3 Base Objects (New Concept)

Definition:

1. Canonical object type definitions.
2. Full object definitions, but not active in-game by themselves.
3. They become active only through object instances placed in rooms (or other allowed scopes).
4. They may be defined at multiple classification scopes to support theme-based catalogs.

Rules:

1. Base objects define canonical behavior/data for instance projection.
2. Base objects are first-class authoring objects but zero-presence runtime entities until instantiated.
3. Instance records reference a base object via DefinitionId.

### 15.4 Scoped Base Object Catalogs (New)

Base object definitions are allowed at major classification scopes:

1. Global Base Objects.
2. Planet Base Objects.
3. Country Base Objects.
4. Area Base Objects.

Design intent:

1. Enable theme catalogs at each organizational layer.
2. Keep reusable definitions close to the producer's world-building context.
3. Preserve discoverability of where a definition is owned.

## 16. Instanceable vs Quantifiable (Both Supported; Different Purposes)

These concepts are related but distinct and must not be conflated.

### 16.1 Instanceable (Type to Many Instances)

Definition:

1. Object feature that a base object type can have many placed object instances across the game.

Examples:

1. Dungeon Door base object can be instantiated into many rooms.

Semantics:

1. About type reuse and placement multiplicity.
2. Not inherently about stack math.

### 16.2 Quantifiable (Stack/Count Semantics)

Definition:

1. Capability that object quantities can be represented as stacked counts or individual placements by distribution mode.

Examples:

1. Flowers, coins, frog hair.

Semantics:

1. About quantity/count behavior.
2. Applies to countable/stackable item behavior and related script tokens.
3. nearByQuantity remains quantifiable-only unless explicitly redesigned.

### 16.3 Relationship

1. An object can be Instanceable without being Quantifiable.
2. An object can be Quantifiable without broad type-reuse expectations.
3. An object can support both when design requires both behaviors.

## 17. Data Modeling Guidance for New Global Base Objects

This section is a draft contract for intentional implementation.

### 17.1 Required Structural Additions (Draft)

1. Add BaseObjects collections at Global, Planet, Country, and Area scopes.
2. Add explicit DefinitionId on instance records that derive from base objects.
3. Keep legacy LinkedBaseObjectId readable during migration; map to DefinitionId.

### 17.1.1 Scoped Definition Identity

Each base object definition should carry:

1. DefinitionId (stable unique id).
2. DefinitionScopeKind (Global, Planet, Country, Area).
3. DefinitionScopeOwnerId (project or planet/country/area owner id).

This makes ownership and edit location explicit for tooling and navigation.

### 17.2 Runtime/Authoring State Separation

1. Base objects hold canonical authoring definitions.
2. Instances hold placement context and allowed instance overrides.
3. Runtime session state remains external to authoring definition records.

### 17.3 Tree View Expectations

Global node should represent three distinct groups clearly:

1. Game Objects: active global entities.
2. Object Templates: design-time scaffolding only.
3. Base Objects: canonical types that must be instantiated into rooms/scopes to become active.

Planet/Country/Area nodes should also expose Base Objects groups for scoped theme catalogs.

1. Planet Base Objects: visible to all descendants in that planet.
2. Country Base Objects: visible to all descendants in that country.
3. Area Base Objects: visible to rooms in that area.

### 17.4 Authoring Workflow Expectations

1. Producers can create/edit base objects from global Base Objects.
2. Room workflows allow Add Instance of Base Object.
3. Instance edit dialogs route definition-owned fields back to base object editor context.
4. Instance-level edits are limited to explicitly allowlisted instance-owned fields.
5. Creating a new Base Object supports Choose Object Template as initial bootstrap, matching existing new-object creation UX.

Scope-aware authoring additions:

1. Producers can create base objects at Global, Planet, Country, or Area.
2. Producers can promote local objects directly into the nearest intended base catalog scope.
3. Producers can move a base definition only upward between scopes via explicit Re-scope workflow with validation (Area -> Country -> Planet -> Global).

### 17.4.1 Scoped Visibility Rule

When adding an instance in a target room, visible candidate definitions are:

1. Area Base Objects (same area).
2. Country Base Objects (same country).
3. Planet Base Objects (same planet).
4. Global Base Objects.

Precedence rule for duplicate names:

1. Nearest scope wins for default selection/sorting.
2. UI must still show source scope badge to avoid ambiguity.
3. Producers may explicitly choose a farther visible scope candidate (for example Country or Planet) via a Look Further/Show Wider Scope option.
4. Nearest scope is strong guidance, not a forced selection.
5. Same display Name (for example Standard Door) may exist at multiple scopes; ObjectType must still be globally unique per D-05.

### 17.4.2 New Base Object Creation Flow (Template Bootstrap)

When creating a new Base Object definition:

1. Offer the same template selection step used by existing Add New Object workflows.
2. If template selected, clone template-authored defaults/behaviors into the new base definition.
3. If no template selected, create from default empty object baseline.
4. After creation, open Base Object editor on the new definition.

Consistency rule:

1. Base object creation and regular object creation should share bootstrap mechanics to reduce producer relearning.

### 17.5 Validation Rules to Add

1. Every instance with DefinitionId must resolve to an existing base object definition.
2. No definition-owned field drift allowed on instances.
3. Quantifiable-only features (including nearByQuantity semantics) require quantifiable capability.
4. Template objects cannot be referenced as runtime DefinitionId targets unless explicitly promoted.
5. ObjectType must be globally unique across the project; enforce uniqueness at create/edit time and via validation rules.

### 17.6 Resolved Policy Snapshot

1. Placement is room-only in this phase.
2. Global objects do not directly reference base definitions in this phase.
3. Templates are create-time bootstrap only.
4. Actions are definition-owned and projected live to instances (never instance-overridden).
5. Ancestor-scope browsing is allowed with local-first guidance.
6. Same Name may appear across scopes; ObjectType remains globally unique.

## 18. Placement UX Mechanics (Menu and Candidate Resolution)

This section defines the intended shift from quantifiable-specific placement actions to a unified instance workflow.

### 18.1 Context Menu Actions

Replace current split options with:

1. Add New Object
2. Add Instance of Existing Object

Design intent:

1. Add New Object creates a new definition and first placed object (or first instance, depending on scope policy).
2. Add Instance of Existing Object always creates a new placed instance from an existing object definition source.
3. Do not expose separate menu entries for quantifiable vs non-quantifiable instance placement.

### 18.2 Unified Instance Candidate List

When user chooses Add Instance of Existing Object, candidate list should be populated from definition sources in deterministic order.

Candidate source groups (in order):

1. Scoped Base Objects visible from target context (Area, Country, Planet, Global precedence).

Selection behavior:

1. Default suggestion should be the nearest-scope candidate when names collide.
2. User can intentionally browse ancestor scope catalogs (Area, Country, Planet, Global) and select farther visible scope candidates through explicit UI affordance.
3. Candidate rows must always show source scope badges to keep cross-scope choice explicit.
4. Candidate rows should show ObjectType and scope path when Name collisions exist across scopes.
5. Placement selection persists by DefinitionId (stable identity), not by Name.

Exclude by default:

1. Object Templates (unless explicitly promoted into a base object definition).
2. Existing instance records (instances should not be direct candidate sources).
3. Objects flagged non-instanceable when instanceability gating is enabled.

### 18.3 Candidate Eligibility Rules

A candidate is eligible for Add Instance of Existing Object only when:

1. It represents a definition source (not a placed instance).
2. It passes scope placement constraints for the target location.
3. It is not archived/disabled/deprecated.
4. It satisfies policy checks for object category compatibility (if category constraints are added later).

### 18.4 Candidate Display Contract

Display each candidate with enough context to avoid collisions:

1. DisplayName (effective definition name).
2. SourceKind badge: Base Object (Global, Planet, Country, Area).
3. Capability tags: Instanceable, Quantifiable, Container, etc.
4. Optional preview of key defaults (for example distribution mode when quantifiable).

### 18.5 Placement Behavior After Selection

After selecting candidate:

1. Create new placed instance with DefinitionId reference to candidate definition.
2. Initialize instance-owned fields only.
3. Set ObjectType from definition as non-editable definition-owned value.
4. Initialize Name from definition default/ObjectType; allow later local Name override per ownership policy.
5. Do not copy definition-owned fields as independent source-of-truth values.
6. If quantifiable and placement mode requires quantity input, ask for quantity in same flow.

Note:

1. Template choice occurs when creating new definitions (including Base Objects), not during Add Instance of Existing Object placement.

### 18.6 Quantifiable Interaction in Unified Flow

Quantifiable behavior remains in the same Add Instance flow:

1. If selected definition is quantifiable, show quantity/distribution options as a step in placement.
2. If selected definition is not quantifiable, skip quantity/distribution step.
3. nearByQuantity and other quantifiable-only runtime semantics continue to apply only when quantifiable capability is true.

### 18.7 Migration Staging for Menu Changes

1. Phase M1: keep old command id available internally, map UI label to Add Instance of Existing Object.
2. Phase M2: switch candidate resolver to unified definition-source pipeline.
3. Phase M3: remove legacy quantifiable-specific menu command once migration readers and telemetry indicate no dependency.

### 18.8 Validation and Tests for Placement Mechanics

Add/maintain tests proving:

1. Menu exposes exactly Add New Object and Add Instance of Existing Object at target scopes.
2. Candidate list excludes templates and placed instances.
3. Candidate list includes scoped Base Objects only.
4. Quantifiable options appear only when selected definition is quantifiable.
5. Created instance references definition id and does not persist definition-owned field duplicates as authoritative values.
6. Created instance receives ObjectType from definition and cannot edit ObjectType locally.
7. Local instance Name edit does not change definition Name or ObjectType.
8. One-time migration utility updates sample projects to base-object DefinitionId sourcing and reports unresolved legacy references.

Additional tests for base creation bootstrap:

1. Creating Base Object with template applies template defaults/actions/variables.
2. Creating Base Object without template uses standard empty baseline defaults.
3. Template-origin metadata (if tracked) is preserved consistently with existing new-object flow.

## 19. Locked Policy: Instance Sources and Promotion Workflow

This section captures locked policy to reduce producer confusion and improve edit discoverability.

### 19.1 Locked Decision

For Add Instance of Existing Object, allow selection from Base Objects only.

Rationale:

1. Producers always know where canonical definitions live.
2. Edit path is unambiguous: update the base object definition.
3. Avoids hidden definition ownership in arbitrary room/global locations.
4. Reduces support and onboarding friction.
5. Supports thematic catalogs by scope (global/planet/country/area) with predictable ownership.

Tradeoff:

1. This is an intentional restriction compared to theoretical any-object sourcing.
2. Restriction is offset by a strong Promote to Base Object workflow.

### 19.2 Promote to Base Object Workflow

Add context action on eligible placed objects:

1. Promote to Base Object.

Eligibility:

1. Object is not already a base-backed instance.
2. Object has valid name and definition data.
3. Object does not violate base object naming constraints.

Promotion operation does two required actions atomically:

1. Create new Base Object definition from the selected object definition-owned fields.
2. Convert original selected object into an instance referencing the new base DefinitionId.

Optional follow-up action:

1. Offer to find and relink similar objects to the new base object in same scope or project-wide.

### 19.3 Field Handling During Promotion

Definition-owned fields:

1. Moved to new base object definition.

Instance-owned fields:

1. Remain on original placed object after conversion.

Quantifiable fields:

1. Preserve quantifiable capability according to ownership matrix.
2. Preserve instance quantity semantics on converted instance.

### 19.4 UX Expectations

1. After promotion, tree selection remains on original placed object now marked as instance.
2. UI shows clear indicator linking instance to new base object.
3. One-click navigation from instance to base definition editor.
4. Base Objects group highlights the newly created definition.

### 19.5 Failure and Safety Rules

1. Promotion is transactional: if base creation or conversion fails, rollback all changes.
2. Duplicate name conflicts in Base Objects must be handled via rename prompt before commit.
3. Validation errors should surface before final apply with actionable fixes.

### 19.6 Tests for Promotion Decision

Add/maintain tests proving:

1. Add Instance of Existing Object candidate list contains Base Objects only.
2. Promote to Base Object creates base definition and converts original object to instance in one operation.
3. Converted instance references new DefinitionId and resolves effective definition-owned fields from base.
4. Promotion rollback leaves project unchanged on failure.
5. Producer can navigate from converted instance to base definition node.

## 20. Final Signoff Reminder (Root Issue Traceability)

Before marking this plan complete, signoff must explicitly confirm closure of the issue that triggered this initiative.

Required signoff checks:

1. Original root issue regression check:
	Renaming the root object of a quantifiable group works correctly and no longer leaves linked/group members out of sync.
2. Added propagation regression check:
	Renaming a base object updates all of its instances consistently across visible scopes.

Signoff note requirement:

1. Final review entry must include explicit pass/fail evidence for both checks above (tests and/or reproducible manual verification steps).

### 20.1 Pre-Signoff Evidence Snapshot (2026-07-04)

1. Root issue regression check status: Pass (automated).
	1. Evidence command: `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~QuantifiableRenamePropagationTests|FullyQualifiedName~RoomTreeTraversalProjectionTests"`
	2. Result: 15/15 passing.
2. Added propagation regression check status: Pass (automated).
	1. Covered in the same focused suite above, including linked-instance rename propagation and effective tree projection assertions.
3. Supporting safety gate:
	1. `dotnet build .\StoryboardDesigner.slnx` passed.

### 20.2 Final Review Entry (2026-07-04)

Final signoff decision for Section 20 checks:

1. Original root issue regression check: Pass.
	1. Evidence: `QuantifiableRenamePropagationTests` included in focused run:
	`dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~QuantifiableRenamePropagationTests|FullyQualifiedName~RoomTreeTraversalProjectionTests"` -> 15/15 passed.
2. Added propagation regression check: Pass.
	1. Evidence: linked-instance rename propagation and effective projection assertions covered in the same focused run above -> 15/15 passed.
3. Supporting confidence evidence: Pass.
	1. Runtime-focused guardrail suite passed (18/18) after playback snapshot refresh.
	2. Full test suite passed (368/368).
	3. `dotnet build .\StoryboardDesigner.slnx` passed.

## 21. Decision Lock Register (Consolidated)

Purpose:

1. This is the master list of decisions that must be explicitly locked before or during implementation slices.
2. Each decision should be marked Locked with date and rationale when approved.

Legend:

1. Timing: T1 means lock in Slice T1 unless otherwise noted.
2. Status values: Open, Locked, Deferred.

| Decision ID | Decision to Lock | Timing | Status |
|---|---|---|---|
| D-01 | Final terminology set for Object Type, Base Object, Object Instance, DefinitionId, ObjectType (first-class field), Instanceable, Quantifiable; update solution terminology document to match locked terms. | T1 | Locked (2026-07-04) |
| D-02 | Naming migration strategy: IsInstanceable is new; IsQuantifiable and QuantifiablePlacementDistributionMode are retained; LinkedBaseObjectId maps to DefinitionId during a time-bounded migration window with one-time sample-project migration support and no legacy naming baggage at final completion. | T1 | Locked (2026-07-04) |
| D-03 | Final field ownership matrix (definition-owned vs instance-owned). | T1 | Locked (2026-07-04) |
| D-04 | Override allowlist: Name (local), NameInGame, Description, and composite message fields are instance-overridable; all other definition-owned fields are not. | T1 | Locked (2026-07-04) |
| D-05 | ObjectType/Name policy: ObjectType is definition-authoritative, non-overridable, and globally unique across the project (create/edit validation enforced); Name defaults from definition/ObjectType and may be locally overridden per instance. | T1 | Locked (2026-07-04) |
| D-06 | NameInGame override constraints: instance-overridable, optional/blank allowed, no project-wide uniqueness requirement, and self.nameInGame available in echo message script editors. | T1 | Locked (2026-07-04) |
| D-07 | Description override constraints: instance-overridable via inherit-then-override semantics, optional/blank allowed, no uniqueness requirement, and self.description resolves effective value (instance first, then definition). | T1 | Locked (2026-07-04) |
| D-08 | Composite message override constraints: per-field instance overrides with inherit-then-override semantics, optional/blank allowed, and script message value resolution must consume the same effective-value resolver as core data. | T1 | Locked (2026-07-04) |
| D-09 | Variable split model contract: DefinitionVariableSchema definition-owned, InstanceVariableOverrides sparse copy-on-write, RuntimeSessionValues runtime-only, and effective variable resolution follows inherit-then-override. | T1 | Locked (2026-07-04) |
| D-10 | Scope catalog policy: Base Objects allowed at Global, Planet, Country, and Area only (not Room or instance scope); visibility flows to descendants. | T1 | Locked (2026-07-04) |
| D-11 | Scoped visibility rule for instance placement: ancestor scope catalogs (Area, Country, Planet, Global) are browseable from room placement with local-first guidance; producer may explicitly choose farther visible scopes (look-further flow), and same Name may appear across scopes while ObjectType remains globally unique. | T1 | Locked (2026-07-04) |
| D-12 | Name conflict policy: same Name may exist across scopes; nearest scope is default suggestion, picker shows scope/path and ObjectType for disambiguation, and selection persists by DefinitionId. | T1 | Locked (2026-07-04) |
| D-13 | Add Instance source policy: Base Objects only; no legacy roots in normal picker UX. Provide one-time migration support for sample projects to move legacy link patterns to DefinitionId-based base sourcing. | T1 | Locked (2026-07-04) |
| D-14 | Promote to Base Object is fully transactional with rollback on failure; on success original object must reference new DefinitionId, and optional relink-similar runs only after successful commit with explicit preview/confirm. | T1 | Locked (2026-07-04) |
| D-15 | Re-scope Base Object workflow policy: upward-only moves (Area -> Country -> Planet -> Global), with validation/collision checks and no downward re-scope. | T2 | Locked (2026-07-04) |
| D-16 | Creation semantics for instances: projection-first with sparse copy-on-write overrides (reset-to-inherit supported), not copy-on-create. | T2 | Locked (2026-07-04) |
| D-17 | Action model semantics: 100% definition-owned, never overridable by instances, projected live from base (no copy-on-create). Linked instance UI should not expose action editing and should provide Go to Base Object navigation. | T2 | Locked (2026-07-04) |
| D-18 | Base Object creation bootstrap uses the same template step as Add New Object; template data is copied once at creation (no live template linkage), and no-template path creates from empty baseline. | T1 | Locked (2026-07-04) |
| D-19 | Placement scope policy: base-object instances are addable to rooms only in this phase (no direct global/player placement). | T1 | Locked (2026-07-04) |
| D-20 | Relationship between Global Game Objects and Base Objects: no direct global-object base-definition references in this phase; revisit after stabilization gates pass. | T1 | Locked (2026-07-04) |
| D-21 | Template relationship policy: templates are create-time bootstrap only; no live inheritance/composition linkage with Base Objects after creation. | T1 | Locked (2026-07-04) |
| D-22 | MVVM subscription policy: deterministic attach/detach lifecycle disposal is required; weak events optional where practical; guardrail tests must verify no leaks and no duplicate notifications after rebuilds. | T2 | Locked (2026-07-04) |
| D-23 | Shared/Simulator boundary gate: Designer-only default; each slice records boundary status, and any Shared/Simulator impact requires explicit escalation artifact and signoff before edits. | Every slice | Locked (2026-07-04) |
| D-24 | Final signoff evidence pack is mandatory: explicit pass/fail evidence for root quantifiable rename regression and base-to-instance rename propagation, including required test outputs and reproducible manual verification for uncovered UX paths; no final signoff if either check is missing/inconclusive. | Final signoff | Locked (2026-07-04) |

### 21.1 Lock Procedure

For each decision when locked, record:

1. Final choice.
2. Date locked.
3. Owner/approver.
4. Implementation slice where it is applied.
5. Any backward-compatibility notes.

## 22. Addendum: Review of Odd Object-Level Properties

Purpose:

1. Keep D-04 locked as approved for current implementation.
2. Track a follow-up architectural review for properties that may not belong directly on GameObject long-term.

Initial review candidates:

1. CompositeBuildSuccessMessage.
2. CompositeBuildFailureMessage.
3. CompositeBreakSuccessMessage.
4. CompositeBreakFailureMessage.
5. Any additional composite workflow messaging fields discovered during implementation.

Review questions:

1. Should these fields move into a dedicated composite behavior/config model instead of the core object model?
2. If moved, what compatibility adapter is required for existing project files and export data?
3. Should override policy remain field-level or shift to a grouped composite-configuration override contract?

Planning note:

1. This addendum does not change current locked behavior; it schedules a deliberate design pass to reduce long-term model coupling.

## 23. Implementation Blueprint: Inherit-Then-Override (MVVM-Safe)

Purpose:

1. Ensure instance objects start fully honoring base definition values.
2. Allow intentional per-instance divergence only when producer edits an overridable field.
3. Keep tree/view updates deterministic and leak-safe under MVVM.

### 23.1 Data Shape (Sparse Overrides)

1. Base definition stores canonical values for definition-owned fields.
2. Instance stores DefinitionId plus InstanceOverrides payload.
3. InstanceOverrides is sparse: missing key means inherit from base.
4. For string overrides, support explicit empty-string override (do not treat empty as inherit).
5. Reset action removes the override key so effective value falls back to base.

### 23.2 Effective Value Resolution Contract

For each overridable field F:

1. If instance override for F exists, effective F = override value.
2. Else effective F = base definition value.
3. Non-overridable fields never read from InstanceOverrides.

Required resolver behavior:

1. Centralize logic in EffectiveObjectFieldResolver (single source of truth).
2. Provide helper: HasOverride(field), GetEffective(field), SetOverride(field, value), ClearOverride(field).
3. Keep this resolver host-local in StoryboardDesigner.App unless explicit cross-boundary signoff is approved.
4. Message-script token/value resolution must read via this resolver instead of duplicating ownership/override logic.

### 23.3 Creation and Edit Lifecycle

Creation of instance from base:

1. Write DefinitionId and instance-owned placement/context fields.
2. Do not deep-copy overridable definition fields into overrides.
3. Initialize Name display from effective resolution (base unless local Name override exists).

Producer edits overridable field on instance:

1. Set override key for that field only (copy-on-write for that field).
2. Raise change notification for both raw override state and effective value.

Producer resets field to inherit:

1. Clear override key.
2. Effective value immediately follows base again.

### 23.4 MVVM Tree Update Mechanics

Each instance node viewmodel should track:

1. DefinitionId.
2. Override-state flags for relevant displayed fields (for example HasNameOverride).
3. Effective display properties (for example DisplayName).

Notification rules:

1. On base definition field change:
	update effective property only for nodes where corresponding override is absent.
2. On instance override set/clear:
	update effective property immediately for that node.
3. On DefinitionId relink:
	detach from old definition notifications, attach to new definition, then refresh effective properties.

Subscription safety:

1. Use weak events or deterministic dispose/unsubscribe on node teardown.
2. Rebuild paths must never leave stale subscriptions.
3. Guard against duplicate subscriptions when tree refreshes/rebinds.

### 23.5 Suggested Event Matrix (Deterministic)

1. Base.Name changed + HasNameOverride=false -> raise DisplayName changed.
2. Base.Name changed + HasNameOverride=true -> no DisplayName change.
3. Instance Name override set -> raise DisplayName changed.
4. Instance Name override cleared -> raise DisplayName changed (now inherited).
5. Instance DefinitionId changed -> resubscribe and raise DisplayName changed.

### 23.6 Validation and Regression Tests (Must Add)

1. New instance starts with no override keys for overridable fields.
2. Base edit propagates to instances without overrides.
3. Base edit does not overwrite instances that have overrides.
4. Clearing override restores inheritance and future propagation.
5. Tree rebuild does not leak subscriptions or double-fire updates.
6. Save/load roundtrip preserves sparse overrides and effective values.

### 23.7 Failure-Proofing Notes

1. Never infer inherit-vs-override by comparing values to base (ambiguous); use explicit override presence.
2. Never treat null/empty interchangeably for override semantics.
3. Keep override schema additive to protect backward compatibility readers.

## 24. Concrete Implementation Task Sequence

Use this as the implementation order to minimize risk.

### 24.1 Foundation Data Model Slice

1. Introduce/normalize ObjectType on definition records.
2. Normalize DefinitionId linkage for instances.
3. Add sparse InstanceOverrides payload with explicit override-presence semantics.
4. Add one-time migration utility for sample projects (legacy link fields -> DefinitionId).

### 24.2 Effective Resolver and Policy Services

1. Implement EffectiveObjectFieldResolver with HasOverride/GetEffective/SetOverride/ClearOverride.
2. Implement ObjectOwnershipPolicy using locked ownership matrix.
3. Route script token/value resolution through resolver to avoid duplicate logic.

### 24.3 MVVM Tree and Editor Routing

1. Bind display to effective values, not raw stored fields.
2. Enforce action editing disabled on linked instances.
3. Add GoToBaseObjectCommand navigation for linked instance contexts.
4. Implement deterministic subscription attach/detach lifecycle.

### 24.4 Placement and Promotion UX

1. Restrict Add Instance source list to Base Objects only.
2. Support ancestor-scope browsing with local-first default guidance.
3. Disambiguate same-name candidates by scope path and ObjectType.
4. Persist selection by DefinitionId.
5. Keep Promote to Base Object transactional with rollback guarantees.

### 24.5 Validation and Regression Gates

1. Add ObjectType global uniqueness validation.
2. Add inherit-then-override propagation/non-propagation tests.
3. Add action-ownership guardrail tests for instance nodes.
4. Add migration verification tests for sample project conversion.
5. Run default build/test gates and focused regression gate before signoff.

## 25. Player Construct Reconciliation Review

Purpose:

1. Reconcile expected "player as object" behavior (for example objects marked with isPlayer semantics) with any existing dedicated Player-scope special handling.
2. Remove ambiguity so definition/instance rules apply consistently and producers understand when behavior is object-driven versus host-special-cased.

Review questions:

1. Which behaviors currently rely on the dedicated Player container/scope rather than generic object capabilities?
2. Which behaviors are already modeled as object-level features (including isPlayer-like intent), and where do these overlap/conflict with Player-specific paths?
3. For each overlap, should behavior remain Player-special-cased, move to object capability rules, or be explicitly documented as intentional host behavior?
4. Do current edit flows, validation, and runtime mapping treat player-linked objects consistently with the locked definition/instance ownership model?

Expected outputs:

1. A short inventory of Player-special-cased code paths and why each exists.
2. A decision table: Keep as Player-special-case, refactor to object-capability path, or document as intentional exception.
3. A follow-up implementation mini-slice (if needed) to align outliers with D-03/D-04 ownership rules and D-19 placement policy.
4. Regression tests covering any adjusted player/object routing behavior.

Planning note:

1. This review item is additive and does not change currently locked decisions; it validates that player-related handling is explicit, minimal, and coherent with the rest of this plan.

### 25.1 Initial Inventory of Player-Special-Cased Paths

Designer-authoring host:

1. Global settings flow stores `PlayerCharacterObjectName` and enforces an `isPlayer=true` variable contract on the selected global/player object.
2. Player objects are currently rooted under the dedicated Player container (`project.Player.GameObjects`) and surfaced in tree UX as "Global Game Objects".
3. Legacy project-load compatibility can hydrate old `PlayerGameProperties` and `PlayerAvailableGameActions` into a synthesized/located default Player object.

Runtime/session host behavior (via shared runtime contracts):

1. Runtime world snapshot keeps a dedicated `PlayerObjects` collection and `PlayerCharacterObjectName` setting.
2. Session bootstrap creates a dedicated Player scope node and appends `PlayerObjects` there.
3. Startup can reparent configured player-character object into the starting room and mark `isPlayer=true`.
4. Container-transfer and command-processing paths use `PlayerCharacterObjectName`/`isPlayer` semantics for default source/target behavior.

### 25.2 Decision Table (Current Recommendation)

| Path | Recommendation | Rationale |
|---|---|---|
| Dedicated Player scope container (`PlayerObjects`) | Keep (intentional host behavior) | Preserves current runtime/session contract and avoids broad runtime contract churn in this initiative. |
| `PlayerCharacterObjectName` + `isPlayer` marker behavior | Keep with tighter documentation | Existing command/action paths depend on this routing; behavior is coherent if explicitly documented as player-selection host behavior. |
| Designer enforcement of `isPlayer` variable on selected global object | Keep | Aligns authoring intent with runtime selection semantics. |
| Legacy load merge of `PlayerGameProperties` / `PlayerAvailableGameActions` into default Player object | Keep short-term, schedule removal gate | Needed for backward compatibility during migration window; should be retired once sample-project migration evidence is complete. |
| Definition/instance mapping for player-rooted objects | Keep aligned with object-capability rules (already in progress) | Player-rooted objects should obey same DefinitionId/effective-value semantics as room/template objects except for explicit player-selection behavior. |

### 25.3 Follow-up Mini-Slice (Planned)

1. Add explicit documentation note in runtime mapping and global settings flows that `isPlayer`/`PlayerCharacterObjectName` are host routing semantics, not ownership exceptions.
2. Add/extend tests proving player-rooted linked instances honor effective definition-owned runtime mapping (name/nameInGame/action metadata parity with room instances).
3. Add a retirement checklist item for legacy `PlayerGameProperties`/`PlayerAvailableGameActions` load-merge path, gated by one-time migration completion evidence.

### 25.4 Mini-Slice Progress (2026-07-04)

Completed:

1. Added explicit in-code intent notes in designer runtime mapping and global settings player contract paths.
2. Added player-rooted linked-instance runtime mapping regression coverage.

Retirement checklist (legacy player load-merge compatibility):

1. Confirm one-time sample-project migration utility has been run on maintained sample projects. Completed (2026-07-04) via maintained sample audit evidence below.
2. Verify no maintained project files still populate `PlayerGameProperties` or `PlayerAvailableGameActions` legacy fields. Completed (2026-07-04).
3. Add/verify regression that loading migrated files preserves player object behavior without legacy merge fallback. Completed (2026-07-04).
4. Remove compatibility merge block from loader and update migration notes in this plan execution log. Completed (2026-07-04).

## 26. Execution Delta Log (2026-07-04)

### 26.1 Completed in This Delta

1. Runtime snapshot mapping for GameObject now resolves effective linked values for ScopeName and ScopeNameInGame using the centralized effective-value resolver.
2. Runtime scope tokens for mapped GameObject descriptors now derive from effective name values instead of stale local linked-instance copies.
3. Runtime mapping now resolves definition-owned command surfaces for linked instances from effective base definition source:
	1. AdditionalVerbs
	2. AdditionalDirectionals
	3. AdditionalDirectionalTraversalMappings
	4. AvailableActions
4. Added regression tests proving:
	1. Linked instances without overrides inherit definition Name/NameInGame in runtime snapshot.
	2. Linked instances with NameInGame override preserve the override in runtime snapshot.
	3. Definition-owned command surfaces are used for linked-instance runtime materialization even when stale local values are present.

Boundary Status: Designer-only (no Shared/Simulator code changes).

### 26.2 Validation Evidence

Passing validations:

1. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~QuantifiableRuntimeMaterializationTests|FullyQualifiedName~LinkedInstanceActionContextMenuTests|FullyQualifiedName~GameCommandProcessorFixtureTests|FullyQualifiedName~GameCommandProcessorLinkedActionsTests"`
2. `dotnet build .\StoryboardDesigner.slnx`

Superseded status update (2026-07-05):

1. Birmingham playback snapshot mismatch was resolved by refreshing recordings with `UPDATE_PLAYBACK_SNAPSHOTS=1`.
2. Full test project now passes with latest baseline (382/382).

### 26.3 Section 25 Mini-Slice Delta (2026-07-04)

1. Added explicit in-code intent notes documenting that `PlayerCharacterObjectName`/`isPlayer` are host routing semantics, not definition/instance ownership exceptions.
2. Added retirement-checklist gating criteria for removing legacy player load-merge compatibility (`PlayerGameProperties`/`PlayerAvailableGameActions`).
3. Validation run:
	1. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~QuantifiableRuntimeMaterializationTests|FullyQualifiedName~GameStateSessionPlayerScopeTests|FullyQualifiedName~JsonExportServiceProjectStateTests"` passed (33/33).
	2. `dotnet build .\StoryboardDesigner.slnx` passed.

### 26.4 Retirement-Gate Regression Delta (2026-07-04)

1. Added regression test `TryLoadProjectModel_PreservesPlayerSelectionFromGlobalsSidecar_WhenLegacyPlayerPayloadFieldsAreEmpty` in `JsonExportServiceProjectStateTests`.
2. Test proves migrated sidecar-based player object data loads correctly with `playerGameProperties`/`playerAvailableGameActions` empty, without requiring legacy merge fallback.
3. Validation run:
	1. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~JsonExportServiceProjectStateTests|FullyQualifiedName~GameStateSessionPlayerScopeTests|FullyQualifiedName~QuantifiableRuntimeMaterializationTests"` passed (34/34).
	2. `dotnet build .\StoryboardDesigner.slnx` passed.

### 26.5 Legacy Merge Retirement Delta (2026-07-04)

1. Removed legacy inline player payload merge fallback from `JsonExportService.TryLoadProjectModel`.
2. Loader now intentionally ignores legacy inline `playerGameProperties` / `playerAvailableGameActions` payloads and relies on migrated global object sources.
3. Added regression `TryLoadProjectModel_IgnoresLegacyInlinePlayerPayloadFields_AfterRetirement`.
4. Validation run:
	1. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~JsonExportServiceProjectStateTests|FullyQualifiedName~GameStateSessionPlayerScopeTests|FullyQualifiedName~QuantifiableRuntimeMaterializationTests"` passed (35/35).
	2. `dotnet build .\StoryboardDesigner.slnx` passed.

### 26.6 Maintained Sample Audit Evidence (2026-07-04)

1. Audited maintained sample project files:
	1. `Samples\Birmingham\Birmingham.sbe.json`
	2. `Samples\MapDemo1\MapDemo1.sbe.json`
	3. `Samples\TraversalExamples\TraversalExamples.sbe.json`
2. Audit result: no maintained sample project has populated legacy inline player payload arrays.
	1. `playerGameProperties` counts: 0, 0, 0.
	2. `playerAvailableGameActions` counts: 0, 0, 0.
3. Compatibility status after loader retirement: maintained samples are aligned with migrated source-of-truth behavior.

### 26.7 Runtime-Focused Gate Refresh (2026-07-04)

1. Refreshed simulator playback recordings with `UPDATE_PLAYBACK_SNAPSHOTS=1` for `GameSimulatorPlaybackRegressionTests`.
2. Re-ran runtime-focused guardrail suite:
	1. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"` passed (18/18).
3. Re-ran solution build:
	1. `dotnet build .\StoryboardDesigner.slnx` passed.

### 26.8 Full Regression Confidence Gate (2026-07-04)

1. Executed full test project regression sweep:
	1. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj` passed (368/368).
2. End-of-slice status: current implementation deltas validated by focused gates, runtime-focused guardrail gate, and full project test pass.

### 26.9 Completion Sweep Delta (2026-07-05)

1. Extended promotion workflow guardrails validated:
	1. Transactional rollback coverage.
	2. Scope-targeted promotion selection (Area/Country/Planet/Global).
	3. Duplicate-name collision blocking in selected scope.
	4. Cancellation no-op behavior for model and hierarchy tree nodes.
	5. Scope picker contract assertions for option order/labels.
2. Re-ran runtime playback updates and full regression:
	1. `UPDATE_PLAYBACK_SNAPSHOTS=1 dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameSimulatorPlaybackRegressionTests"` passed (3/3).
	2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj` passed (382/382).
	3. `dotnet build .\StoryboardDesigner.slnx` passed.
3. Non-item-6 signoff status:
	1. Slices T1-T5: Signed off.
	2. Section 22 addendum review: Deferred intentionally per request; execute only after all other work is clearly signed off.
