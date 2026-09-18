# Refocus On ObjectIds Plan

Last updated: 2026-07-07
Status: Closed

## Intent

Reduce ID complexity and eliminate runtime/design-time ID drift by adopting a single identity rule:

1. Authored `objectId` values are the canonical IDs at design time.
2. The same `objectId` values are used at runtime for authored objects.
3. Runtime must stop generating alternate IDs for authored objects.
4. Scope position must be represented explicitly by parent linkage (`parentScopeId` / `runtimeParentScopeId` style), not by deriving identity from tree position.

## Core Direction (Non-Negotiable)

1. No mapper should persist runtime-transformed object IDs back into authored project content.
2. Composite action references (`compositeTargetObjectId`, `compositeRequiredPartObjectIds`) remain authored-ID references in project files.
3. Runtime object movement/placement is represented by parent relationships, not identity rewrites.
4. Diagnostics should emit IDs that map directly to project files for authored objects.
5. Composite recipe ownership is target-centric: recipe membership is defined by the target recipe's required parts, not by reverse-link IDs on part objects.

## Non-Goals (For This Plan)

1. No broad runtime redesign in one patch.
2. No schema-breaking clean export jump without explicit version decision.
3. No high-risk rewrite without intermediate compatibility and validation gates.

## Migration Strategy

Adopt a phased compatibility approach. Each phase has explicit tests and rollback criteria.

---

## Phase 0: Lock In Current Guardrails (Already Started)

### Goals

1. Detect invalid authored references early.
2. Prevent future silent corruption while deeper refactor proceeds.

### Work

1. Keep warning validations active:
- `ACT-010`: missing required part IDs.
- `ACT-011`: missing BuildCompositeByParts target object ID.
- `ACT-012`: BuildCompositeByParts target-recipe mismatch.

2. Ensure validation registry includes all three rules in default flow.

### Validation Gate

1. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "ActionIntegrityRulesTests"`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`

### Exit Criteria

1. Warning rules fire on bad data and stay silent on valid data.
2. No regression in full test suite.

---

## Phase 1: Authoring Boundary Hardening (Stop New Drift)

### Goals

1. Ensure editor/viewmodel persistence paths only write authored object IDs.
2. Block any runtime-ID leakage into `CommandAction` composite fields.

### Work

1. Trace and patch BuildCompositeByParts authoring selection paths to source object IDs from authored model graph only.
2. Add regression tests that save/reload a project and assert composite IDs equal authored object IDs.
3. Add a targeted fixture asserting no deterministic runtime ID patterns are persisted into authored composite fields.

### Validation Gate

1. Focused tests:
- `ActionIntegrityRulesTests`
- `JsonExportServiceProjectStateTests`
- composite editor/payload tests related to BuildCompositeByParts
2. `dotnet build .\StoryboardDesigner.slnx`

### Exit Criteria

1. New authored saves never contain runtime-derived IDs in composite fields.
2. Existing warning rules remain green on valid newly-authored content.

---

## Phase 1B: Recipe Ownership Simplification (Design-Time Model Clarity)

### Goals

1. Scrutinize why non-target part objects carry `compositeRecipeId`.
2. Move toward one-way recipe definition where target recipes own required-part linkage.
3. Prevent multi-directional ID coupling that increases drift risk.

### Design Principle

1. Recipe topology should be represented as:
- Target object owns recipe metadata (`compositeRecipeId`, required parts list).
- Part objects do not need reverse recipe pointers for normal build-from-parts behavior.

2. If part-level recipe fields remain for compatibility, they must be treated as optional legacy data and must not be authoritative.

### Work

1. Inventory every runtime/validation/editor path that reads `GameObject.CompositeRecipeId` on non-target objects.
2. Classify each usage as:
- required for target behavior
- compatibility only
- redundant and removable
3. Introduce target-owned lookup as canonical recipe source for BuildCompositeByParts flows.
4. Add migration-safe fallback behavior for legacy files that still have reverse-link recipe IDs.
5. Add authoring-save behavior to avoid writing reverse recipe IDs on part objects (behind compatibility flag if needed).

