# Runtime Navigation Area-First File Structure Plan

## Goal
Refactor runtime navigation export and loading from a single project-level navigation file to an area-first file structure where each area conveys its own traversal links and room placements.

## Why
- Reduce growth pressure on one large navigation file as project area count increases.
- Co-locate area traversal metadata with area-scoped runtime data.
- Improve incremental diffs and inspection by isolating navigation changes to affected areas.

## Related Follow-Up Linkage
- This plan is directly related to the TraversalConnectionDto designer/runtime mapping gap.
- Current flow stores rich traversal connection semantics in designer area data but runtime navigation currently carries only simplified links and placements.
- During this refactor, explicitly decide whether area-first runtime payloads should preserve richer traversal-leg semantics (for example open-state binding, passable shared-variable linkage, and traversal leg actions) instead of continuing to flatten to minimal link data.

## Locked Regression Requirement
- Regression statement: locked-door traversal gating is currently bypassed because traversal leg passability/linkage semantics are not fully present in runtime navigation payloads.
- This plan must restore end-to-end traversal gating by persisting and loading traversal-leg passability semantics.
- Required runtime navigation payload coverage per leg:
  - `gameProperties` entry for `isPassable` (including default value)
  - `isPassable` shared-variable linkage via game property `sharedVariableId` when bound
  - `openableObjectId` when bound
  - `openStatePolicy`
  - `openStateBindingMode`
- Required behavior contract:
  - Traversal blocked state must be driven by traversal `isPassable` effective value.
  - Door opening must work through shared-variable linkage from door `isOpen` to traversal `isPassable`.
  - Mandatory regression proof: attempting movement through a locked door must fail until door-open behavior flips effective traversal passability to true.

## Contract Canonicalization Gap (Designer vs Shared Core)
- Observed issue: `AreaDto.authored.cs` currently carries structural persistence members (`TraversalConnections`, `RoomPlacements`, and legacy `Links`) that are not fully expressed through the shared schema-generated area contract shape.
- Current schema state:
  - `Area.core.schema.json` includes only `adjacencyMode`, `startingRoomId`, and `roomIds`.
  - `RoomPlacementDto` and `ProjectRoomLinkDto` types are schema-backed, but their containment on `Area` is currently authored partial composition.
  - `TraversalConnectionDto` / traversal leg state shape is not yet represented in shared schema contracts.
- Lock implication for this plan:
  - Area navigation structure needed by both designer and runtime must be promoted to schema-owned contracts, with area containment defined in shared core/wrapper schemas instead of hand-authored partial-only structure.
- Required contract promotion outcomes:
  - Add schema-owned traversal connection and traversal leg state contracts.
  - Add schema-owned area navigation containment members so designer and runtime wrappers compose from the same canonical area shape.
  - Keep any temporary legacy fields explicitly compatibility-scoped and deprecation-tagged.

## Shared Core Convergence Rule (Locked)
- Temporary bridge allowance: runtime partial DTO enrichment used during regression recovery is transitional only.
- Target architecture: core traversal/navigation structures are schema-owned shared contracts consumed by both designer and runtime wrappers.
- Allowed divergence: only host-specific fields required by one host, with explicit rationale and narrow scope.
- Not allowed as end-state: duplicate parallel authored structures carrying the same core traversal semantics across designer/runtime.

## Current State Summary
- Runtime export writes one navigation file: `<Project>.sbr.runtime.navigation.json`.
- File contains all areas with `planet`, `country`, `area`, `adjacencyMode`, `links`, and `roomPlacements`.
- Runtime bootstrap reads this aggregate file and builds an in-memory area-keyed navigation index.

## Target State Summary
- Runtime export writes per-area navigation sidecars in an area-scoped folder.
- Each area sidecar contains only that area's navigation payload.
- Runtime bootstrap composes the same in-memory area-keyed index from per-area files.
- Optional compatibility period: support both legacy aggregate and area-first layouts.

## Scope
- In scope:
  - Runtime contract/schema design for area-first navigation payload.
  - Export pipeline updates to emit area-first navigation files.
  - Runtime loader/bootstrap updates to read area-first files.
  - Traversal-leg passability/linkage field propagation into runtime navigation payloads and runtime leg descriptors.
  - Backward compatibility and migration behavior.
  - Snapshot and regression test updates.
- Out of scope:
    - New traversal gameplay feature expansion unrelated to restoring authored passability/linkage behavior.
  - Changes to command grammar/parsing behavior.
  - Non-navigation runtime file layout redesign.

