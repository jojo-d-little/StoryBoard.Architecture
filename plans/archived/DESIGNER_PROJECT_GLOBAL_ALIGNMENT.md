# Designer Project Global Alignment Plan

Status: Closed
Owner: Designer persistence alignment workflow
Priority: Complete before any additional schema contract work
Last updated: 2026-08-06

## Execution Progress
1. Step 1 Ownership Charter Slice: completed (ownership matrix and migration/cutover rules recorded).
2. Step 2 Naming Alignment Slice: completed (global-sidecar to global-node naming completed across interfaces, services, viewmodels, and tests with behavior unchanged).
3. Step 3 Structural Alignment Slice: completed (ProjectGlobalNodeDto now inherits DesignerScopeNodeDtoBase, ScopeKind is pinned to Global, build/tests validated).
4. Step 4 Ownership Enforcement Slice: completed (global-owned data now has canonical global-node ownership across vocabulary/mappings, global available actions, global variables, shared variables, global procedure ids, planet ids, and global object/template/room-template/base-object references).
5. Step 5 Save-Path Canonicalization Slice: completed (root serialization omits global-owned fields; canonical writes for global-owned data target the global node and associated scope sidecars only).
6. Step 6 Load-Path Hardening Slice: completed (global node is authoritative when present, mixed-source no-merge behavior is enforced across field groups, and diagnostics IDs remain locked and field-aware).
7. Step 7 Sample Migration Slice: completed (tools/migrate-samples.ps1 executed successfully across all repository samples; report: Backups/SampleMigration-20260806-065228/migration-report.json; post-migration build/app-tests/playback gate are green).
8. Step 8 Required Global Node Slice: completed (normal OpenProject flow now fails fast when global node is missing or malformed; diagnostics report still emits locked IDs, with missing global node elevated to load-time error severity for failed loads).
9. Step 9 Root DTO Cleanup Slice: completed (ProjectAuthoringRootDto reduced to project-header/app-level fields only; global-owned compatibility members removed; JsonExportService save/load paths source global-owned fields from ProjectGlobalNodeDto only; inline procedures and root global-owned fallback reads retired).
10. Step 10 Test Hardening Slice: completed (focused and full regression coverage updated and passing under strict global-node-required behavior, including sample round-trip compatibility and playback gate).
11. Step 11 Schema Contract Follow-On Slice: deferred by agreement to a separate follow-on plan after this closeout.

## Field Cutover Checklist
1. commandVerbs/directionals/directionalTraversalMappings
- Canonical owner: ProjectGlobalNodeDto
- Save behavior: canonical-only in global node, omitted from root
- Load behavior: strict global-node-required load; no root fallback
- Root DTO member removal: completed
2. globalAvailableGameActions
- Canonical owner: ProjectGlobalNodeDto
- Save behavior: canonical-only in global node, omitted from root
- Load behavior: strict global-node-required load; no root fallback
- Root DTO member removal: completed
3. gameProperties (project global variable definitions)
- Canonical owner: ProjectGlobalNodeDto (via DesignerScopeNodeDtoBase)
- Save behavior: canonical-only in global node, omitted from root
- Load behavior: strict global-node-required load; no root fallback
- Root DTO member removal: completed
4. globalProcedureIds
- Canonical owner: ProjectGlobalNodeDto
- Save behavior: canonical-only in global node, omitted from root
- Load behavior: strict global-node-required load; no root fallback
- Root DTO member removal: completed
5. sharedVariables
- Canonical owner: ProjectGlobalNodeDto
- Save behavior: canonical-only in global node, omitted from root
- Load behavior: strict global-node-required load; no root fallback
- Root DTO member removal: completed
6. planetIds
- Canonical owner: ProjectGlobalNodeDto
- Save behavior: canonical-only in global node, omitted from root
- Load behavior: strict global-node-required load; no root fallback
- Root DTO member removal: completed
7. gameObjects/objectTemplates/roomTemplates/baseObjects references
- Canonical owner: ProjectGlobalNodeDto ids plus authoring object sidecars
- Save behavior: canonical ids in global node, omitted from root
- Load behavior: strict global-node-required load; no root fallback
- Root DTO member removal: completed
8. procedures
- Canonical owner: procedure sidecars + global-node-owned globalProcedureIds routing
- Save behavior: externalized procedure sidecars, no root inline payload
- Load behavior: strict global-node-required load; no root inline fallback
- Root DTO member removal: completed