### Validation Gate

1. Focused tests:
- `CompositeBuildActionTests`
- `ActionIntegrityRulesTests`
- `JsonExportServiceProjectStateTests`
2. Add new tests:
- BuildCompositeByParts succeeds when parts have no `compositeRecipeId` but target recipe lists them.
- BuildCompositeByParts validation still catches missing target/required-part IDs.
- Legacy project with reverse-link part recipe IDs remains load-compatible.
3. Full suite pass.

### Exit Criteria

1. BuildCompositeByParts runtime no longer depends on part-owned recipe IDs.
2. Recipe authority is unambiguous (target-owned).
3. Existing sample projects continue to load and run.

---

## Phase 2: Runtime Mapping Compatibility Layer

### Goals

1. Preserve behavior while transitioning runtime identity semantics.
2. Introduce explicit parent linkage representation in runtime scope nodes.

### Work

1. Extend runtime scope descriptors/nodes with parent linkage field (`runtimeParentScopeId` naming TBD).
2. Populate parent linkage during snapshot/bootstrap mapping and session construction.
3. Keep current runtime ID generation temporarily for compatibility while parent linkage becomes first-class.

### Validation Gate

1. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`
2. Playback strict-mode pass on recorded cases.
3. Full test suite pass.

### Exit Criteria

1. Runtime logic can resolve scope location via parent linkage without relying on path-encoded identity.
2. No functional regression in command processing/playback.

---

## Phase 3: Switch Authored Objects To ObjectId-As-RuntimeId

### Goals

1. Remove alternate runtime ID generation for authored objects.
2. Use authored `objectId` as runtime identity for authored objects.

### Work

1. Update native snapshot mapper and clean-runtime bootstrap so authored objects carry authored IDs through runtime descriptors.
2. Keep GUID generation only for truly runtime-born objects with no authored identity.
3. Update composite resolution code paths to assume authored IDs for authored entities.
4. Ensure diagnostics/logging surfaces authored IDs consistently.

### Validation Gate

1. Focused runtime gate:
- `GameManagerTests`
- `GameCommandProcessorFixtureTests`
- `GameCommandProcessorLinkedActionsTests`
- `GameSimulatorPlaybackRegressionTests`
- `CompositeBuildActionTests`
2. Full suite pass.
3. Manual spot-check: diagnostic IDs map to project JSON object IDs.

### Exit Criteria

1. No alternate runtime IDs are generated for authored objects.
2. Runtime behavior remains stable across playback and composite actions.

---

## Phase 4: Remove Legacy Runtime-ID Assumptions

### Goals

1. Remove dead translation logic and identity remap paths.
2. Keep only explicit parent linkage and canonical object IDs.

### Work

1. Delete no-longer-needed model->runtime remap dictionaries where authored objects now use `objectId` directly.
2. Simplify normalization code paths that previously converted composite IDs.
3. Tighten tests to fail on any authored-object runtime ID divergence.

### Validation Gate

1. Full test suite pass.
2. Focused regression rerun of composite flows and playback.
3. Build passes: `dotnet build .\StoryboardDesigner.slnx`.

### Exit Criteria

1. Identity model is single-source for authored objects.
2. Parent linkage handles location semantics.
3. No runtime/design-time ID drift remains.
4. Runtime-stable lookup artifacts for authored target identity (for example `CompositeTargetStableIds`) are removed from production validation/mapping paths.

---

## Data Repair Plan (Parallel Track)

### Goals

1. Fix existing corrupted sample/fixture payloads.
2. Keep clean-export artifacts aligned with corrected authored data.

### Work

1. Correct mismatched composite IDs in affected sample files (starting with ObjectPlayLevel2).
2. Regenerate corresponding clean-export sample artifacts.
3. Remove or normalize non-target part `compositeRecipeId` usage in sample data where safe.
4. Add fixture validation test that scans known sample projects for unresolved composite IDs.

### Validation Gate

1. Targeted sample-based tests pass.
2. Warnings no longer appear on corrected sample content.

---

## Risk Controls

1. Small PR slices by phase; do not combine Phase 2-4 in one patch.
2. Keep compatibility toggles while transitioning critical runtime paths.
3. Require focused runtime gate + full suite before advancing phase.
4. If a phase introduces >2 unrelated failures, pause and stabilize before continuing.

## Decision Log (Initial)

1. Canonical identity for authored objects: `objectId`.
2. Runtime placement semantics: explicit parent linkage field, not encoded identity.
3. Runtime-generated IDs allowed only for runtime-born objects with no authored source identity.
4. Validation rules ACT-010/ACT-011/ACT-012 are baseline guardrails during migration.
5. Composite recipe linkage is target-owned; reverse-link recipe IDs on parts are deprecated unless a compatibility case proves they are required.

## Current Checkpoint (2026-07-06)

1. Full regression suite is green after adding clean-runtime composite action ID normalization in `CleanRuntimeBootstrapSnapshotMapper`.
2. Compatibility is currently preserved by mapping clean-export authored IDs to runtime-resolved object IDs for composite target/part payloads.
3. This fix is a stabilizer, not the final identity model.

## Closeout Checkpoint (2026-07-07)

Technical closeout state:

1. Load/save/export ID auto-heal fallbacks were removed and replaced with fail-fast validation where required.
2. Break-composite restore flow was adjusted to optional/non-creating behavior.
3. Dead runtime restore-anchor synthesis chain was removed (`TryCreateQuantifiableRestoreAnchor` path and gateway passthroughs).
4. Legacy authored-object remap dictionary concepts were removed from both mapper paths while preserving recipe-based fallback.
5. Room scope descriptors in both mapper paths now emit `RoomId` and `ObjectId` as the same room identity value (initial consolidation step).
6. Full test regression gate is green after each major identity slice up to this checkpoint.

Deferred follow-up items (intentional):

1. Scope-tree identity course correction (`ScopeKind` + single canonical scope ID) is queued in:
2. `plans/active/SCOPEKIND_SINGLE_ID_COURSE_CORRECTION_PLAN.md`
3. Activation is deferred until this plan receives manual-testing signoff and formal closure.

Remaining step before closure:

1. Manual testing signoff from owner.
2. After signoff, flip this plan status to `Closed` and archive/rollup disposition in consolidated tracker.

## Closure Record (2026-07-07)

1. Manual testing signoff received.
2. Plan status flipped to `Closed`.
3. Successor work remains queued in `plans/active/SCOPEKIND_SINGLE_ID_COURSE_CORRECTION_PLAN.md` and is not part of this closed scope.

## Safe Stepwise Execution Playbook

Use the following implementation packet sequence. Do not skip validation gates between packets.

### Packet A: Stop New Authoring Drift (Phase 1)

Scope:
1. `StoryboardDesigner.App` only (editor/viewmodel/persistence write paths for composite action fields).

Changes:
1. Force composite action writes to use authored graph object IDs only.
2. Add save/reload tests proving authored IDs round-trip unchanged.
3. Add a targeted assertion that runtime-stable deterministic IDs are never persisted into authored composite fields.

Validation:
1. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "ActionIntegrityRulesTests|JsonExportServiceProjectStateTests|CompositeBuildActionTests"`
2. `dotnet build .\StoryboardDesigner.slnx`