## Design Lock-Off Questions (Commit Gate)
Lock-off question count: 18

1. Should area navigation be embedded into `RuntimeAreaDto` sidecars, or emitted as dedicated area-navigation sidecars?
2. If embedded, which exact fields move into runtime area files (`adjacencyMode`, `links`, `roomPlacements`, and any future traversal-leg semantics)?
3. If dedicated sidecars are chosen instead, what exact per-area file name pattern is canonical?
4. What is the canonical area-key identity for composition (`planet|country|area` by name vs stable id-based key)?
5. What deterministic ordering rules are required for areas, links, and room placements to preserve snapshot stability?
6. During transition, does runtime read precedence use area-first as authoritative with aggregate fallback only when area-first data is absent?
7. When both layouts exist for the same area, do we hard-fail, soft-warn with deterministic winner, or merge?
8. If merge is allowed, which source wins per field on conflict (`adjacencyMode`, links, placements)?
9. What diagnostics contract is required for mixed-source, duplicate-area, missing-room, and invalid-direction issues?
10. Should writer behavior be dual-write (area-first plus aggregate) for a bounded window, and what is the exact removal milestone?
11. Is aggregate navigation officially deprecated in this plan, and in which slice is deletion of aggregate write/read code approved?
12. Do we keep current flattened `RuntimeRoomLinkDto` semantics only, or extend runtime area payload now to carry richer traversal leg semantics (open-state policy, shared variable linkage, leg actions)?
13. If richer traversal semantics are deferred, what explicit no-regression guarantee is required for current traversal behavior?
14. What schema/version decision is required for this change (stay `1.0` additive vs explicit version bump if breaking)?
15. Which exact sample projects require one-time runtime export migration in the same change set?
16. What is the acceptance rule for stale sample artifacts (must regenerate all `GameRuntimeJson` outputs vs selective updates)?
17. Which test gates are mandatory before merge (full app tests, focused runtime guardrails, playback smoke, snapshot baselines)?
18. What is the final exit signal for lock-off completion (all questions answered in plan + compatibility window decision + migration checklist complete)?

## Lock-Off Decisions To Date
1. Question 1: Area navigation is embedded into `RuntimeAreaDto` sidecars as the canonical source.
2. Question 2: Runtime area payload must include traversal-leg passability/linkage semantics required to restore locked-door traversal gating (`isPassable` game property semantics including property-level `sharedVariableId`, `openableObjectId`, `openStatePolicy`, `openStateBindingMode`).
3. Question 3: Closed as not applicable because Question 1 selected embedded runtime-area payloads; no dedicated area-navigation sidecar naming contract is introduced.
4. Question 4: Canonical area composition identity is stable `ScopeNodeId` (GUID). Name-based composite keys are non-authoritative and allowed only for diagnostics/transition logging.
5. Question 5: Deterministic ordering is ID-first. Area-level traversal and placement payloads use stable sort rules to preserve snapshot/diff stability.
6. Question 6: Read path is area-first authoritative. Aggregate navigation is compatibility fallback only for areas that cannot supply area-first navigation due to legacy format, with explicit diagnostics for mixed-source resolution.
7. Question 7: When both layouts exist for the same area, runtime uses soft-warning with deterministic winner. Area-first wins, aggregate for that area is ignored, and diagnostic context is emitted.
8. Question 8: Closed as not applicable. Field-level merge is disallowed; conflict resolution is whole-area winner selection per Question 7.
9. Question 9: Runtime navigation loading emits structured NAV diagnostics with stable codes and remediation hints. Invalid traversal references and malformed gating semantics are errors; mixed-source and legacy-fallback events are warnings with explicit source resolution context.
10. Question 10: Export writer dual-writes area-first and aggregate navigation for one transition release window only. Aggregate write is removed after migration and validation gates demonstrate area-first-only readiness.
11. Question 11: Aggregate navigation is deprecated effective immediately. Aggregate write path is removed in Cleanup Slice after transition gates pass. Aggregate read fallback is removed after migration evidence shows no fallback use and compatibility fixtures are retired.
12. Question 12: After prerequisite designer contract canonicalization, runtime area payload must include traversal leg gating/linkage semantics (`isPassable` game property semantics including property-level `sharedVariableId`, `openableObjectId`, `openStatePolicy`, `openStateBindingMode`) in addition to directional leg identity.
13. Question 13: Closed as not applicable because richer traversal semantics are not deferred. Regression safety remains mandatory via targeted traversal gating tests and playback validation.
14. Question 14: Keep `schemaVersion` at `1.0` throughout this initiative. Changes are implemented with additive contract evolution and compatibility behavior inside v1.0 rather than version bumps.
15. Question 15: One-time sample runtime migration scope includes all sample projects currently carrying aggregate navigation runtime artifacts (11 total under `Samples/*/GameRuntimeJson/*.sbr.runtime.navigation.json`).
16. Question 16: Sample migration acceptance is exporter-regenerated from authored `.sbe` source files only (runtime JSON is never migration input). If authored traversal linkage exists but regenerated runtime output still misses it, migration is blocked until exporter/contract mapping is fixed; locked-door traversal regression gate must pass before migration acceptance.
17. Question 17: Mandatory pre-merge gates are locked to build, focused runtime guardrails, playback smoke, full app tests, runtime-export snapshot validation, and a dedicated replay regression that attempts movement through a locked door before unlock/open and re-attempts after unlock/open.
18. Question 18: Lock-off completion requires all 18 decisions captured, prerequisite area-contract canonicalization merged, traversal linkage present in runtime area payload and consumed by runtime, new locked-door replay regression passing, all validation gates green, full sample regeneration complete, and aggregate deprecation/removal milestones explicitly resolved.

