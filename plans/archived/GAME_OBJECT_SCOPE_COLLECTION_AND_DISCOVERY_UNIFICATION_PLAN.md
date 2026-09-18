# Game Object Scope Collection and Discovery Unification Plan

Status: Closed
Owner: StoryboardDesigner.App authoring and scope modeling
Last updated: 2026-07-17

## Objective

Unify how authored game object collections are modeled and discovered across Global and Room scopes first, then extend the same authored game object collection capability to Planet/Country/Area scopes without changing Base Object semantics.

## Program-Level Goal (North Star)

Standardize scope-tree treatment around shared object-collection patterns so each scope can express its unique rules without inventing a separate structural model.

1. Prefer one common shape for authored Game Objects collection nodes across scopes.
2. Keep scope-specific differences explicit and minimal (for example, action availability or scope ownership semantics).
3. Centralize object candidate discovery so lock/composite/select workflows do not fork behavior by scope.
4. Reduce special-case code paths that make Global, Room, and future scope levels feel inconsistent.

## High-Level Topics (Explicitly Denoted)

1. Global scoped game actions are missing at the Global node and must be implemented.
2. Global and Room game object children are represented with inconsistent node/viewmodel patterns and must be unified.
3. Planet/Country/Area currently do not support authored game object children and must be extended to do so.

## In Scope

1. Global node game actions authoring, persistence, mapping, and runtime execution support.
2. Authoring-model support for GameObjects collections at the selected scope levels.
2. Tree/viewmodel unification for Game Objects collection nodes.
3. Selection/discovery behavior for object pickers that depend on scope traversal.
4. Composite recipe builder and lock key picker use of a single discovery path.
5. Persistence/export/runtime mapping updates required by new authored GameObjects collections.

## Out of Scope

1. Base Object behavior, storage semantics, or UX workflows.
2. Any change to template/base catalog intent.
3. Runtime command semantics unrelated to object discovery or scope ownership.

## Non-Negotiable Constraint

1. Base Objects remain untouched in behavior and semantics during this plan.

## Why This Plan Is Needed

1. Global and Room currently represent similar Game Objects collections with different viewmodel types.
2. Composite recipe part selection still uses custom source-object lookup code instead of the discovery service.
3. Upcoming addition of authored GameObjects at Planet/Country/Area will force a broader scope traversal update.
4. Consolidating discovery logic first reduces duplicate refactor risk.

## Current Hotspots

1. Global scoped actions are absent from Global node editing and runtime pathways.
2. Composite recipe builder currently uses custom candidate path:
   1. BuildAvailableCompositePartOptions and GetCompositePartSourceObjects overloads in MainWindowViewModel.ProjectExplorer.
   2. This bypasses GameObjectSelectionOptionDiscoveryService.
3. Lock key picker already uses discovery service:
   1. BuildAvailableLockKeyOptions in MainWindowViewModel.ProjectExplorer.
4. Discovery service has room/global and template/base assumptions that will drift once new scope-level GameObjects are added.

## Phase A - Global Game Actions at Global Node

Status update (2026-07-17): Closed. Global-scoped game actions are now authorable from the Global root node, persisted in globals sidecar storage, emitted in clean export, mapped into runtime snapshots, and resolved by runtime command processing with regression coverage.

1. Add Global-scoped Game Actions to the Global node authoring surface (tree/context and edit dialog wiring).
2. Persist/load Global-scoped Game Actions in project storage.
3. Include Global-scoped Game Actions in clean export/snapshot mapping.
4. Ensure runtime command resolution includes Global-scoped actions.
5. Add regression tests for Global action authoring and execution behavior.

Exit criteria for Phase A:

1. User can author/edit/remove Game Actions directly on Global node.
2. Global actions survive save/load/export.
3. Runtime can resolve and execute Global actions in command processing.

## Prerequisite (Phase 0) - Mandatory Before Scope Collection Refactor

1. Route composite recipe builder candidate lookup through IGameObjectSelectionOptionDiscoveryService.
2. Remove direct custom composite part source lookup helpers once parity is achieved.
3. Ensure composite part picker and lock key picker both resolve candidates from the same discovery service entry point.
4. Add regression tests proving composite part options are driven by discovery service behavior rather than local custom lists.

Exit criteria for Phase 0:

1. No production composite-part candidate path calls BuildAvailableCompositePartOptions/GetCompositePartSourceObjects-style custom source traversal.
2. Composite-part candidate tests and lock-key candidate tests pass under the same discovery policy assumptions.

## Phase 1 - Discovery Service Hardening for Scope-First Modeling

1. Make discovery semantics scope-oriented rather than room-oriented naming.
2. Refactor sibling/local and project traversal so authored real object discovery does not depend on base-object branches.
3. Keep template discovery explicit and isolated from real-object discovery.
4. Add tests for:
   1. Local scope candidates in Room and Global containers.
   2. Ancestor and children behavior.
   3. Project-wide real-object discovery excluding base/template catalogs.
   4. Template-mode discovery behavior unaffected.

Exit criteria for Phase 1:

