# Room Template Bootstrap Plan

Status: Active (Lockoff pending)
Owner: StoryboardDesigner.App room authoring workflow + serialization
Last updated: 2026-07-08

## 1. Purpose

Add room-level template workflow parity with object templates:

1. Room -> template promotion (`Create Template From`).
2. Template -> new room bootstrap (`Add New Room` chooses template or blank).
3. One-time copy semantics only (no persistent link between created room and room template).

## 2. Why This Plan Exists

Room authoring currently lacks a reusable template loop equivalent to object templates.

Symptoms:

1. Producers can reuse object templates but not room structures.
2. Recreating similar room setups is repetitive and risks inconsistency.
3. There is no explicit room-template catalog under Global for durable reuse.

## 3. Goals

1. Add a `Room Templates` node under Global in the hierarchy.
2. Allow promoting a well-defined room into a room template with naming guardrails.
3. Prompt for optional room template selection during `Add New Room`.
4. Bootstrap new room from selected template using one-time deep copy.
5. Keep runtime/designer behavior deterministic and serialization safe.

## 4. Non-Goals (Phase 1)

1. No long-term room-to-template linkage or sync.
2. No inherited/override model for rooms.
3. No broad redesign of room editing UX beyond template selection/promote entry points.
4. No automatic migration of existing rooms into templates.

## 5. Core Design Direction

## 5.1 Data Model Direction

1. Add `ProjectModel.RoomTemplates` collection (parallel to `ObjectTemplates`).
2. Introduce room-template scope node model for hierarchy/validation context.
3. Store room templates in project serialization shape with deterministic ordering.

## 5.2 UX Entry Points

1. Room context menu: `Create Template From`.
2. Global node: `Room Templates` catalog with `Add New Room Template`.
3. `Add New Room` flow: prompt template picker (first option: blank room).

## 5.3 Copy Semantics

1. Template application is deep clone at creation time.
2. New room receives new identifiers for room, contained objects, actions, traversal legs where applicable.
3. No `DefinitionId`/link metadata equivalent for rooms in phase 1.
4. Post-bootstrap edits to room and template are independent.

## 6. Lockoff Questions (Must Resolve Before Full Implementation)

1. RBT-01: Scope and placement of `Room Templates` in hierarchy:
   1. Under Project root near `Object Templates` / `Base Objects`?
   2. Global-only catalog (initial scope) vs scoped catalogs (future).
2. RBT-02: Template payload breadth:
   1. Include room settings, variables, actions, verbs, directionals.
   2. Include contained room objects and nested objects.
   3. Exclude traversal connections/area placement metadata for phase 1.
3. RBT-03: Naming policy:
   1. Template name prompt behavior.
   2. Duplicate-name handling (warn and block vs auto-suffix).
4. RBT-04: Add New Room prompt behavior:
   1. Always prompt when templates exist.
   2. Offer `Blank Room` as explicit first option.
5. RBT-05: Serialization placement and sidecar behavior:
   1. Main project JSON only in phase 1.
   2. Keep parity with existing global import/export patterns where applicable.
6. RBT-06: Validation scope behavior for room templates:
   1. Whether all room-level rules run on templates immediately.
   2. Whether any room-runtime-only rules should be suppressed.
7. RBT-07: Identifier remap policy during template bootstrap:
   1. Confirm all cloned entities receive new IDs (room, contained objects, nested objects, actions, action blocks).
   2. Define how action payload object-ID references are remapped to cloned object IDs.
   3. Define failure behavior when a payload reference cannot be remapped deterministically.
8. RBT-08: Traversal-leg and traversal-action policy:
   1. Confirm whether room template includes traversal-leg state/actions at all in phase 1.
   2. If excluded, define explicit normalization so cloned rooms never carry stale leg/openable references.
9. RBT-09: Shared-variable reference policy:
   1. Decide whether shared-variable IDs inside cloned room/object variables are preserved, cleared, or gated.
   2. Define expected behavior when referenced shared variables do not exist in destination project state.
10. RBT-10: Add New Room naming and prompt ordering policy:
   1. Confirm whether template selection happens before or after room name/settings edit dialog.
   2. Confirm uniqueness strategy for room name conflicts when template default name collides in target area.
11. RBT-11: Import/export and globals workflow integration:
   1. Decide whether room templates participate in global import/export dialogs in phase 1.
   2. If deferred, define explicit non-goal note and follow-up owner.