## Proposed Execution Slices
0. Prerequisite Slice: Designer Area Contract Canonicalization
- Promote area navigation containment and traversal connection/leg structures from authored-only partials to schema-owned contracts.
- Ensure designer area DTO composition is primarily schema-driven for navigation structure needed by runtime alignment.
- Preserve legacy compatibility members only when explicitly required for migration.

1. Contract Proposal Slice
- Define schema options and choose final area-first contract shape.
- Decide compatibility strategy (dual-read, dual-write, or staged read-first).

2. Loader-First Compatibility Slice
- Add runtime reader support for area-first layout while preserving legacy aggregate read.
- Build same `NavigationByArea` runtime index shape used today.
- Add diagnostics for mixed/duplicate/conflicting navigation sources.

3. Export Writer Slice
- Emit per-area navigation files from export.
- Keep aggregate file optionally behind a temporary compatibility flag if needed.
- Ensure deterministic ordering and stable serialization.

4. Contract Lock + Regeneration Slice
- Regenerate DTO contracts from approved schema changes.
- Lock and verify generated contract parity.

5. Cleanup Slice
- Remove temporary compatibility switches once migration window closes.
- Remove obsolete aggregate-only code paths when approved.

## Compatibility Strategy (Draft)
- Read path (transition):
  - Prefer area-first files when present.
  - Fall back to aggregate file when area-first files are absent.
- Write path (transition):
  - Emit area-first files.
  - Optional: also emit aggregate navigation file for one release window.
- Diagnostics:
  - Emit warnings when duplicate area keys are discovered across sources.
  - Emit warnings when area entries reference unknown room IDs.

## Validation Plan
1. Build and baseline tests:
- `dotnet build .\StoryboardDesigner.slnx`
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`

2. Runtime-focused guardrail tests:
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`

3. Playback smoke gate per change:
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`

4. Snapshot coverage:
- Refresh runtime export snapshots intentionally and verify both legacy and area-first fixture expectations during transition.

5. Locked-door traversal gate (mandatory):
- Add or update a focused automated regression test that verifies:
  - Move attempt through locked door fails while effective traversal `isPassable=false`.
  - After door open (shared-variable linkage update), the same move succeeds.
- Add a replay-based regression case that explicitly attempts locked-door movement before unlock/open and validates success only after unlock/open.
- Replay capture sequencing note: record this replay only after traversal-linkage export/load fixes are merged; current known-broken behavior (walking through locked doors) should be treated as pre-fix baseline and not accepted as replay truth.
- Run this gate in every slice that touches area contracts, traversal mapping, export, or runtime bootstrap.

## Risks
- Contract churn affecting existing export consumers.
- Mixed-layout ambiguity during migration window.
- Snapshot noise from path/layout changes.
- Hidden assumptions in bootstrap mapping keyed by `planet|country|area`.

## Exit Criteria
- Runtime export supports area-first navigation file structure.
- Runtime loader/bootstrap produces equivalent traversal behavior from area-first data.
- Compatibility behavior is explicit, tested, and documented.
- Build and targeted/full regression suites pass.
- Legacy aggregate path has a defined deprecation/removal decision.