Rollback criteria:
1. Any composite authoring scenario begins writing altered IDs after save/reload.
2. Any existing sample project requires manual ID translation just to remain editable.

### Packet B: Recipe Ownership Clarification (Phase 1B)

Scope:
1. `StoryboardDesigner.App` + `Storyboard.Shared` recipe-read paths.

Changes:
1. Introduce target-owned recipe lookup as canonical for BuildCompositeByParts.
2. Keep part-level `CompositeRecipeId` as compatibility-only fallback.
3. Stop writing reverse-link part recipe IDs during normal authoring saves (behind compatibility flag if needed).

Validation:
1. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "CompositeBuildActionTests|ActionIntegrityRulesTests|JsonExportServiceProjectStateTests"`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameSimulatorPlaybackRegressionTests"`

Rollback criteria:
1. Any legacy project with reverse-link IDs fails to load or run playback.
2. BuildCompositeByParts requires part reverse-links to succeed in non-legacy authored content.

### Packet C: Parent Linkage Foundation (Phase 2)

Scope:
1. Runtime descriptors/session construction only.

Changes:
1. Add explicit runtime parent linkage field.
2. Populate linkage in both native snapshot and clean bootstrap mapping.
3. Keep runtime generated IDs temporarily; no identity model switch in this packet.