## 6.1 Lock Decisions (In Progress)

1. RBT-01: Locked (2026-07-08)
   1. `Room Templates` is a Global-only catalog in phase 1.
   2. Hierarchy placement is under project root, parallel to existing global template/base catalogs.
   3. Planet/Country/Area scoped room-template catalogs are explicitly deferred to a future enhancement.
   4. Future-ready constraint for phase 1 implementation:
      1. Use catalog abstraction seams now (do not hardcode logic exclusively to one concrete node type where avoidable).
      2. Keep promotion/bootstrap logic catalog-agnostic so future scoped catalogs reuse the same core workflow.
      3. Preserve a stable scope-kind path for future extension (Global now, scoped variants later) without changing room-template payload semantics.
2. RBT-02: Locked (2026-07-08)
   1. Phase 1 room template payload includes:
      1. Room settings/basic fields.
      2. Room-level variables.
      3. Room-level scoped actions.
      4. Room-level verbs and directionals.
      5. Contained room objects recursively (properties, variables, actions, nested contained objects).
   2. Phase 1 room template payload excludes:
      1. Traversal connections.
      2. Area placement/grid-position metadata.
      3. Any cross-room references outside the templated room boundary.
   3. Bootstrap normalization rule:
      1. New room/bootstrap clone receives regenerated IDs.
      2. Out-of-boundary references are cleared deterministically with diagnostics/test coverage.
3. RBT-03: Locked (2026-07-08)
   1. `Create Template From` always prompts for template name, prefilled from source room name.
   2. Name is trimmed/normalized; blank names are blocked with warning.
   3. Duplicate template names are blocked with warning (no auto-suffix in phase 1).
   4. Duplicate comparison is case-insensitive for consistency with existing template workflows.
4. RBT-04: Locked (2026-07-08)
   1. `Add New Room` opens room-template picker when at least one room template exists.
   2. Picker always provides `Blank Room` as first option.
   3. If no room templates exist, flow goes directly to blank-room creation.
   4. Canceling template picker cancels room creation with no side effects.
5. RBT-05: Locked (2026-07-08)
   1. Phase 1 persists room templates in the primary project JSON (same persistence tier as object templates).
   2. Sidecar split for room templates is out of scope in phase 1.
   3. Global import/export dialog integration is deferred in phase 1 except where required to guarantee no-data-loss round-trip behavior.
   4. Serialization compatibility rule: older project files that do not contain room templates load with an empty `RoomTemplates` collection by default.
6. RBT-06: Locked (2026-07-08)
   1. Room templates run the full structural and authoring-time room validation rules in phase 1.
   2. Rules that require live placement/runtime wiring context (for example area-traversal linkage presence) are suppressed in template scope.
   3. Suppressed-rule handling emits explicit informational diagnostics (not errors) indicating rule skip due to template scope.
   4. Regression coverage must verify both parity for shared rules and explicit skip behavior for suppressed context-dependent rules.
7. RBT-07: Locked (2026-07-08)
   1. Identifier schema remains unchanged in phase 1: room/object/action node identity continues to use the existing singular `ScopeNodeId` property.
   2. During template bootstrap, every cloned in-boundary entity gets a newly assigned `ScopeNodeId` value (room, contained/nested objects, room/object actions, action blocks where represented as scope nodes).
   3. Clone process constructs and carries an old->new `ScopeNodeId` map for the full cloned graph.
   4. Action payload fields that reference node IDs are remapped through the old->new `ScopeNodeId` map when the field semantics are known.
   5. If a payload reference cannot be remapped deterministically, that specific reference is cleared and an authoring diagnostic is emitted with action/field context.
   6. Bootstrap hard-fails only when unresolved references violate required integrity constraints; otherwise it succeeds with deterministic normalization diagnostics.
8. RBT-08: Locked (2026-07-08)
   1. Traversal data is never part of room templates in phase 1.
   2. Room templates do not participate in map/area-editor traversal creation workflows (cross-room link authoring remains room-instance-only after placement).
   3. When creating a template from an existing room, all traversal data from the source room is dropped and excluded from the resulting template.
   4. During bootstrap from a room template, created rooms start with traversal defaults equivalent to blank-room creation (no implicit leg wiring).
   5. Normalization must clear traversal/openable linkage fields that reference out-of-template boundary entities.
   6. Authoring diagnostics/log output should indicate traversal-data drop/normalization when applicable.
