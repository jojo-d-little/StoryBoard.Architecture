# Designer DTO File Structure Consistency Plan

Status: Active (rebased to current implementation state; near closeout)
Owner: StoryboardDesigner.App authoring persistence + runtime export alignment
Last updated: 2026-08-06 (rebased to current code/test state)

## 0. Execution Snapshot

Current implementation status relative to this plan:

1. Runtime canonical Room scope-kind path is now operational and validated.
2. Legacy runtime clean room sidecar writer path is removed from forward writes.
3. Runtime ids-first containment work is substantially complete for Room/Area/Country/Planet/Project runtime export shapes.
4. Designer sidecar contracts remain mixed-shape for object containment (ids plus embedded object payloads in some files).
5. Import Globals discovery now resolves canonical project linkage from globals sidecars; diagnostics hardening is in progress.
6. Designer sidecar physical layout convergence is implemented: canonical scope-kind folder names and scope-kind object folder routing are active with compatibility read fallback.
7. One-time sample migration automation has been implemented and executed with backup + report output.

Latest validation evidence:

1. dotnet build .\StoryboardDesigner.slnx passes.
2. dotnet test .\StoryboardDesigner.slnx passes (1129 passed, 0 failed).

## 1. Purpose

Define a focused, low-risk plan to increase consistency between designer DTO persistence shapes and runtime scope-kind persistence shapes.

Primary outcomes:

1. Move scope-level object containment toward ids-first file contracts (instead of embedded full object arrays).
2. Remove active reliance on legacy clean room sidecar format for runtime consumption.
3. Align designer disk layout directionally with runtime physical structure (folder by scope kind, one object per file, navigable index artifact).
4. Preserve and maintain Import Globals from another project across the persistence layout transition.

## 2. Problem Statement (Rebased)

Canonical runtime room persistence is now established, but designer-side persistence and compatibility hardening are still mid-transition:

1. Runtime room canonical read/write behavior has been cut over and validated.
2. Designer sidecar contracts still include mixed containment shapes in some scopes (ids plus embedded payloads).
3. Import Globals compatibility behavior is partially upgraded, but diagnostics and parity coverage are still being hardened.
4. Sample migration has executed, but final signoff/cleanup gates are not yet fully closed.

Result: canonical direction is established, but plan closeout still depends on finishing designer ids-first normalization, import hardening, and migration signoff evidence.

## 3. Scope and Non-Goals

In scope:

1. Runtime room persistence/read-path standardization to Room scope-kind files as canonical.
2. Designer DTO contract direction toward ids-first containment references.
3. File-structure alignment planning for designer persisted artifacts.
4. Import Globals compatibility strategy and maintenance worklist.
5. One-time migration strategy for existing sample projects.

Out of scope for this slice:

1. Full schema-lock rollout for all designer DTOs.
2. Large UX refactors unrelated to persistence shape.
3. Runtime behavior/command engine changes unrelated to file format.

## 4. Principles and Decision Defaults

1. One canonical disk representation per relationship (avoid dual-shape write contracts).
2. Backward-compatible reads may be temporary, but forward writes should be canonical early.
3. Keep runtime/designer boundaries explicit (authoring model flexibility can remain in-memory).
4. Small, observable slices with regression gates before each follow-on phase.
5. Migration safety before cleanup speed.

## 5. Target Direction (High Level)

1. Runtime room canonical disk shape: Room scope-kind files under GameRuntimeJson/Room with gameObjectIds.
2. Runtime clean room sidecar folder becomes deprecated and then removed.
3. Designer persisted scope files progressively move from embedded BaseObjects/GameObjects lists to BaseObjectIds/GameObjectIds with sidecar object nodes.
4. Designer disk layout converges toward folder-per-scope-kind and one-node-per-file conventions where practical.
5. Runtime-style index artifact pattern is introduced/extended for designer persistence navigation.

## 6. Phase Plan

### Phase A - Baseline Lock and Decision Packet