Validation:
1. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`
2. Full suite pass.

Rollback criteria:
1. Command resolution or traversal semantics regress.
2. Playback strict mode diverges from recordings.

### Packet D: ObjectId-As-RuntimeId Cutover (Phase 3)

Scope:
1. Runtime identity assignment for authored objects only.

Changes:
1. Switch authored objects to carry authored `objectId` through runtime.
2. Restrict new GUID generation to runtime-born objects.
3. Keep temporary compatibility translation seams until Packet E.

Validation:
1. Focused runtime gate + `CompositeBuildActionTests`.
2. Full suite pass.
3. Manual diagnostic spot-check against authored JSON IDs.

Rollback criteria:
1. Any authored object identity mismatch appears between runtime diagnostics and project JSON.
2. Composite target/part resolution requires remap shims to behave correctly.

### Packet E: Legacy Translation Removal (Phase 4)

Scope:
1. Remove transitional remap code and stale stable-ID assumptions for authored objects.

Changes:
1. Delete authored-object ID remap dictionaries no longer needed.
2. Remove compatibility branches that only existed for alternate runtime IDs.
3. Keep focused regression tests that would fail on renewed divergence.

Validation:
1. Full suite + focused runtime gate.
2. `dotnet build .\StoryboardDesigner.slnx`

Rollback criteria:
1. Any regression that requires restoring authored-object ID translation logic.

## Phase 1B Inventory Snapshot (Initial)

Initial inventory from current code scan:

1. Runtime-critical composite ID translation seams:
- `StoryboardDesigner.App/GameServices/ProjectModelRuntimeSnapshotMapper.cs`
- `Storyboard.Shared/GameServices/Bootstrap/CleanRuntimeBootstrapSnapshotMapper.cs`

2. Authoring write/read surfaces for composite IDs:
- `StoryboardDesigner.App/Views/RoomActionEditorDialog.xaml.cs`
- `StoryboardDesigner.App/ViewModels/MainWindowViewModel.ProjectExplorer.cs`
- `StoryboardDesigner.App/Services/JsonExportService.cs`
- `StoryboardDesigner.App/Models/CommandAction.PayloadProjections.cs`

3. Validation/guardrails already enforcing target/part integrity:
- `StoryboardDesigner.App/Validation/Rules/Actions/CompositeActionMissingTargetRule.cs`
- `StoryboardDesigner.App/Validation/Rules/Actions/BuildCompositeByPartsMissingTargetObjectRule.cs`
- `StoryboardDesigner.App/Validation/Rules/Actions/BuildCompositeByPartsTargetRecipeMismatchRule.cs`
- `StoryboardDesigner.App/Validation/Rules/Actions/CompositeActionMissingRequiredPartRule.cs`

4. Recipe-ownership pressure points for Phase 1B follow-up classification:
- `StoryboardDesigner.App/Models/GameObject.cs` (`CompositeRecipeId` on object model)
- `StoryboardDesigner.App/ViewModels/MainWindowViewModel.cs` (recipe choice/lookup)
- `StoryboardDesigner.App/Services/JsonExportService.cs` (serialization of object/action recipe fields)

## Lock-Off Decisions (Must Confirm Before Packet A)

Status legend:
1. `[Proposed]` recommended default from current architecture direction.
2. `[Approved]` confirmed decision.
3. `[Deferred]` intentionally postponed with fallback noted.

### A. Identity Contract

1. Authored object runtime identity is authored `objectId` across all hosts.
- Proposed answer: Yes. No exceptions for authored objects.
- Status: [Proposed]

2. Runtime-born objects must remain distinguishable from authored objects.
- Proposed answer: Yes. Keep generated IDs for runtime-born objects and mark them as runtime-born in runtime state/metadata.
- Status: [Proposed]

3. Deterministic stable IDs for authored identity are transitional-only.
- Proposed answer: Yes. Allow only as temporary compatibility in mapping layers until Packet E removal.
- Status: [Proposed]

### B. Parent Linkage Model

1. Canonical parent linkage field naming and scope.
- Proposed answer: Introduce `RuntimeParentScopeObjectId` (runtime-only field; not part of authored file schema).
- Status: [Proposed]

2. Nodes requiring parent linkage.
- Proposed answer: Require on runtime object nodes; keep optional/null for non-object scope nodes (planet/country/area/room roots).
- Status: [Proposed]

3. Move semantics.
- Proposed answer: Runtime moves update parent linkage only; object identity never mutates.
- Status: [Proposed]

### C. Composite Recipe Ownership

1. Canonical recipe authority.
- Proposed answer: Target-owned recipe metadata is authoritative immediately for new behavior.
- Status: [Proposed]

2. Non-target part `CompositeRecipeId` handling.
- Proposed answer: Read as compatibility-only during migration; stop writing it in normal authoring saves after Packet B.
- Status: [Proposed]

3. Conflict resolution.
- Proposed answer: Target recipe wins over part reverse-link metadata in all conflicts.
- Status: [Proposed]

### D. Authoring Persistence Rules

1. Save behavior on unresolved composite references.
- Proposed answer: Block save with validation error for invalid composite IDs in edited content paths; retain existing warning-only behavior for untouched legacy files until migrated.
- Status: [Proposed]

2. UI ID source restrictions.
- Proposed answer: Composite editor selection surfaces authored graph IDs only; never runtime-resolved IDs.
- Status: [Proposed]

3. Legacy correction strategy.
- Proposed answer: Provide explicit repair path (manual command/tooling), not silent auto-rewrite on normal save.
- Status: [Proposed]

### E. Clean Export And Schema

1. Schema bump requirement.
- Proposed answer: No schema bump for Packets A-D if field set remains additive/compatible; revisit only if field contract changes.
- Status: [Proposed]

2. Compatibility fields during transition.
- Proposed answer: Keep compatibility fields until Packet D validates across samples/playback; remove in Packet E.
- Status: [Proposed]

3. Contract removal timing.
- Proposed answer: Single controlled removal window in Packet E after focused and full regression gates pass.
- Status: [Proposed]

### F. Diagnostics And Supportability

1. Authored ID visibility in diagnostics.
- Proposed answer: Diagnostics for authored objects must emit authored `objectId` consistently.
- Status: [Proposed]

2. Human-readable context.
- Proposed answer: Include scope/object name context where available, without replacing canonical IDs.
- Status: [Proposed]

3. Migration tracing.
- Proposed answer: Add temporary trace counters/log markers for remap path usage; remove in Packet E.
- Status: [Proposed]

### G. Migration And Data Repair

1. Mandatory repair set baseline.
- Proposed answer: Start with `ObjectPlayLevel2`, then all playback-manifest-backed samples.
- Status: [Proposed]

2. Repair mechanism.
- Proposed answer: Explicit repair workflow (script/tool + review), not implicit load-time mutation.
- Status: [Proposed]

3. Snapshot policy for repaired fixtures.
- Proposed answer: Refresh only intentionally with `UPDATE_PLAYBACK_SNAPSHOTS=1` and include rationale in PR notes.
- Status: [Proposed]

### H. Rollout And Risk Controls

1. Feature flag policy.
- Proposed answer: Feature flags for Packet D cutover and any Packet B behavior that can alter legacy semantics.
- Status: [Proposed]

2. Packet advancement gate.
- Proposed answer: Do not advance packet unless focused gate + full suite + per-change playback smoke gate are green.
- Status: [Proposed]

3. Required approvals.
- Proposed answer: Engineering owner + runtime behavior owner sign-off at Packet B and Packet D boundaries.
- Status: [Proposed]

## Decision Log (Approvals)

Record approved outcomes as they are confirmed in discussion:

1. Q1 (Authored objectId as runtime identity for authored objects): Approved with caveat. Any authored object with a valid, non-empty, unique `objectId` uses that same ID at runtime. Runtime-born objects are handled separately under Q2. If runtime detects authored ID integrity issues (missing/duplicate/invalid IDs), it must identify them, log diagnostics, and exit without performing runtime repair.
2. Q2 (Runtime-born identity separation): Approved with modification. Runtime-born objects use the same ID pool as authored objects and must carry a simple runtime-born marker flag.
3. Q3 (Deterministic stable IDs for authored identity): Approved. Stable IDs are compatibility-only during migration and must be removed from authored-object identity paths by Packet E.
4. Q4 (Parent linkage field model): Approved and finalized with explicit semantics. Both project design JSON and clean runtime JSON persist `designTimeParentObjectId`. Runtime in-memory objects do not expose that property name; they use `runtimeParentObjectId`. At runtime startup/bootstrap, runtime WILL copy `designTimeParentObjectId` into initial `runtimeParentObjectId` values. After startup, `runtimeParentObjectId` is authoritative and may diverge as objects move through the runtime scope tree.
	 - Clarifying design principle: there is exactly one parent linkage representation per layer to avoid ambiguity.
		 - Design/authored layer: `designTimeParentObjectId`.
		 - Runtime in-memory layer: `runtimeParentObjectId`.
	 - Outcome: each layer has a single source of truth for parent linkage, reducing wrong-field usage and identity/linkage drift bugs.
5. Q5 (Parent linkage coverage): Approved with modification. Parent linkage is a required concept in both design-time and runtime models; exact node-level coverage details remain aligned under the Q4 follow-up decision.
6. Q6 (Move semantics): Approved. Runtime moves update parent linkage while object identity remains immutable.
7. Q7 (Composite recipe authority): Approved. Target-owned recipe metadata is the canonical source of truth for composite behavior.
8. Q8 (Non-target part `CompositeRecipeId` handling): Approved with modification. Part objects must not carry direct recipe knowledge; reverse-link recipe property usage is removed fully rather than retained as compatibility behavior.
9. Q9 (Conflict resolution): Approved. Where legacy reverse-link values exist, target recipe metadata always wins.
10. Q10 (Save behavior for invalid composite references): Approved with modification. UX should strongly encourage correction before save, but users can always explicitly override and save.
11. Q11 (UI source of composite IDs): Approved. Composite editor selection controls must use authored graph IDs only and never runtime-resolved IDs.
12. Q12 (Legacy correction mode): Approved with modification. No user-facing legacy migration path is required. During implementation, only minimal in-flight compatibility needed to keep the active test suite green is allowed; after feature completion, legacy correction pathways are removed.
13. Q13 (Schema bump policy): Approved with modification. Favor no schema bump while completing this feature; if a breaking contract change becomes unavoidable, perform a single coordinated update tied to fixture/test updates rather than maintaining legacy compatibility modes.
14. Q14 (Compatibility fields during transition): Approved with modification. Compatibility fields are temporary implementation scaffolding only and may exist solely to keep tests green during the work; they must be removed by completion.
15. Q15 (Contract removal timing): Approved with modification. Remove compatibility contract elements as part of feature completion (final stabilization window), with no post-completion legacy compatibility commitment.
16. Q16 (Diagnostic ID standard): Approved. Diagnostics for authored objects must emit authored `objectId` as canonical identity.
17. Q17 (Diagnostic readability context): Approved. Diagnostics should include human-readable scope/object context in addition to canonical IDs.
18. Q18 (Migration remap tracing): Approved with modification. Keep tracing lightweight and temporary for in-flight validation only; remove tracing and remap diagnostics with final compatibility cleanup.
19. Q19 (Mandatory repair set): Approved. Start with `ObjectPlayLevel2`, then expand to all playback-manifest-backed sample projects.
20. Q20 (Repair mechanism mode): Approved with modification. No general repair workflow/tooling is required; only direct fixture/data updates needed by the repository test corpus are in scope.
21. Q21 (Snapshot refresh policy for repaired fixtures): Approved. Refresh playback snapshots only intentionally with `UPDATE_PLAYBACK_SNAPSHOTS=1` and documented PR rationale.
22. Q22 (Feature flag policy): Approved with modification. Feature flags are optional and should be used only if needed to keep tests green during implementation; no long-term flag-controlled legacy mode should remain at completion.
23. Q23 (Packet advancement gate): Approved. Advancing packets requires focused regression gate pass, full suite pass, and per-change playback smoke gate pass.
24. Q24 (Required sign-off owners): Approved. Packet B and Packet D boundaries require both engineering owner and runtime behavior owner sign-off.

## Immediate Next Steps

1. Execute Packet A in a single PR slice with focused tests and explicit save/reload assertions.
2. In parallel, finish Phase 1B classification as a tracked table (required vs compatibility vs removable) before any Packet B behavior changes.
3. Keep clean-runtime normalization in place until Packet D cutover is complete and verified.

## Approved High-Level Phase Plan (2026-07-07)

This is the approved execution baseline for implementation.

### Phase 0: Plan Normalization

1. Align active plan text to approved decisions only.
2. Keep per-change playback smoke gate mandatory.

### Phase 1: Authoring ID Purity

1. Ensure designer editing/save paths persist authored graph IDs only for composite references.
2. Ensure composite selectors source authored IDs only.
3. Add save/reload assertions that authored composite IDs round-trip unchanged.

### Phase 2: Recipe Ownership Cutover

1. Make target-owned recipe metadata authoritative.
2. Remove part-side recipe knowledge from behavior/persistence paths.
3. Keep tests green via temporary in-flight compatibility only where required during implementation.

### Phase 3: Parent Linkage Model Implementation

1. Persist `designTimeParentObjectId` in project design JSON and clean runtime JSON.
2. Use `runtimeParentObjectId` in runtime in-memory objects only.
3. At runtime startup/bootstrap, copy `designTimeParentObjectId` into initial `runtimeParentObjectId`.
4. After startup, `runtimeParentObjectId` is authoritative and updates on movement.

### Phase 4: Runtime Identity Cutover

1. Use authored `objectId` as runtime identity for authored objects.
2. Keep runtime-born objects in same ID pool with explicit runtime-born marker.
3. On authored ID integrity errors (missing/duplicate/invalid), runtime logs diagnostics and exits with no runtime repair.

### Phase 5: Temporary Compatibility Removal

1. Remove temporary remap logic, temporary compatibility fields, temporary tracing, and short-lived flags.
2. Keep only canonical end-state model.
3. No long-term legacy compatibility or migration path remains after completion.

### Phase 6: Final Stabilization And Sign-Off

1. Run full validation matrix.
2. Confirm diagnostics emit canonical authored IDs with readable context.
3. Enforce packet gate criteria and sign-off boundaries.
4. Deferred review item: decide whether `runtimeParentObjectId` should use a Player-root sentinel ID when parent scope is Player, or remain strictly derived from parent `ObjectId`/`RoomId` (current behavior). Do not implement sentinel behavior before this final review decision.
5. Deferred review item: scrutinize why `CleanScopeNodeBase` exposes both `RoomId` and `ObjectId`, and decide whether parent/scope identity should be consolidated to a single canonical ID field. No implementation change before final review decision.
6. Deferred review item: perform a deep post-refactor audit of all remaining ID remapping paths to confirm where remapping is still required versus accidental legacy carryover; document justified keep-cases and remove any residual unnecessary remapping discovered in that review.