## Closeout Remaining
1. None. Exit criteria accepted and this plan is closed.

## Closeout Decision Record
1. Schema/contract follow-on is explicitly out of scope for this plan closeout and will be handled in a separate future plan.

## Goal
Promote the current globals sidecar into the canonical Global node document for designer persistence, remove ambiguous root/sidecar duplication, and harden single ownership boundaries before further schema-contract expansion.

## JSON Format Change Policy
1. Preserve current JSON wire format by default while performing naming/ownership cleanup.
2. If JSON shape/path/ownership changes become necessary, treat them as explicit compatibility events, not incidental side effects.
3. Every intentional JSON format change must include:
- a migration note describing old vs new layout
- loader compatibility behavior for existing files
- sample project migration and verification evidence

## Preconditions
1. No new schema-contract migration slices start until this plan is complete.
2. Keep changes incremental and reversible.
3. Run build and tests after each slice.

## Agreed Direction
1. The current globals sidecar is critical and canonical, not optional metadata.
2. Global node data must have single ownership.
3. Root project file should retain project-header/app-level responsibilities only.
4. Fallback behavior is transitional and must be removed after migration window.

## Ownership Charter (Step 1)

### Canonical Owner Matrix
1. `ProjectAuthoringRootDto` owns project header and app-level persistence settings:
- project identity/header fields (for example name, hideEmptyConfiguration)
- autosave and designer/simulator host settings
- project-level validation ignore lists that are not global-scope node data
2. `ProjectGlobalNodeDto` owns canonical Global scope node data:
- command verbs and directionals
- directional traversal mappings
- global available actions
- global variables/shared variables
- procedures and globalProcedureIds
- planetIds
- global object/template/base object/room template references

### Precedence Rules (Migration Window)
1. For global-owned fields, `ProjectGlobalNodeDto` is authoritative when present.
2. Root fallback reads are compatibility-only and must emit diagnostics.
3. No merge behavior between root and global node for global-owned fields.
4. When both sources contain values, global node wins and conflict diagnostics are emitted.

### Save Rules (Migration Window)
1. Canonical writes must target `ProjectGlobalNodeDto` for global-owned fields.
2. Root mirror-write is allowed only for approved temporary compatibility fields.
3. Any root mirror-write path must be explicitly removable at migration-window cutoff.

### Cutover Rules (Post Migration Window)
1. `ProjectGlobalNodeDto` is required for normal load.
2. Root fallback reads for global-owned fields are removed.
3. Root mirror-writes for global-owned fields are removed.
4. Missing/malformed global node is an actionable load error.

## Open Design Questions To Lock
1. Global node file identity: keep existing globals file name/path for compatibility, or rename file on disk as part of this effort?
2. Required-global-node enforcement timing: immediate cutover after migration slice, or one release with compatibility fallback first?
3. Fallback policy during migration window: warn-only on root fallback usage, or fail in strict mode with opt-in compatibility flag?
4. Field canonicalization policy: when both root and global node contain values during migration, should global node always win with no merge?
5. Planet topology ownership: move `planetIds` to global node only, or temporarily mirror-write to root until migration completes?
6. Procedure ownership split: should `procedures` and `globalProcedureIds` both become global-node-only in the same slice, or stage separately?
7. Shared variable ownership: should `sharedVariables` be migrated in the same slice as procedures/planetIds, or as a dedicated migration step?
8. Diagnostics contract: what exact warning/error codes and messages are required for missing global node, fallback usage, and conflicting values?
9. Sample migration strategy: one-time scripted migration for all repository samples, or save-on-load migration with committed updated samples?
10. Contractization threshold: what explicit completion criteria must be met before starting the schema-governed Global node follow-on?

## Locked Design Decisions
1. Keep current global node file name/path during this cleanup phase; do not rename the on-disk file yet.
2. Use one migration release window with compatibility fallback, then enforce required-global-node.
3. During migration: strict-by-default in tests/CI, compatibility-by-default in interactive app load, with explicit diagnostics in both modes.
4. No merge policy for global-owned fields: global node always wins when both root and global values exist.
5. `planetIds` becomes global-node canonical now; mirror-write to root only during migration window, then remove.
6. Move `procedures` and `globalProcedureIds` together in the same ownership-enforcement slice.
7. Move `sharedVariables` in the same ownership-enforcement slice as `procedures`/`globalProcedureIds`/`planetIds`.
8. Adopt diagnostics contract:
 - `global-node.missing.v1` (warning in migration window, error after cutover)
 - `global-node.fallback-used.v1` (warning)
 - `global-node.conflict-root-vs-global.v1` (warning)
 - `global-node.malformed.v1` (error)
 - `global-node.legacy-root-write.v1` (info/warning in migration window)