Status: Complete

1. Confirm and document canonical Room file decision: GameRuntimeJson/Room is primary.
2. Capture current read/write behavior evidence for rooms and scope-kind nodes.
3. Freeze migration acceptance criteria and rollback strategy.

Deliverables:

1. Decision note in plan updates.
2. Baseline artifact inventory (what files are currently emitted and consumed).

#### Phase A Design Lock-Off Questions

Use this checklist to lock decisions before implementation slices begin.

1. Canonical room source of truth
- Question: Should GameRuntimeJson/Room be the only canonical runtime room source?
- Recommendation: Yes. Treat clean room sidecars as temporary compatibility-read only, then remove.
- Status: Agreed (mandatory prerequisite before runtime ids-only contract normalization)

2. Reader precedence during transition
- Question: During migration, should reader resolve Room scope files first and only use clean room sidecars when Room files are missing?
- Recommendation: Yes. Prefer Room first to enforce canonical behavior while preserving transitional safety.
- Status: Agreed (mandatory prerequisite before runtime ids-only contract normalization)

3. Writer cutover timing
- Question: Should writer stop emitting clean room sidecars in the first implementation slice?
- Recommendation: Yes, unless an explicit release-gated compatibility switch is required.
- Status: Agreed (mandatory prerequisite before runtime ids-only contract normalization)

4. ids-only policy scope
- Question: Should ids-only be required for runtime disk contracts first, with designer contract changes phased after runtime room cutover?
- Recommendation: Yes. Runtime-first keeps blast radius bounded.
- Status: Agreed

5. Dual-shape conflict rule
- Question: If payload includes both ids and embedded objects, which source wins?
- Recommendation: ids win; embedded objects are ignored and diagnostics are emitted.
- Status: Agreed

6. Designer physical layout target
- Question: Do we lock now that designer persistence will converge to folder-per-scope-kind + one-node-per-file + index artifact?
- Recommendation: Yes, lock target now and phase implementation later.
- Status: Agreed (with caveat: requires robust one-time migration of existing sample projects)

7. Import Globals compatibility window
- Question: Must Import Globals support both legacy and canonical donor layouts during transition?
- Recommendation: Yes, for at least one migration window with diagnostics for mixed/corrupt donor layouts.
- Status: Agreed (transition support allowed, but end state must be one mechanism against canonical/new format only)

8. Project/sample migration strategy
- Question: Should migration be one-time and deterministic with explicit integrity reporting?
- Recommendation: Yes. Use id-preserving deterministic migration and produce a migration report.
- Status: Agreed (with operational safeguard: maintain backup copy of samples before migration runs)

9. Versioning and layout signaling
- Question: How should canonical layout be signaled on disk?
- Recommendation: Use explicit version/manifest marker; do not rely only on folder-shape detection.
- Status: Agreed (use schemaVersion as the canonical file-format/version signal)

10. Legacy-path removal gate
- Question: What evidence is required before deleting legacy readers/writers?
- Recommendation: Require green focused/full tests, sample migrations complete, and no fallback-hit diagnostics in validation runs.
- Status: Agreed

### Phase B - Runtime Room Canonical Cutover

Status: Complete (functional), pending minor naming/comment cleanup

Gate: This phase is a prerequisite and must complete before any runtime ids-only contract normalization changes.

1. Change runtime read path to prefer Room scope-kind files first (or exclusively).
2. Stop writing clean room sidecar files once read compatibility is settled.
3. Keep temporary backward read fallback only if needed for transition.
4. Add explicit tests proving canonical source and fallback behavior.

Deliverables:

1. Reader precedence update.
2. Writer cleanup for legacy room sidecar emission.
3. Regression tests and snapshot updates.

Implemented notes:

1. Runtime writer emits Room scope-kind files and deletes legacy clean room sidecar folder when present.
2. Runtime reader resolves Room scope-kind folder as canonical runtime room input.
3. Full solution build and test gates are green after cutover-related refactors.