1. Discovery service behavior is deterministic for real objects across supported authored scopes.
2. Base object traversal is not part of real-object discovery path.

## Phase 2 - Core ViewModel Unification for Game Object Collection Nodes

1. Replace parallel room/global/player collection node types with a shared generalized collection node model for authored GameObjects.
2. Keep user-facing label as Game Objects.
3. Preserve current add/remove object commands and context menus while switching to generalized node plumbing.
4. Keep this phase behavior-preserving: no new scope-level GameObjects yet.

Exit criteria for Phase 2:

1. Global and Room Game Objects tree behavior is consistent and uses the same node pattern.
2. Existing object add/remove/edit flows pass unchanged tests.

## Phase 3 - Remove Children Folder Layer (Post-Unification Flattening)

Status update (2026-07-17): Closed for targeted Room/Template Room scope. Parent-scope grouping (Planet/Country/Area) remains unchanged by design for this phase.

1. Remove intermediate Children group folder from Room and aligned scope nodes where it is currently only a container layer.
2. Project Traversal Legs and Game Objects directly under the owning node where applicable.
3. Preserve existing context actions and command routing behavior after flattening.
4. Preserve node path restore compatibility for previously persisted selections.

Exit criteria for Phase 3:

1. Tree interaction no longer requires expanding a Children folder to reach Traversal Legs or Game Objects for targeted scopes.
2. Existing add/remove/edit actions remain behavior-equivalent.
3. UI state restore remains backward-compatible with prior saved node paths.

## Phase 4 - Remove Configuration Folder Layer (Post-Unification Flattening)

Status update (2026-07-17): Closed. Configuration folder layer removed from projected hierarchy nodes (including scope and object nodes) while preserving existing context actions and traversal/object access paths.

1. Remove intermediate Configuration group folder from Room and aligned scope nodes where it is currently only a container layer.
2. Project Settings, Properties, Actions, Verbs, and Directionals directly under the owning node where applicable.
3. Preserve existing editing workflows and context command coverage after flattening.
4. Preserve node path restore compatibility for previously persisted selections.

Exit criteria for Phase 4:

1. Tree interaction no longer requires expanding a Configuration folder to reach editable configuration entries for targeted scopes.
2. Existing settings/property/action/token editing flows remain behavior-equivalent.
3. UI state restore remains backward-compatible with prior saved node paths.

## Phase 5 - Model Extension for Planet/Country/Area Authored GameObjects

Status update (2026-07-17): Closed. Planet/Country/Area authored Game Objects collections are modeled, attached to scope hierarchy ownership rules, projected in hierarchy using shared Game Objects collection node patterns, and covered by passing regression validation.

1. Introduce authored GameObjects collections to Planet/Country/Area models.
2. Define child-scope ownership and parent attachment rules for these new collections.
3. Extend hierarchy builder to show Game Objects collections at these scopes using the generalized node pattern.
4. Extend add/remove/edit object workflows for the new scopes.

Exit criteria for Phase 5:

1. Planet/Country/Area can author and display GameObjects collections.
2. Scope hierarchy parent links remain valid.

## Phase 6 - Persistence, Export, and Runtime Mapping

Status update (2026-07-17): Closed. Planet/Country/Area authored Game Objects now persist through native project save/load, are emitted in clean export contracts, and are attached/mapped into runtime bootstrap scope trees with regression coverage.

1. Persist/load newly introduced scope-level authored GameObjects.
2. Extend clean export and snapshot mapping for these objects.
3. Validate runtime bootstrap/session mapping includes new scope-owned objects where intended.
4. Add migration-safe handling for older project files missing new fields.

Exit criteria for Phase 6:

1. Save/load roundtrip preserves new scope-level GameObjects.
2. Export and runtime mapping include those objects correctly.

## Phase 7 - Validation Rules and Regression Protection

Status update (2026-07-17): Closed. Validation lookup/rule traversal now includes Planet/Country/Area authored Game Objects across ACT-009/010/011/012, OBJ-003/004, shared-variable consistency support, and scripting token discovery; focused and full regression gates are green.

1. Update validation lookup/rules that enumerate scoped objects.
2. Add regression tests for:
   1. Discovery service consumers (lock keys, composite recipes).
   2. Tree/context command behavior at all supported scopes.
   3. Save/load/export/runtime roundtrip for new scope-level objects.
3. Confirm architecture guardrails remain intact.

Exit criteria for Phase 7:

1. Validation and regression suite covers all affected object ownership paths.

## Recommended Order of Attack

1. Phase 0 first: move composite part picker onto discovery service to eliminate duplicate lookup logic.
2. Phase 1 next: harden discovery service semantics before broadening scope-level object storage.
3. Phase 2 next: core unification of Room/Global/Player Game Objects collection node/viewmodel shape.
4. Run the Unification Hard Gate (automated and manual validation) before any additional post-unification phases.
5. Phase 3 next: remove Children folder layer for targeted scopes.
6. Phase 4 next: remove Configuration folder layer for targeted scopes.
7. Phase 5 then Phase 6 then Phase 7 for Planet/Country/Area collection expansion and full pipeline integration.
8. Phase A last: add Global Game Actions after disruptive scope-modeling work is stable.