9. Use one-time scripted migration for repository samples and commit updated sample artifacts; keep save-on-load migration only as a safety net for external legacy projects.
10. Contractization may begin only after this plan exit criteria are fully met (including sample migration validation and removal of ambiguous ownership/fallback behavior).

## Locked Naming Map
1. `ProjectGlobalsSidecarDto` -> `ProjectGlobalNodeDto`.
2. `ProjectFileDto` -> `ProjectAuthoringRootDto`.
3. `BuildProjectGlobalsFilePath` -> `BuildProjectGlobalNodeFilePath` (code symbol rename; keep on-disk file name/path unchanged during compatibility window).
4. Local variables/usages named `globalsSidecar` -> `globalNode` in persistence flow.

## Naming Rollout Order
1. Apply global-node naming (`ProjectGlobalsSidecarDto` -> `ProjectGlobalNodeDto`) in early alignment slices.
2. Apply root naming (`ProjectFileDto` -> `ProjectAuthoringRootDto`) in a later slice within this same plan after ownership/fallback hardening is stable.
3. Keep wire format/backward compatibility behavior intact while symbol renames are introduced.

## Step-by-Step Execution
1. Ownership Charter Slice
- Add explicit ownership matrix to project planning notes and developer docs.
- Declare canonical owner for each currently duplicated field.
- Mark root fallback reads as temporary migration behavior.

2. Naming Alignment Slice
- Rename ProjectGlobalsSidecarDto to ProjectGlobalNodeDto.
- Keep wire format and file path behavior unchanged in this slice.
- Update references, comments, and diagnostics wording only.

3. Structural Alignment Slice
- Make ProjectGlobalNodeDto inherit from DesignerScopeNodeDtoBase.
- Keep ScopeKind pinned to Global.
- Preserve current behavior while aligning type semantics.

4. Ownership Enforcement Slice
- Move canonical ownership to ProjectGlobalNodeDto for:
  - command verbs
  - directionals
  - directional traversal mappings
  - global available actions
  - sharedVariables
  - procedures
  - globalProcedureIds
  - planetIds
  - global objects/templates/base objects/room template references
- Keep root duplicates readable only for migration fallback.

5. Save-Path Canonicalization Slice
- Save canonical global-node-owned fields only via ProjectGlobalNodeDto.
- Stop writing competing values in ProjectAuthoringRootDto except migration-specific placeholders.
- Keep deterministic output ordering.

6. Load-Path Hardening Slice
- If global node document exists, it is authoritative.
- If fallback is used, emit explicit migration diagnostics.
- Prevent silent mixed-source ambiguity.

7. Sample Migration Slice
- Define and run migration steps for repository sample projects that rely on old root/sidecar distribution.
- Re-save or migrate sample projects to canonical global-node ownership where required.
- Verify migrated sample projects load, save, and clean-export without drift.

8. Required Global Node Slice
- After migration window, require Global node document for normal load.
- Missing/corrupt global node becomes actionable load error (or explicit migration mode only).

9. Root DTO Cleanup Slice
- Reduce ProjectAuthoringRootDto to project-header/app-level fields only.
- Remove duplicated global-node-owned fields and dead fallback code.

10. Test Hardening Slice
- Add/update tests for:
  - round-trip save/load with canonical global node ownership
  - fallback diagnostics during migration window
  - required-global-node behavior after hardening
  - no-duplication guarantees for canonical fields
  - sample project migration compatibility

11. Schema Contract Follow-On Slice
- Only after this plan is complete, evaluate introducing schema-governed contract coverage for Global node shape.
- Keep this as a separate phase to isolate risk.

## Validation Gate Per Slice
1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"

## Exit Criteria
1. Global node document is canonical and required (post-migration).
2. Single ownership exists for all previously duplicated fields.
3. Root project DTO no longer carries global-node-owned data.
4. Sample projects are migrated/validated against the finalized ownership layout.
5. Build, focused playback gate, and app test suite are green.
6. Team agrees cleanup is complete and schema-contract follow-on can begin.