### Phase C - Designer DTO ids-first Contract Introduction (After Layout Stabilization)

Status: In Progress (partial), sequenced after Phase D completion

1. Add ids-based fields for scoped containment where missing.
2. Switch writer intent to canonical ids references while preserving tolerant read for one transition window.
3. Add normalization rules to resolve ids into in-memory object graphs on load.
4. Add diagnostics for ambiguous dual-shape payloads (both ids and embedded objects).

Deliverables:

1. DTO contract updates and mapper updates.
2. Compatibility read rules and diagnostics.
3. Focused tests for ids-only persistence and roundtrip hydration.

Implemented notes:

1. Runtime export path writes ids-first scope relationships (PlanetIds/CountryIds/AreaIds/RoomIds and object id references).
2. Runtime contract alignment and fallback hydration behavior were strengthened in shared bootstrap mapping.

Remaining within Phase C:

1. Designer authoring sidecar DTOs still include embedded object arrays in multiple scopes.
2. Dual-shape conflict diagnostics policy (ids win, embedded ignored with diagnostics) is not yet fully enforced across designer-side persistence reads.

### Phase D - Designer Physical Layout Convergence

Status: Complete

1. Introduce or standardize folder-per-scope-kind persisted layout for designer artifacts.
2. Persist one scope/object node per file in canonical folders.
3. Generate an index artifact to navigate persisted graph paths and file locations.
4. Remove superseded legacy placement paths after migration succeeds.

Deliverables:

1. New canonical folder layout.
2. Index artifact generation and validation.
3. Old layout deprecation plan and cleanup criteria.

Notes:

1. Runtime export index artifact generation exists; designer-native folder convergence in this phase is implemented and active.
2. This phase now precedes remaining Phase C DTO normalization work to reduce compounded migration risk.
3. Pilot implementation landed for canonical writer folder names on designer sidecars using existing DTOs.
4. Legacy fallback for core project scope loading has now been removed; loader behavior is canonical-only for Room/Planet/Country/Area/Procedure project reads.

Pilot implementation detail (completed):

1. Canonical writer folders now use scope-kind names for designer sidecars: Room, Planet, Country, Area, Procedure.
2. Reader compatibility supports both canonical and legacy folder conventions with canonical-first precedence.
3. Sidecar loading deduplicates by id across canonical/legacy dual presence to avoid duplicate graph materialization.
4. Regression test coverage now includes explicit legacy-folder fallback load verification.

### Phase E - Import Globals Maintenance and Compatibility Hardening

Status: In Progress

1. Update Import Globals discovery to support new canonical layout.
2. Preserve support for old incoming donor projects during transition period.
3. Add parity tests proving imported objects/actions/properties survive layout changes.
4. Add conflict-resolution diagnostics for duplicate ids or mixed legacy/canonical inputs.

Deliverables:

1. Import Globals reader compatibility matrix.
2. Regression tests for old and new donor project shapes.

### Phase F - One-time Migration of Existing Projects and Samples

Status: Complete for Samples (workspace migration executed); signoff/cleanup pending

1. Implement one-time migration utility/path for existing projects.
2. Run migration on Sample projects and validate output parity.
3. Produce migration report with changed file counts and integrity checks.
4. Remove transition toggles only after migration confidence gates pass.

Deliverables:

1. Migration mechanism.
2. Updated sample projects in canonical layout.
3. Migration verification report and signoff checklist.

## 7. Risk Assessment

Overall risk: Moderate (mostly due to file-structure migration and import compatibility).

Key risks:

1. Reader/writer mismatch during transition causing partial loads.
2. Import Globals regressions from changed discovery paths.
3. Hidden assumptions in tests/tools expecting legacy room sidecars.
4. Mixed-shape payload ambiguity (ids + embedded objects) leading to data drift.
5. Sample migration one-time failures (or accidental data loss).

Mitigations:

1. Enforce canonical write early; tolerate legacy read only temporarily.
2. Add explicit dual-shape conflict diagnostics.
3. Land changes by phase with focused tests and snapshot checks.
4. Keep migration deterministic and id-preserving.
5. Validate with both focused runtime gates and full app test pass.

### Import Globals Critical Gate (Phase D/E Coupled Risk)

This is the highest-risk area for physical layout changes.

Required protections before final closeout of layout-transition work:

1. Import Globals resolver supports both legacy and canonical donor folder shapes during transition.
2. Resolver emits deterministic diagnostics when canonical path lookup fails and legacy fallback is used.
3. Mixed-layout donor detection is explicit (diagnostic + deterministic precedence rules).
4. Duplicate-id conflicts produce deterministic resolution diagnostics and do not silently override content.

Required evidence to close Phase E and declare transition hardening complete:

1. Legacy donor import regression tests pass.
2. Canonical donor import regression tests pass.
3. Mixed donor layout tests pass with expected diagnostics.
4. Sample-project migration rehearsal shows no import parity regressions.
5. Full solution regression remains green after layout pilot merges.

## 8. Test Strategy

Automated coverage additions/updates:

1. Runtime reader precedence tests: Room scope-kind first.
2. Runtime writer tests: no clean room sidecar emission once cutover is complete.
3. DTO roundtrip tests for ids-only scoped containment.
4. Import Globals compatibility tests for legacy and canonical donor layouts.
5. Migration tests validating count and identity parity before/after conversion.

Validation commands:

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"
4. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"

## 9. Migration and Rollout Controls

1. Use phased feature flags or explicit transition markers where needed.
2. Keep one version window where legacy read remains available.
3. Emit migration diagnostics and summary artifacts on conversion runs.
4. Require sample-project migration pass before declaring layout lock.
5. Defer hard deletion of legacy readers until post-migration verification closes.

## 10. Exit Criteria

1. Runtime room canonical source is Room scope-kind files.
2. Clean room sidecar format is no longer written or required.
3. Designer scoped containment contracts are ids-first on disk.
4. Designer file layout is canonical, navigable, and documented.
5. Import Globals remains viable for both legacy and canonical donor projects during transition.
6. Existing sample projects are migrated once with verified parity.

## 10.1 What Remains (Rebased)

Priority-ordered remaining implementation and signoff items:

1. Complete Phase E Import Globals compatibility matrix and parity tests for legacy, canonical, and mixed donor layouts.
2. Complete Phase E diagnostics hardening for canonical lookup misses, legacy fallback use, mixed-layout donors, and duplicate-id conflict handling.
3. Complete remaining Phase C designer-side ids-first DTO normalization for scopes still persisting embedded object arrays.
4. Fully enforce Phase C dual-shape conflict policy across designer reads (ids win; embedded payload ignored with diagnostics).
5. Close Phase F signoff/cleanup by publishing final migration verification evidence and confirming cleanup gates.
6. Remove any temporary compatibility fallback paths only after fallback-hit diagnostics remain clean through validation runs.
7. Apply minor naming/comment cleanup noted in Phase B and then move this plan to archived once exit criteria are met.

Known cleanup item:

1. Continue monitoring nullable-analysis output in CI/full local builds; no active CS8601 warning is currently emitted in JsonExportService.

## 11. Immediate Next Slice Recommendation

1. Finish the Phase E Import Globals hardening gate first (matrix coverage + deterministic diagnostics).
2. In the same slice (or immediately after), complete remaining Phase C designer ids-first DTO normalization and dual-shape enforcement.
3. Run focused and full regressions after C/E changes, then finalize Phase F signoff evidence package.
4. Once the remaining gates are complete, update status to Complete and archive this plan.

## 12. Notes

1. This plan intentionally stages behavior and file-structure changes separately to minimize compounding risk.
2. Schema-locking designer DTOs can proceed after ids-first canonical layout is stable.