9. RBT-09: Locked (2026-07-08)
   1. Phase 1 permits authoring-time shared-variable links without hard UI blocking, but template boundary enforcement is mandatory during normalization workflows.
   2. Boundary rule: shared-variable links that point outside the room-template boundary are out of scope and must not persist in room templates or bootstrap-created rooms.
   3. `Create Template From` runs boundary validation; any out-of-boundary shared-variable link is reported with warning diagnostics and dropped from the resulting room template.
   4. `Add New Room` bootstrap from room template repeats boundary validation; any remaining out-of-boundary shared-variable link is reported with warning diagnostics and dropped during room creation.
   5. In-boundary shared-variable links are preserved and remapped via the old->new `ScopeNodeId` map so independently bootstrapped rooms do not cross-link.
10. RBT-10: Locked (2026-07-08)
   1. Prompt order is template picker first, then room configuration/name entry.
   2. Selected template does not provide or prefill the created room name.
   3. Room creation from template requires explicit manual room-name entry by the producer.
   4. Room-name uniqueness conflicts are blocked and require explicit rename (no silent auto-suffix in phase 1).
   5. Cancel at either step aborts room creation with no side effects.
11. RBT-11: Locked (2026-07-08)
   1. Room templates participate in global import/export workflows in phase 1.
   2. Import flow prompts the user to include or exclude room templates, consistent with existing global import behavior in other areas.
   3. Export flow includes room templates as part of the supported global-transfer payloads for phase 1.
   4. Import/export coverage must include no-data-loss round-trip verification for room-template payloads.

## 7. Implementation Slices

### Slice T0: Lockoff + Contract Definition

Tasks:

1. Resolve RBT-01..RBT-11.
2. Define exact room-template clone contract and exclusions.
3. Define minimal test matrix and acceptance criteria.

Exit criteria:

1. Lock decisions recorded in this plan.
2. No unresolved behavior ambiguity remains for phase 1.

### Slice T1: Model + Serialization Foundation

Tasks:

1. Add `RoomTemplates` model collection and scope-node support.
2. Add hierarchy node/viewmodel for `Room Templates` catalog.
3. Add JSON read/write support and regression tests.

Exit criteria:

1. Room templates persist/load correctly.
2. Hierarchy renders `Room Templates` node under Global.

### Slice T2: Room -> Template Promotion

Tasks:

1. Add room context action `Create Template From`.
2. Implement name prompt, duplicate detection, and catalog insertion.
3. Implement deep-clone + normalization behavior for room template definition.

Exit criteria:

1. Promotion flow works end-to-end.
2. Tests verify cancel/duplicate/success paths and tree updates.

### Slice T3: Template -> Add New Room Bootstrap

Tasks:

1. Add template picker into `Add New Room` flow.
2. Include `Blank Room` option.
3. Clone selected room template into new independent room instance.
4. Keep existing room configuration dialog behavior consistent.

Exit criteria:

1. New room can be bootstrapped from template or blank.
2. Created room has no persistent link to template.

### Slice T4: Validation + UX Hardening

Tasks:

1. Align validation traversal and diagnostics for template scope.
2. Add focused UI/workflow regression tests.
3. Clean up UX labels/status output and help text.

Exit criteria:

1. Validation behavior is explicit and tested.
2. Core workflows are stable and documented.

## 8. Validation Gates

Between-slice gate (required before moving from one slice to the next):

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`

Per slice (minimum):

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "PromoteToObjectTemplateWorkflowTests|JsonExportServiceProjectStateTests|GameCommandProcessorFixtureTests"`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`

Slice exit full cycle:

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`

## 9. Initial Acceptance Criteria (Phase 1)

1. `Room Templates` appears under Global.
2. User can create room template from an existing room.
3. `Add New Room` offers template picker with `Blank Room` option.
4. Created room is independent snapshot copy with no ongoing template relationship.
5. Project save/load round-trips room templates without data loss.
6. All relevant regression tests pass.

## 10. Notes

1. This plan intentionally mirrors proven object-template workflow patterns to reduce implementation risk.
2. Future enhancements (scoped room template catalogs, live linkage, partial-template application) are out of scope for phase 1.
