# Runtime Export File-Per-Scope Plan

## 0. Current Status (2026-08-02)
- Completed and archive-ready: file-per-scope runtime export rollout is complete for current scope, snapshot baselines were refreshed to current runtime contract/output shape, and full test suite is green.
- Completed: runtime naming cutover to GameRuntimeJson and sbr.runtime file suffixes.
- Completed: image path semantics token migration to runtimeExportRelative.
- Completed: cross-project regression sweep (App, Shared, Simulator, smoke tests) after migration updates.
- Completed: runtime export index HTML emission at runtime root as a non-breaking first artifact for file-per-scope rollout.
- Completed: writer now emits scope-kind room node files under GameRuntimeJson/Room using uppercase Guid D filenames with .runtime.json suffix.
- Completed: writer now emits scope-kind area node files under GameRuntimeJson/Area using uppercase Guid D filenames with .runtime.json suffix.
- Completed: writer now emits scope-kind country node files under GameRuntimeJson/Country using uppercase Guid D filenames with .runtime.json suffix.
- Completed: writer now emits scope-kind planet node files under GameRuntimeJson/Planet using uppercase Guid D filenames with .runtime.json suffix.
- Completed: shared reader now falls back to GameRuntimeJson/Room/*.runtime.json when legacy *.sbr.runtime.rooms sidecar folder is absent.
- Completed: shared reader now falls back to GameRuntimeJson/Planet/*.runtime.json for project planet hierarchy when scope-kind planet nodes are present.
- Completed: shared reader now composes scope-kind Country/Area nodes by explicit referenced ids from parent hierarchy.
- Completed: orphan scope-kind Country/Area nodes are ignored by loader and surfaced via load diagnostics (rule: scope-kind.orphan.v1).
- Completed: orphan scope-kind diagnostics CSV emission is regression-covered with deterministic ordering assertions.
- Decision: keep shared reader dual-shape compatibility for the full duration of this plan's transition window; remove legacy shape compatibility at plan closeout.
- Completed: sample authored projects were regenerated to current runtime export shape (scope-kind folders present alongside transition compatibility artifacts).

Plan closeout removal checklist (legacy compatibility):
- Remove legacy room sidecar-first loading path and keep scope-kind Room node loading as the only reader path.
- Remove transition fallback branches that read legacy embedded hierarchy shapes when scope-kind files are available.
- Remove or rewrite tests that assert legacy compatibility behavior; keep and expand scope-kind-only coverage.
- Keep architecture guardrail validations green during the removal cut:
	- `dotnet build .\StoryboardDesigner.slnx`
	- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`
	- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`
- Confirm runtime export samples and fixtures no longer depend on legacy shape artifacts.
- Record completion of compatibility removal in this status section and move historical notes to archived plan material.

## 1. Goal
Realign runtime export layout away from deep containment and toward one runtime node per file by default.

Primary default rule:
- Any exported runtime node that has a scope node id gets its own file.
- File names include both scope kind and scope node id.
- Runtime export root contains one folder per scope kind.

## 2. Why this Plan Exists
Current runtime export trends toward embedding large nested payloads (especially room-owned object payloads), which creates very large room files and raises review, diff, and maintenance cost.

This plan shifts the disk shape to improve:
- reviewability
- incremental diffs
- file size distribution
- operational diagnostics and toolability

## 3. Non-Goals
- No runtime behavior change in command processing or gameplay semantics.
- No redesign of scope model itself.
- No immediate breaking removal of current v1 runtime export reader path without a compatibility window.

## 4. Current State Summary
Observed current runtime export package structure:
- root runtime project file
- root runtime navigation file
- runtime room sidecar folder containing room files
- assets folder with shared staged images and manifest
- runtime-generated sidecars for load diagnostics and direct-command echoes

Observed containment pattern to unwind:
- room files include full game object payloads, not child references only.

## 5. Target Structure (Default)
Under runtime export root:
- one root project index file (contract entry point)
- one folder per scope kind
- each scope-kind folder stores files named by scope kind plus scope node id

Example conceptual layout:
- runtime root
- runtime root / project index
- runtime root / ScopeKind-Planet
- runtime root / ScopeKind-Country
- runtime root / ScopeKind-Area
- runtime root / ScopeKind-Room
- runtime root / ScopeKind-GameObject
- runtime root / assets

Behavior payload note:
- Procedures and actions remain embedded in their owning node payloads (not standalone scope-kind node files in this phase).

Node payload guidance:
- parent nodes contain arrays of child ids (and optional ordered child id arrays when ordering matters)
- parent nodes do not inline full child objects by default

## 6. Naming Convention (Draft)
Default file naming:
- {ScopeKind}-{ScopeNodeId}.runtime.json

Rules:
- ScopeKind token comes directly from runtime scope kind enum token.
- ScopeNodeId uses canonical guid text form.
- Preserve stable casing policy across writer and tests.

## 7. Folder Convention (Draft)
Default folder naming:
- one folder per scope kind token exactly, unless a filesystem-safe normalization rule is required.

Rules:
- avoid deep nesting by parent hierarchy
- keep homogeneous node types together
- keep assets separate from node payloads

## 8. Explicit Exceptions (Initial)
Exceptions to file-per-scope default (land at export root unless otherwise noted):
1. Project entry file (single root file).
2. Navigation aggregate (candidate: keep as separate aggregate file for phase 1).
3. Asset manifest and binary assets (remain under assets).
4. Runtime-generated diagnostics sidecars (temporary root placement unless moved to diagnostics folder).
5. Runtime-generated direct-command-echo sidecar (temporary root placement unless moved to runtime-state folder).

Potential follow-on exception category:
- tiny immutable lookup tables that are contract-level metadata rather than scope nodes.
- procedure/action behavior payloads that are owner-scoped and id-referenced, but not modeled as runtime scope nodes.

## 9. Data Shape Shift
For all scope-node relationships:
- replace inline child object arrays with child id arrays.

Where helpful, keep both in compatibility window:
- old inline property accepted by reader
- new child id property preferred by writer

## 10. HTML Index Feature (Convenience)
Add export-root html index for human browsing.

Intent:
- user/developer convenience only
- not consumed by runtime loader

Minimum content:
- fully qualified name per exported node
- scope kind
- scope node id
- relative link to node json file

Quality expectations:
- static file, no script dependencies required
- works from file explorer open
- stable sort order for predictable diffs

## 11. Contract and Versioning Direction
Because this changes file layout and containment semantics, treat as a contract-shape evolution.

Proposed approach:
- introduce runtime export schema version 1.1 (or 2.0 if strict break decision is made)
- maintain reader compatibility with prior package shape during migration window
- enforce explicit version detection in runtime loader entry path

## 12. Migration Strategy (Phased)
Phase 0: Design lock
- finalize naming, folder, and reference rules
- finalize exception list

Phase 1: Additive writer support
- writer emits new structure behind a controlled switch or version flag
- optionally dual-write old and new structures in transition

Phase 2: Reader compatibility
- reader supports both old containment and new id-reference package
- runtime selects parse path by schema version and file presence

Phase 3: Index + diagnostics layout
- generate html index
- decide final home for runtime-generated sidecars

Phase 4: Samples and fixtures
- one-time restructure of sample exports and snapshots
- update test baselines intentionally

Phase 5: Cutover
- default to new format
- deprecate old format write path
- keep old format read path for defined window

## 13. Risks and Mitigations
Risk: reference integrity drift (missing child file, orphan ids)
- Mitigation: export-time integrity validation and fail-fast report.

Risk: load-time perf impact from many small files
- Mitigation: deterministic load ordering, lazy read where safe, and targeted perf checks.

Risk: merge churn from broad fixture updates
- Mitigation: staged fixture migration by sample set and explicit baseline refresh process.

Risk: runtime-generated sidecars polluting contract root
- Mitigation: decide dedicated diagnostics/runtime-state subfolders early.

## 14. Validation and Test Impact
Need focused coverage for:
- exporter emits file-per-scope layout
- child references resolve deterministically
- loader accepts both old and new layouts during migration
- html index links are valid and complete
- assets manifest references remain consistent

Guardrail focus:
- runtime/shared boundary tests
- clean export deterministic ordering tests
- sample snapshot tests

## 15. Design Questions (Lock List)
1. What exact schema version bump policy applies: 1.1 additive or 2.0 breaking?
2. Should the project entry file remain named with current clean suffix or move to a new canonical root filename?
3. Should navigation remain a single aggregate file or be decomposed by area scope files?
4. Is folder naming exactly enum token text, or normalized for filesystem safety?
5. Is file naming exactly kind-id.runtime.json, or should suffix remain clean.node.json for consistency?
6. What guid format is canonical in filenames (with hyphens vs no hyphens)?
7. Do we include node display name in filename for readability, or keep id-only for stability?
8. For child references, do we store id only, or id plus expected child scope kind?
9. How do we represent ordered children where order matters: ordered id list only, or id plus explicit order metadata?
10. Should parent files include relative path hints to child files, or should loader derive path by kind/id convention only?
11. Where should runtime-generated sidecars land: root, diagnostics folder, or runtime-state folder?
12. Should direct-command-echo sidecar stay mutable in-place, or move out of export package entirely?
13. Should load diagnostics remain csv, or switch to json plus optional csv export?
14. Do we require full backwards read compatibility indefinitely, or for a bounded deprecation window?
15. During transition, do we dual-write both old and new structures or write only new and keep old reader support?
16. For assets, do we keep current shared hash bucket only, or add per-scope convenience mirrors (links only)?
17. Should html index include unresolved/missing reference warnings from export-time checks?
18. Should html index be a single page only, or include per-scope sub-index pages?
19. Should html index link text be fully qualified name only, or include kind and id badges?
20. What is the fully qualified name canonical format across project, world, and object nesting?
21. How strict should exporter be on invalid references: fail export vs emit partial with diagnostics?
22. What is the one-time sample migration strategy: in-place rewrite vs regenerate from authoring source and re-baseline?

## 16. Question Count
Total design lock questions in this plan: 22.

## 17. Immediate Next Step
Run a short lock session to answer questions 1 through 7 first (versioning, naming, folder conventions), because those decisions unblock all implementation slices.

## 18. Lock Decisions
1. Lock 1 (versioning): Use schema/runtime export version 1.1 for this restructuring cycle, with no planned legacy-format read support after sample migration in this repo.
2. Lock 2 (project/runtime naming): Replace the clean naming path and shift file token from sbe to sbr for runtime package outputs. Project entry becomes <ProjectName>.sbr.runtime.json, and companion runtime artifacts use the same sbr + runtime naming direction.
3. Lock 3 (navigation granularity): Decompose navigation by area (one runtime navigation file per area) instead of a single project-wide aggregate navigation file.
4. Lock 4 (scope-kind folders): Use exact ScopeNodeKind enum token text as folder names with no normalization/transform step.
5. Lock 5 (scope-node file naming): Use id-first naming inside scope-kind folders: <ScopeNodeId>.runtime.json.
6. Lock 6 (guid filename format): Use Guid D format with uppercase hex in scope-node filenames (XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX.runtime.json).
7. Lock 7 (readability strategy): Keep scope-node filenames id-only. Human-readable name navigation is provided by export-root html index with links.
8. Lock 8 (reference payload shape): Parent/child references are id-only (child scope node id only). Scope-kind folders are disk storage concerns, while runtime load builds in-memory indexes for fast id-based resolution.
9. Lock 9 (ordering semantics): Treat relationship reference arrays as unordered by default (no order meaning) and only preserve order where the relationship is explicitly order-sensitive. Capture and test those exceptions explicitly.
10. Lock 10 (path hints): Do not store file paths in runtime data payloads. Resolution is id-driven via in-memory index, with disk lookup by id filename only when necessary.
11. Lock 11 (runtime-generated artifact placement): Deferred. Make specific placement decisions case-by-case using concrete runtime artifacts encountered during implementation.
12. Lock 12 (direct-command-echo sidecar): Keep this file in the runtime export package so runtime load has one package location for required run artifacts. Apply runtime/sbr naming cleanup as part of export naming realignment.
13. Lock 13 (load diagnostics format): Keep current simple csv diagnostics output for now; revisit JSON-primary diagnostics as a future enhancement.
14. Lock 14 (compatibility posture): Hard cutover to new runtime/sbr format (no old-format runtime load support), with an explicit migration mechanism to update existing in-repo sample and test projects.
15. Lock 15 (writer behavior): Exporter writes new runtime/sbr format only; no dual-write path.
16. Lock 16 (assets): No asset-layout changes in this effort. Keep existing asset export/dedupe structure as-is.
17. Lock 17 (index diagnostics affordance): Runtime index html includes a top status section and links to concerns csv files when present.
18. Lock 18 (index shape): Use a single-page runtime index. Entries use fully qualified scope paths (for example planet.country.area.room.object) and are sorted alphanumerically for quick lookup.
19. Lock 19 (index row display): Clickable link text is the fully qualified path; scope kind and scope id are shown as separate columns.
20. Lock 20 (canonical fully-qualified path): Use dot-separated paths rooted at global for all entries (including planets), with no special-case treatment for global. Use NameInGame fallback Name for each segment; sort alphanumerically by path with stable guid tie-break when labels collide.
21. Lock 21 (export strictness): Treat unresolved required references as blocking. Maintain existing Designer validation workflow as the primary quality gate and fail export when blocking validation issues remain.
22. Lock 22 (one-time migration strategy): Regenerate runtime exports from authoring project sources, then intentionally update sample/test baselines in-repo.
23. Lock 23 (procedure/action containment): Keep procedures and actions embedded in owner node payloads for this phase. They stay id-referenced behavior blocks, not standalone scope-kind node files.

Lock status summary:
- Questions 1-22 are now locked.
- Question 11 remains intentionally deferred for case-by-case artifact placement decisions during implementation.
- Implementation decision 23 is locked for this phase.

## 19. Validation Rule Expansion Plan (Reference Integrity)
Direction lock:
- Use the existing Designer validation pipeline as the primary mechanism.
- Add missing reference-integrity rules rather than introducing a separate export-specific technique.
- Keep current save flow behavior where validation issues can be reviewed and user can choose to continue save/export.

### 19.1 New Rules To Add
1. ACT-013 InvokeProcedure Procedure Reference Exists (Error)
- Scope: Action candidates.
- Checks: InvokeProcedure actions with ProcedureId must resolve to an existing procedure in project procedures.

2. ACT-014 Materialize Source Object Reference Exists (Error)
- Scope: Action candidates.
- Checks: MaterializeObjectCopy actions with MaterializeSourceObjectId must resolve to an existing project object id.

3. PROJ-012 Procedure Ownership Reference Exists (Error)
- Scope: Global candidate (whole-project check).
- Checks:
	- Project GlobalProcedureIds must resolve to existing procedure ids.
	- GameObject OwnedProcedureIds must resolve to existing procedure ids.

4. TRV-006 Room Placement Reference Integrity (Error)
- Scope: Area candidates.
- Checks: Area room placement entries must resolve to room ids that exist in that area and should not duplicate same room placement entry.

5. PROJ-013 Runtime Export Id Uniqueness (Error)
- Scope: Global candidate (whole-project check).
- Checks: IDs that will become runtime file identities must be unique across exported runtime scope nodes; emit collisions as blocking validation errors.

### 19.2 Registration and Execution
1. Register new rules in the main validation registry creation path used by save/validate workflows.
2. Keep rule categories aligned with existing conventions:
- Actions: ACT-*
- Traversal: TRV-*
- Project: PROJ-*
3. Ensure each rule emits actionable issue paths and fix hints consistent with existing rule style.

### 19.3 Test Coverage Additions
1. Add focused rule tests for each new rule with both failing and passing fixtures.
2. Add at least one integration-style validation run asserting combined behavior with existing reference rules.
3. Ensure no regression in current save validation prompt behavior.

### 19.4 Sequencing
1. Implement rules before runtime file-per-scope writer cutover so migration catches bad references early.
2. Apply sample/test project migration after rules are in place, using the rule set as the primary quality gate.

## 20. Execution Phases And Slices
Phase A - Validation Hardening (Designer-First)
- Slice A1: Add ACT-013 (InvokeProcedure procedure reference exists).
- Slice A2: Add ACT-014 (Materialize source object reference exists).
- Slice A3: Add PROJ-012 (procedure ownership reference exists).
- Slice A4: Add TRV-006 (room placement reference integrity).
- Slice A5: Add PROJ-013 (runtime-export id uniqueness across exportable scope nodes).
- Slice A6: Run validation across all sample projects and capture a consolidated report of newly emitted warnings/errors by rule id and sample project.
- Exit gate: Save/validation workflows surface new blocking issues correctly, focused rule tests pass, and sample-wide validation results are reviewed/triaged.

### 20.1 Phase A Status (2026-08-01)
- A1 complete: ACT-013 implemented and covered by focused action tests.
- A2 complete: ACT-014 implemented and covered by focused action tests.
- A3 complete: PROJ-012 implemented and covered by focused project-rule tests.
- A4 complete: TRV-006 implemented and covered by focused traversal-rule tests.
- A5 complete: PROJ-013 implemented and covered by focused project-rule tests.
- A6 complete: sample-wide sweep executed and consolidated results written to `plans/active/PHASE_A_SAMPLE_VALIDATION_REPORT.csv`.
- A6 triage note: resolved. Updated three sample authoring room files to use enum token `ChildrenAfterParent` for `childCommandForwardingMode` (instead of numeric `2`), then re-ran sweep. Current report shows all scanned sample authoring projects load and emit no ACT-013/ACT-014/PROJ-012/TRV-006/PROJ-013 findings.

Phase B - Runtime Naming And Root Contract Realignment
- Slice B1: Rename runtime package root naming from clean/sbe to runtime/sbr.
- Slice B2: Update root artifact names consistently (project root file, navigation-by-area files, sidecars retained in package).
- Slice B3: Keep asset structure unchanged.
- Exit gate: Runtime load resolves new runtime/sbr package names end-to-end.

Phase C - File-Per-Scope Writer Refactor
- Slice C1: Introduce per-ScopeNodeKind folders (exact enum token names).
- Slice C2: Emit scope-node files as id-only names using uppercase Guid D format and .runtime.json suffix.
- Slice C3: Replace deep inline child payloads with id-reference arrays (unordered by default; preserve explicit order-only exceptions).
- Slice C4: Decompose navigation into one file per area.
- Exit gate: Export output shape matches locked conventions and determinism expectations.

Phase D - Runtime Reader/Loader Refactor (Hard Cut)
- Slice D1: Remove old clean-format runtime load path assumptions.
- Slice D2: Build runtime in-memory id index at load startup for fast id resolution.
- Slice D3: Keep references id-only and avoid path hints in payloads.
- Exit gate: New-format-only runtime package loads successfully in simulator and shared runtime paths.

Phase E - Runtime Index HTML
- Slice E1: Emit single-page index at export root.
- Slice E2: Show fully qualified global-rooted path as primary link text.
- Slice E3: Add scope kind and scope id columns.
- Slice E4: Sort rows alphanumerically by fully qualified path with stable guid tie-break.
- Slice E5: Add top status section with link to concerns csv when present.
- Exit gate: Index provides reliable name-based navigation to id-named files.

Phase F - Migration And Baseline Refresh
- Slice F1: Regenerate runtime exports from authoring sources for in-repo samples.
- Slice F2: Refresh tests/snapshots/baselines intentionally for new runtime/sbr structure.
- Slice F3: Triage any validation failures found during migration and fix source authoring data.
- Exit gate: Samples, tests, and runtime host flows all green on new format only.

Phase G - Stabilization And Closeout
- Slice G1: Resolve deferred artifact-placement decisions (Lock 11) with concrete cases.
- Slice G2: Update docs (README, enhancement guidance references) to runtime/sbr terminology and new layout examples.
- Slice G3: Run full solution build and focused runtime regression gates.
- Exit gate: Plan marked complete and archived with validation evidence.