## Implementation Order and Risk Control

1. Do not begin Phase 2+ until Phase 0 is complete.
2. Treat Phase 0 to Phase 2 as the disruptive tranche and keep behavior parity during this tranche.
3. Do not begin Phase 3+ until the Unification Hard Gate is fully passed.
4. Keep each phase in small reviewable slices with passing tests.
5. Avoid touching Base Object code paths unless a compile-time dependency requires neutral adaptation only.

## Unification Hard Gate (Must Pass Before Phase 3)

1. Full automated test run must pass:
   1. dotnet build .\StoryboardDesigner.slnx
   2. dotnet test .\Storyboard.Shared.Tests\Storyboard.Shared.Tests.csproj
   3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
2. Runtime-focused guardrail tests must pass:
   1. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"
   2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
3. Manual validation must be explicitly completed and recorded before moving on:
   1. Verify lock-key picker options still match discovery-service scope rules in Room and Global contexts.
   2. Verify composite recipe part picker options are discovery-driven and parity-match expected authored objects.
   3. Verify add/remove/edit object flows remain unchanged for Room and Global Game Objects trees.
   4. Verify save/load roundtrip for representative projects does not regress existing object relationships.
   5. Verify no Base Object workflows or semantics changed.

Waiver note for this run: Manual item 3.5 (Base Object workflows/semantics unchanged) is explicitly deferred and waived for Phase 2 completion approval dated 2026-07-17.

## Manual Validation Run Log (Phase 2 Closeout)

Use this checklist to explicitly record completion of the required manual hard-gate items before moving to Phase 3.

- Run date: 2026-07-17
- Tester: User (manual validation)
- Build/reference branch or commit: ____________________
- Notes summary: Completed 4 of 5 manual hard-gate checks; Base Object workflow check intentionally deferred.

| Check | Status (`Pending` / `Pass` / `Fail`) | Evidence / Notes |
|---|---|---|
| Lock-key picker options match discovery-service scope rules in Room and Global contexts | Pass | Manually validated by user on 2026-07-17. |
| Composite recipe part picker options are discovery-driven and parity-match expected authored objects | Pass | Manually validated by user on 2026-07-17. |
| Add/remove/edit object flows remain unchanged for Room and Global Game Objects trees | Pass | Manually validated by user on 2026-07-17. |
| Save/load roundtrip for representative projects preserves existing object relationships | Pass | Manually validated by user on 2026-07-17. |
| Base Object workflows/semantics unchanged | Pending | Deferred by user and explicitly waived for Phase 2 closeout. |

Phase 2 manual hard-gate signoff:

- Signoff owner: User (manual waiver)
- Signoff date: 2026-07-17
- Decision: `Approved with deferred Base Object validation waiver for Phase 2 completion`

## Validation Matrix (Minimum)

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"
4. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"

## Open Decisions

1. Scope naming: whether enum/API names should move from LocalRoom terminology to LocalScope in one pass or staged aliases.
2. Runtime ownership semantics for scope-level authored objects outside rooms (materialization rules and traversal visibility expectations). Resolved 2026-07-17.
3. Migration policy for existing projects once Planet/Country/Area GameObjects are introduced.

Post-implementation review snapshot (2026-07-17):

1. Scope naming recommendation:
   1. Keep current naming stable in the shipped implementation slice.
   2. Plan a separate low-risk cleanup pass that introduces staged aliases first, then optional rename follow-up after one release cycle.
2. Runtime ownership semantics recommendation:
   1. Scope-owned authored objects outside rooms must never behave as if they are room-present objects unless explicitly moved into a room.
   2. These objects are not room-rendered while they remain at parent scopes (Area/Country/Planet).
   3. Command handling expectation: if room-level object resolution does not satisfy a verb, scope lookup may continue up the ancestry chain and execute matching parent-scope object actions.
   4. These parent-scope objects are valid long-lived state carriers for game logic (for example, set/flag/chat variable driven progression across a long area session).
   5. Future movement is allowed: when a parent-scope object is moved into the current room, it is treated as a room object from that point forward.
3. Migration policy recommendation:
   1. Keep additive, tolerant-read migration policy.
   2. Do not force one-shot conversion for existing projects.
   3. Continue writing new fields on save/export while preserving backward-compatible defaults for older files missing those fields.

## Deferred Cleanup Review (Post-Implementation)

1. Review GameObjectOptionScopeTarget enum as a whole after this work lands and decide whether it remains clear as-is or needs targeted cleanup/renaming.

Status update (2026-07-17): Deferred as non-blocking follow-up. No functional regression tied to current enum naming was identified during closure validation.

## Plan-End Completion Item (Required Before Plan Close)

1. Rename clean-export contract field `playerGameObjects` to `globalGameObjects` and perform a one-time migration of repository sample/snapshot clean JSON files.
2. Status: Addressed in current implementation slice (DTO/property rename, mapper/export updates, sample/snapshot clean JSON migration).
