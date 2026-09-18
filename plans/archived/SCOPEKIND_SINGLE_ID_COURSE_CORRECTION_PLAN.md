# ScopeKind + Single ID Course Correction Plan

Last updated: 2026-07-08
Status: Completed (Archived)

## Intent

Course-correct runtime scope identity toward one canonical identity per node, with semantics driven by ScopeKind and compatibility fields retained during migration.

## Archive Note

Execution is complete through Slice 6 and post-Slice-6 cleanup queue item 11.

1. All defined implementation slices (0-6) are complete and validated.
2. Post-Slice-6 cleanup queue entries in this plan are marked completed.
3. Plan is archived for historical traceability.

## Locked Decisions

1. Canonical runtime identity property name is ScopeNodeId.
2. ScopeNodeId is additive first; no removal of RoomId/ObjectId in early slices.
3. Runtime behavior selection is always by ScopeKind, never by whichever legacy identity field is non-null.
4. Room transitional invariant is RoomId == ObjectId == ScopeNodeId.
5. Clean-export schema remains 1.0 through Phases 0-3.
6. Rapid prototype mode is active: backward compatibility for historical disk/json payloads is not required beyond keeping in-repo sample projects and tests green.
7. Primary implementation focus is Shared/runtime identity semantics; Designer model updates are secondary and only taken when they clearly improve runtime-aligned workflows.
8. Designer is allowed to preserve more distinct node-type handling than runtime when that distinction is host-UX-specific and does not leak into Shared/runtime contracts.
9. ScopeNodeId is the globally unique identifier for a runtime scope node; ScopeKind classifies node type and is not part of uniqueness.
10. Designer alignment toward generalized node identity is preferred over time, but each Designer-side step must include explicit risk discussion and can be deferred when risk outweighs near-term value.

## Design Lockoff Questions (Discuss And Commit)

Each question must be answered and marked Locked before Slice 1 starts.

| ID | Question | Proposed Options | Decision | Owner | Locked Date |
| --- | --- | --- | --- | --- | --- |
| DLQ-01 | What is the canonical runtime identity naming split between contracts and implementation: ScopeId everywhere, or ScopeNodeId on interface with ScopeId in mapping internals? | A: ScopeId everywhere. B: ScopeNodeId on IRuntimeScopeNode and ScopeId in mapper descriptors. C: ScopeNodeId everywhere. | C: ScopeNodeId everywhere. | User | 2026-07-07 |
| DLQ-02 | In the transition period, do we keep both RoomId and ObjectId on runtime nodes as compatibility aliases that mirror canonical identity values by ScopeKind? | A: Keep both through Slice 5. B: Keep one alias only. C: Remove both earlier in Slice 4. | A: Keep both through Slice 5. | User | 2026-07-07 |
| DLQ-03 | For room nodes, should the transitional invariant remain strict equality RoomId == ObjectId == canonical id in all mapper paths? | A: Yes, strict in all paths. B: Strict only in bootstrap path. C: Best-effort only. | A: Yes, strict in all paths. | User | 2026-07-07 |
| DLQ-04 | For non-room non-object nodes (planet/country/area/global/player/templates), must canonical identity always be non-null? | A: Required non-null for all runtime nodes. B: Required except global/templates. C: Allow null for non-addressable nodes. | A: Required non-null for all runtime nodes. | User | 2026-07-07 |
| DLQ-05 | What is the final runtime contract end state for legacy members: remove RoomId and rename ObjectId to ScopeNodeId in the same breaking slice or staged over two slices? | A: Same slice. B: Two-step staged rename/removal. C: Keep deprecated members one extra release branch. | B: Two-step staged rename/removal. | User | 2026-07-07 |
| DLQ-06 | Should RuntimeParentObjectId remain named as-is, or be generalized to parent canonical identity naming as part of final gates? | A: Keep RuntimeParentObjectId. B: Rename to RuntimeParentScopeNodeId. C: Split parent type + parent id fields. | B: Rename to RuntimeParentScopeNodeId. | User | 2026-07-07 |
| DLQ-07 | For runtime logic branching, is ScopeKind the only approved discriminator, with explicit ban on identity-field-presence checks? | A: Yes, hard ban. B: Ban for new code only. C: Allow temporary exceptions behind TODO tags. | A: Yes, hard ban. | User | 2026-07-07 |
| DLQ-08 | What is the expected behavior when ScopeKind and identity values are inconsistent at runtime construction time? | A: Throw hard exception. B: Log and normalize. C: Skip malformed node and continue. | A: Throw hard exception with node context diagnostics (name, parent, and available identity fields). | User | 2026-07-07 |
| DLQ-09 | Do we require parity tests asserting equivalent identity semantics from both mapper ingress paths (clean bootstrap and designer project model)? | A: Required for every slice touching identity mapping. B: Required only before Slice 6. C: Best-effort. | A: Required for every slice touching identity mapping. | User | 2026-07-07 |
| DLQ-10 | Which compatibility target is mandatory in prototype mode? | A: Only in-repo sample projects and tests. B: In-repo plus latest saved project format. C: Broad external payload support. | A: Only in-repo sample projects and tests. | User | 2026-07-07 |
| DLQ-11 | When Designer keeps distinct node semantics for UX, what is the required documentation level in this plan? | A: One-line rationale in slice notes. B: Rationale plus adapter contract note. C: Separate designer divergence log section. | B: Rationale plus adapter contract note, with per-step risk discussion for Designer alignment/defer decisions. | User | 2026-07-07 |
| DLQ-12 | Should we enforce no-new-usage guardrails for RoomId/ObjectId in Shared generic runtime code before final removal? | A: Yes, analyzers/tests from Slice 3 onward. B: Tests only in Slice 5+. C: No guardrails until final slice. | A: Yes, analyzers/tests from Slice 3 onward. | User | 2026-07-07 |
| DLQ-13 | What is the minimum required green gate for merging each identity slice in prototype mode? | A: Build + focused runtime + smoke replay. B: Full test suite every slice. C: Build + focused runtime only. | A (amended): Build + focused runtime + smoke replay as baseline; full suite is mandatory progression gate before moving from each of Slices 0-4 to the next slice. | User | 2026-07-07 |
| DLQ-14 | Do we update public DTO naming in the same final gate as interface renames, or keep DTO naming stable with adapters? | A: Rename DTOs in same gate. B: Keep DTO names stable, adapt internally. C: Hybrid by DTO type. | C: Hybrid by DTO type during transition, with mandatory final cleanup to single canonical identity naming and no lingering legacy adapters. | User | 2026-07-07 |
| DLQ-15 | What is the explicit cutover trigger to start Slice 6 final approval gates? | A: All DLQs locked + Slice 0-5 complete + green full suite. B: Product-owner signoff only. C: Timebox-based cutover with residual risk acceptance. | A: All DLQs locked + Slice 0-5 complete + green full suite. | User | 2026-07-07 |

Lockoff completion rule:

1. Each DLQ must have a non-TBD decision, owner, and locked date.
2. Decision log entries must reference DLQ IDs when a lockoff answer changes plan behavior.
3. Slice 1 cannot start until DLQ-01 through DLQ-05 are locked.
4. Slice 6 cannot start until all DLQs are locked.

## Final Approval Gates (Long-Stretch End State)

These are approval gates for the final stretch and are not authorized for early slices.

1. Remove CleanRoomExportV1Dto.Id.
2. Remove CleanScopeNodeBase.RoomId.
3. Remove IRuntimeScopeNode.RoomId.
4. Rename IRuntimeScopeNode.ObjectId to ScopeNodeId (generalized identity naming).
5. Runtime logic must branch on node semantics using ScopeKind only; identity field presence must not determine node type behavior.
6. End state must remove transitional identity aliases/adapters so only one canonical identifier path remains.

Final-gate readiness preconditions:

1. All generic runtime flows are ScopeNodeId-first and verified by tests.
2. Compatibility strategy is approved for in-repo samples/tests; external historical payload compatibility is optional in prototype mode.
3. Legacy read paths are isolated to explicit compatibility shims only.

## Scope

In scope:

1. Shared runtime contract augmentation with ScopeNodeId.
2. Mapper and runtime consumer migration to ScopeNodeId-first generic identity handling.
3. Guardrail tests preventing reintroduction of identity-branching drift.
4. Explicit callouts for any Designer-side parallel updates that are intentionally coupled to runtime identity changes.

Out of scope for initial activation:

1. Clean-export breaking contract/version bump.
2. Designer UX or editing workflow rewrites unrelated to runtime scope identity.
3. Any Storyboard.Simulator dependency on StoryboardDesigner.App (must remain independent).
4. Broad compatibility guarantees for arbitrary external legacy project files.

## Runtime-First Boundary Rule

1. Treat Shared/runtime as the source of truth for canonical scope identity behavior.
2. Evaluate Designer model changes independently; do not mirror runtime identity changes into Designer by default.
3. When Designer is changed in parallel, document classification: Required boundary adapter update.
4. When Designer is changed in parallel, document classification: Optional alignment for maintainability.
5. When Designer is changed in parallel, document classification: Intentional divergence for authoring UX.
6. Any intentional divergence must be called out in slice notes with a short rationale.

## Code Touchpoint Inventory (Prepared)

Primary contract/state surfaces:

1. Storyboard.Shared/GameStateData/RuntimeScopeNodeDescriptor.cs
2. Storyboard.Shared/GameStateData/GameStateScopeNode.cs
3. Storyboard.Shared/GameStateData/RuntimeScopeReference.cs
4. Storyboard.Shared/RuntimeContracts/IRuntimeScopeNode.cs

Primary mapper ingress points:

1. Storyboard.Shared/GameServices/Bootstrap/CleanRuntimeBootstrapSnapshotMapper.cs
2. StoryboardDesigner.App/GameServices/ProjectModelRuntimeSnapshotMapper.cs

Primary runtime consumer hotspots:

1. Storyboard.Shared/GameStateData/GameStateSession.cs
2. Storyboard.Shared/GameServices/Commands/RuntimeCommandProcessorService.cs
3. Storyboard.Shared/GameServices/Commands/GameCommandPreprocessorService.cs
4. Runtime action executables under Storyboard.Shared/GameServices/Actions/GameActions/RuntimeActionExecutable/

Existing test anchors to extend (do not duplicate coverage blindly):

1. StoryboardDesigner.App.Tests/RuntimeActionPayloadAccessorsTests.cs
2. StoryboardDesigner.App.Tests/CompositeBuildActionTests.cs
3. StoryboardDesigner.App.Tests/GameCommandProcessorFixtureTests.cs
4. StoryboardDesigner.App.Tests/GameSimulatorPlaybackRegressionTests.cs
5. StoryboardDesigner.App.Tests/ArchitectureSeparationGuardrailsTests.cs

## Implementation Slices (PR-Sized)

Each slice must be mergeable on its own with tests green.

## Slice Progression Gate (Mandatory)

1. Before moving from Slice 0 to Slice 1, run full test suite and require green result.
2. Before moving from Slice 1 to Slice 2, run full test suite and require green result.
3. Before moving from Slice 2 to Slice 3, run full test suite and require green result.
4. Before moving from Slice 3 to Slice 4, run full test suite and require green result.
5. Before moving from Slice 4 to Slice 5, run full test suite and require green result.
6. Full suite command for progression gates: dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj

## Slice Size Heuristic (Prototype Mode)

1. A slice should usually target one primary objective and one subsystem boundary at a time.
2. Prefer slices that touch approximately 3 to 8 production files plus targeted tests.
3. If a slice changes shared contracts, mapper semantics, or identity invariants, treat it as elevated risk and run the full suite before merge.
4. If a slice is local refactor with no contract or mapper behavior change, default to build + focused runtime + smoke replay.
5. Split slices when reviewability drops or when the expected verification exceeds practical iteration time.

### Slice 0: Guardrails + Baseline

Goal: Create failing tests for identity ambiguity before production edits.

Tasks:

1. Add/extend tests asserting ScopeKind identity invariants, including room triple-equality invariant.
2. Add tests for generic identity consumers to reject ambiguous fallback precedence.
3. Capture a short decision note in this plan under Decision Log.

Exit criteria:

1. New tests fail against old behavior where ambiguity exists.
2. Baseline gates pass after tests are aligned with intended behavior.

Concrete Slice 0 task board:

1. Add contract guardrail tests for canonical identity shape in StoryboardDesigner.App.Tests/RuntimeScopeNodeExtensionsTests.cs.
2. Add runtime initialization failure tests for missing ScopeKind/ScopeNodeId diagnostics in StoryboardDesigner.App.Tests/QuantifiableRuntimeMaterializationTests.cs.
3. Add mapper parity identity assertions across both ingress paths in StoryboardDesigner.App.Tests/QuantifiableRuntimeMaterializationTests.cs.
4. Add room transitional invariant assertions (RoomId == ObjectId == ScopeNodeId during transition) in StoryboardDesigner.App.Tests/RuntimeActionPayloadAccessorsTests.cs.
5. Add architecture guardrails preventing new generic-runtime branching on identity-field presence in StoryboardDesigner.App.Tests/ArchitectureSeparationGuardrailsTests.cs.
6. Add a focused parity fixture proving equivalent identity semantics between CleanRuntimeBootstrapSnapshotMapper and ProjectModelRuntimeSnapshotMapper in StoryboardDesigner.App.Tests/SharedManagerHostFixtureTests.cs.

Slice 0 test case expectations:

1. Identity contract tests assert every runtime node has non-null ScopeKind and non-empty canonical identity value.
2. Identity uniqueness tests assert ScopeNodeId uniqueness is global within a runtime graph and independent from ScopeKind.
3. Failure-path tests assert hard exceptions include node name and parent context when identity/kind is invalid.
4. Mapper parity tests compare identity fields for equivalent node kinds and fail on semantic drift.
5. Branching guardrail tests fail if generic runtime logic uses RoomId/ObjectId presence to infer node type.

Slice 0 merge validation sequence:

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "ArchitectureSeparationGuardrailsTests|RuntimeActionPayloadAccessorsTests|QuantifiableRuntimeMaterializationTests|SharedManagerHostFixtureTests|RuntimeScopeNodeExtensionsTests"
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"
4. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"

Slice 0 execution log (2026-07-07):

1. Implemented guardrail tests in RuntimeScopeNodeExtensionsTests, ArchitectureSeparationGuardrailsTests, QuantifiableRuntimeMaterializationTests, RuntimeActionPayloadAccessorsTests, and SharedManagerHostFixtureTests.
2. Focused Slice 0 suite passed: 76 passed, 0 failed.
3. Focused runtime gate passed: 31 passed, 0 failed.
4. Replay smoke gate passed: 5 passed, 0 failed.
5. Full suite progression gate passed: 505 passed, 0 failed.

### Slice 1: Additive Contract Introduction

Goal: Introduce ScopeNodeId into shared contracts without behavior change.

Tasks:

1. Add ScopeNodeId to RuntimeScopeNodeDescriptor, GameStateScopeNode, RuntimeScopeReference, IRuntimeScopeNode.
2. Ensure constructors/factory paths are additive and binary-safe where possible.
3. Keep RoomId/ObjectId semantics untouched and mapped exactly as before.

Exit criteria:

1. All runtime nodes materialize ScopeNodeId.
2. No snapshot/playback regressions.

Slice 1 execution log (2026-07-07):

1. Added additive ScopeNodeId contract/property on IRuntimeScopeNode with compatibility fallback semantics.
2. Added ScopeNodeId properties to RuntimeScopeNodeDescriptor and RuntimeScopeReference (non-serialized).
3. Added ScopeNodeId property to GameStateScopeNode and aligned parent-link helper to canonical fallback.
4. Added ScopeNodeId to CleanScopeNodeBase and aligned parent fallback to ScopeNodeId.
5. Added guardrail tests for ScopeNodeId contract presence and fallback behavior.
6. Focused runtime gate passed.
7. Replay smoke gate passed.
8. Full suite progression gate passed: 506 passed, 0 failed.

### Slice 2: Mapper Population + Invariant Enforcement

Goal: Populate ScopeNodeId consistently from both ingress paths.

Tasks:

1. Populate ScopeNodeId in CleanRuntimeBootstrapSnapshotMapper for every ScopeKind.
2. Populate ScopeNodeId in ProjectModelRuntimeSnapshotMapper for every ScopeKind.
3. Add invariant assertions for room nodes and object parent-link scenarios.

Exit criteria:

1. Both mappers produce identical identity semantics for equivalent fixtures.
2. Focused runtime suite passes.

Slice 2 execution log (2026-07-07):

1. Added canonical scope identity carrier `CanonicalScopeNodeId` to runtime descriptors/references and mapped `ScopeNodeId` from canonical fallback.
2. Added deterministic scope identity helper in `RuntimeScopeIdentity` for non-room/non-object scope kinds.
3. Populated canonical scope identity in both ingress mappers (`CleanRuntimeBootstrapSnapshotMapper` and `ProjectModelRuntimeSnapshotMapper`) for planet/country/area/room/object nodes.
4. Updated runtime session materialization to carry canonical scope identity into `GameStateScopeNode` creation paths, including stable global/player scope identities.
5. Extended mapper parity tests to assert non-null `ScopeNodeId` and parity for planet/country/area/room/object nodes across both ingress paths.
6. Focused slice gate passed: 77 passed, 0 failed.
7. Focused runtime gate passed: 31 passed, 0 failed.
8. Replay smoke gate passed: 5 passed, 0 failed.
9. Full suite progression gate passed: 506 passed, 0 failed.

### Slice 3: Consumer Migration to ScopeNodeId-First Generic Logic

Goal: Move generic runtime codepaths to ScopeNodeId-first lookup/traversal.

Tasks:

1. Update generic lookup, traversal, and scope reference flows to prefer ScopeNodeId.
2. Keep domain-specific APIs that are explicitly room/object-centric on RoomId/ObjectId.
3. Remove or quarantine ambiguous fallback chains in generic paths.

Exit criteria:

1. Generic code no longer depends on RoomId/ObjectId when ScopeKind + ScopeNodeId is sufficient.
2. Full suite passes.

Slice 3 execution log (2026-07-07):

1. Updated runtime scope resolution in `GameStateSession` to resolve generic scope proxies by `ScopeNodeId` first (including room/area/country/planet target scope resolution), while preserving existing in-session object resolution semantics for runtime state nodes.
2. Updated generic traversal action-target mapping in `RuntimeCommandProcessorService` to match room traversal source nodes via canonical `ScopeNodeId`.
3. Added regression coverage in `GameStateSessionPlayerScopeTests` proving `TryReparentObject` can resolve both source object and target room from `ScopeNodeId` on proxy scope inputs even when names are stale.
4. Focused slice gate passed: 88 passed, 0 failed.
5. Focused runtime gate passed: 31 passed, 0 failed.
6. Replay smoke gate passed: 5 passed, 0 failed.
7. Full suite progression gate passed: 507 passed, 0 failed.

### Slice 4: Compatibility Tightening

Goal: Constrain legacy identity fields to compatibility surfaces only.

Tasks:

1. Document where RoomId/ObjectId remain required and why.
2. Reduce internal reliance on legacy identity reads in shared runtime internals.
3. Add guardrail test(s) preventing new generic logic from bypassing ScopeNodeId.

Exit criteria:

1. Internal generic identity flow is canonicalized.
2. Remaining legacy reads are explicit and justified.

Slice 4 execution log (2026-07-07):

1. Added canonical runtime scope index in `GameStateSession` (`_scopeNodesById`) and migrated `ScopeNodeId` resolution to indexed lookup with duplicate-id detection.
2. Kept object `ScopeNodeId`-first resolution constrained to non-session proxy scopes so compatibility behavior for in-session runtime nodes remains unchanged.
3. Tightened architecture guardrail baseline in `ArchitectureSeparationGuardrailsTests` by reducing `RuntimeCommandProcessorService` identity-presence branching budget from 1 to 0.
4. Added/retained regression coverage proving stale-name proxy scope operations resolve via `ScopeNodeId` (`GameStateSessionPlayerScopeTests.TryReparentObject_ResolvesObjectAndTargetRoom_ByScopeNodeId`).
5. Documented compatibility-only legacy identity surfaces for this stage:
6. `MoveToRoom(Guid roomId)` and traversal leg DTO fields (`SourceRoomId`, `DestinationRoomId`) remain `RoomId`-based pending final contract gate.
7. Composite/provenance and authored-object recipe semantics remain `ObjectId`-based pending final rename/removal gate.
8. Focused slice gate passed: 52 passed, 0 failed.
9. Focused runtime gate passed: 31 passed, 0 failed.
10. Replay smoke gate passed: 5 passed, 0 failed.
11. Full suite progression gate passed: 507 passed, 0 failed.

### Slice 5: Deprecation Decision Package (No Immediate Break)

Goal: Produce decision-ready package for contract versioning.

Tasks:

1. Assess whether clean-export contract bump is required for legacy field removal.
2. Keep migration support only to the extent required for repository samples/tests to pass.
3. Record prototype-mode compatibility stance and deferred hardening items.

Exit criteria:

1. Version/migration decision approved and documented.
2. In-repo sample and test matrix identified and green.

Slice 5 execution log (2026-07-07):

1. Decision package approved: final-gate identity removals/renames require a versioned contract cutover (clean export v2/runtime contract v2) and are not permitted on v1 surfaces.
2. Prototype compatibility stance confirmed: compatibility guarantee is limited to in-repo samples and automated tests; broad historical external payload compatibility remains out of scope.
3. Added explicit automated sample runtime-load matrix coverage in `SampleRuntimeLoadMatrixTests`:
4. Authored + runtime-loadable samples (green): Birmingham, ObjectPlayLevel1, ObjectPlayLevel2, TraversalExamples.
5. Clean-export samples (green): Birmingham.sbe.clean.json, ObjectPlayLevel1.sbe.clean.json, SingleRoomAntics.sbe.clean.json, ObjectPlayLevel2.sbe.clean.json, TraversalExamples.sbe.clean.json.
6. Classified authored sample without generated clean export: MapDemo1.sbe.json (expected load diagnostic, covered by test).
7. Deferred hardening items recorded for Slice 6 gate:
8. Remove v1 transitional aliases (`CleanRoomExportV1Dto.Id`, `CleanScopeNodeBase.RoomId`, `IRuntimeScopeNode.RoomId`).
9. Rename generalized runtime identity member from `ObjectId` to `ScopeNodeId` on contract-facing runtime surfaces.
10. Align parent identity naming (`RuntimeParentObjectId` -> `RuntimeParentScopeNodeId`) and migrate dependent composite/provenance flows.
11. Focused sample matrix gate passed: 14 passed, 0 failed.
12. Focused runtime gate passed: 31 passed, 0 failed.
13. Replay smoke gate passed: 5 passed, 0 failed.
14. Full suite gate passed: 517 passed, 0 failed.

### Slice 6: Final Approval Gates Execution (Breaking/Versioned)

Goal: Land approved end-state identity surface after migration is complete.

Tasks:

1. Remove CleanRoomExportV1Dto.Id and migrate room identity readers to canonical generalized identity.
2. Remove CleanScopeNodeBase.RoomId.
3. Remove IRuntimeScopeNode.RoomId.
4. Rename IRuntimeScopeNode.ObjectId to ScopeNodeId and migrate all runtime consumers.
5. Enforce ScopeKind-based branching rules in runtime logic and guardrails.

Exit criteria:

1. No runtime code infers node semantics from identity property presence.
2. All contract, playback, and focused runtime regressions pass under the approved version strategy.
3. Migration/compatibility notes are documented in-repo.

Slice 6 execution log (2026-07-07):

1. Final contract surface cutover completed in Shared runtime contracts: `IRuntimeScopeNode` now exposes canonical `ScopeNodeId` and `RuntimeParentScopeNodeId`, with `RoomId` removed and generalized identity naming no longer exposed as `ObjectId` on the interface.
2. Clean-room export DTO migration executed for final gate: removed `CleanRoomExportV1Dto.Id`, introduced `RoomScopeNodeId` serialized as `id`, and migrated room identity readers/writers to canonical scope identity flow.
3. Migrated remaining test callsites to `RoomScopeNodeId` in `QuantifiableRuntimeMaterializationTests` and aligned clean export snapshot expectations.
4. Refreshed targeted clean export snapshot baseline intentionally via `UPDATE_CLEAN_EXPORT_SNAPSHOTS=1` for `JsonExportServiceCleanExportSnapshotTests.ExportCleanProjectV1_MatchesSingleRoomSnapshotBaseline`.
5. Focused migration regression gate passed: 33 passed, 0 failed.
6. Focused runtime gate passed: 31 passed, 0 failed.
7. Replay smoke gate passed: 5 passed, 0 failed.
8. Full suite gate passed: 517 passed, 0 failed.
9. Completed parent identity naming cleanup by renaming remaining internal runtime-state and descriptor members from `RuntimeParentObjectId` to `RuntimeParentScopeNodeId` across Shared mappers/session/state and associated tests.
10. Post-rename validation rerun passed: focused runtime gate (31/31), replay gate (5/5), and full suite (517/517).

### Shared/Runtime Identity Member Cleanup Queue (Post Slice 6)

Goal: remove remaining duplicate/legacy identity member naming from Shared runtime surfaces in small validated increments.

1. Completed: Clean room DTO canonicalization.
	- Removed `CleanRoomExportV1Dto.RoomScopeNodeId`.
	- Bound serialized room `id` directly to canonical `ScopeNodeId`.
	- Enabled settable canonical identity in `CleanScopeNodeBase.ScopeNodeId` to support direct DTO binding.
	- Validation: focused migration tests, focused runtime gate, replay gate, and full suite all green.
2. Completed: runtime descriptor canonicalization in `RuntimeScopeNodeDescriptor`.
	- Removed `RoomId`/`ObjectId` as descriptor member properties.
	- Switched shared runtime descriptor consumption to canonical `ScopeNodeId` derivation by `ScopeKind` where room/object-typed state nodes are materialized.
	- Migrated shared mapper descriptor creation to canonical-only argument flow.
	- Left a temporary compatibility constructor (`RoomId`/`ObjectId` parameters) marked obsolete to keep remaining callsites compiling while cleanup proceeds item-by-item.
3. Completed: runtime reference canonicalization in `RuntimeScopeReference`.
	- Removed `RoomId`/`ObjectId` member properties.
	- Retained only canonical `ScopeNodeId` on the reference shape.
	- Validation: build, runtime-focused gate, replay gate, and full suite all green.
4. Completed: runtime node canonicalization in `GameStateScopeNode`.
	- Converted runtime state handling to `ScopeNodeId`-first usage across Shared runtime consumers (`GameStateSession`, navigation action flow).
	- Removed duplicate identity storage by treating `RoomId`/`ObjectId` as computed compatibility aliases over canonical `ScopeNodeId`.
	- Validation: build, runtime-focused gate, replay gate, and full suite all green.
5. Completed: room-centric contract review in `CleanRoomPlacementDto.RoomId` and related area/traversal contracts.
	- Decision: keep room-domain identifiers where the payload semantics are explicitly room-edge/topology concerns (`CleanRoomPlacementDto.RoomId`, `CleanAreaDto.StartingRoomId`, `CleanAreaDto.RoomIds`, `RuntimeTraversalLegDescriptor.SourceRoomId/DestinationRoomId`, `RuntimeGameWorldSnapshot.StartingRoomId`).
	- Rationale: these members represent room-to-room graph linkage, not duplicate identity surfaces on a single scope-node contract type.
6. Completed: runtime navigation API review.
	- Decision: keep room-domain navigation signatures (`MoveToRoom(Guid roomId)`, `IRuntimeScopeMutationGateway.TryMoveToRoom(Guid roomId)`) as intentional room-addressing APIs.
	- Rationale: command/runtime traversal semantics are room-targeted operations even under canonical scope-node identity elsewhere.
7. Completed: transitional descriptor constructor callsite cleanup.
	- Migrated remaining legacy `RuntimeScopeNodeDescriptor` callsites in test fixtures from paired `RoomId`/`ObjectId` arguments to canonical `CanonicalScopeNodeId` argument usage.
	- Result: removed `CS0618` obsolete-constructor warnings tied to transitional descriptor identity parameters.
	- Validation: `dotnet build .\StoryboardDesigner.slnx` now reports only 3 pre-existing `xUnit2031` warnings (down from 74 total warnings); runtime-focused gate, replay gate, and full suite remain green.
8. Completed: runtime state-node room alias removal in `GameStateScopeNode`.
	- Removed compatibility alias `GameStateScopeNode.RoomId` from Shared runtime state.
	- Migrated simulator/designer runtime tree projection and room-map callsites to canonical `ScopeNodeId` with explicit `GameScopeKind.Room` guards.
	- Updated runtime-focused tests that asserted `CurrentRoom.RoomId` to assert `CurrentRoom.ScopeNodeId` instead.
	- Validation: build, runtime-focused gate, and replay gate all green.
9. Completed: runtime state-node object alias removal in `GameStateScopeNode`.
	- Removed compatibility alias `GameStateScopeNode.ObjectId` from Shared runtime state.
	- Migrated remaining runtime materialization assertion from object alias usage to canonical `ScopeNodeId`.
	- Validation: build, runtime-focused gate, and replay gate all green.
10. Completed: clean DTO base object-identity fallback removal in `CleanScopeNodeBase`.
	- Removed transitional fallback from `ScopeNodeId` getter (`_scopeNodeId ?? ObjectId`) and deleted `ObjectId` virtual member from the base contract shape.
	- Result: `CleanScopeNodeBase` now exposes canonical `ScopeNodeId` only for generalized identity.
	- Validation: build, runtime-focused gate, and replay gate all green.
11. Completed: runtime descriptor canonical-name compaction in `RuntimeScopeNodeDescriptor`.
	- Removed the obsolete transitional descriptor constructor and collapsed descriptor identity naming from `CanonicalScopeNodeId` to `ScopeNodeId`.
	- Updated all mapper and test callsites to use named argument `ScopeNodeId` for descriptor construction.
	- Validation: build, runtime-focused gate, replay gate, and full suite all green.

## Definition of Done (Per Slice)

1. Build passes: dotnet build .\StoryboardDesigner.slnx
2. Focused runtime gate passes:
	- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"
3. Smoke replay gate passes:
	- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
4. Full test suite passes for merge-bound slices and is mandatory before advancing from Slices 0-4:
	- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj

Additional gate for Slice 6:

1. Add/enable architecture guardrail asserting runtime contracts no longer expose IRuntimeScopeNode.RoomId.
2. Add/enable guardrail asserting runtime contracts use ScopeNodeId naming instead of ObjectId for generalized identity.

## Risk Register

1. Contract drift risk: Legacy field behavior may change unintentionally while adding ScopeNodeId.
2. Behavioral regression risk: Navigation/traversal and composite actions have historical identity coupling.
3. Fixture brittleness risk: Tests may encode prior fallback precedence.
4. Prototype tradeoff risk: external legacy payloads may fail to load after contract simplification.

## Mitigations

1. Additive-first sequencing and no schema break through Phase 3.
2. Mapper parity checks for clean-export and designer-runtime mapping paths.
3. Focused runtime gate on every slice, full suite on merge-bound slices.
4. Keep repository samples and regression fixtures as the operative compatibility safety net during prototype mode.

## Rollback Strategy

1. If a slice regresses runtime behavior, revert only that slice; do not partially revert prior guardrail tests.
2. Keep ScopeNodeId additive during early slices so rollback does not require contract removal.
3. Preserve compatibility reads until Slice 5 decision is approved.

## Decision Log

1. 2026-07-07 (DLQ-01): Canonical runtime identity name locked to ScopeNodeId everywhere.
2. 2026-07-07 (DLQ-02): Keep both RoomId and ObjectId compatibility aliases through Slice 5.
3. 2026-07-07 (DLQ-03): Enforce room transitional invariant strictly in all mapper paths.
4. 2026-07-07 (DLQ-04): Every runtime scope node must have both ScopeKind and non-null ScopeNodeId; this pair is the fundamental node identity.
5. 2026-07-07 (DLQ-05): Final legacy-member transition will be staged over two slices rather than collapsed into one breaking slice.
6. 2026-07-07 (DLQ-06): Parent identity naming will align to canonical terminology via RuntimeParentScopeNodeId.
7. 2026-07-07 (DLQ-07): ScopeKind is the only approved runtime branching discriminator; identity-presence branching is prohibited.
8. 2026-07-07 (DLQ-08): Runtime initialization must throw hard exception when ScopeKind or ScopeNodeId is missing/invalid; exception details should include node name, parent context, and identity fields.
9. 2026-07-07 (DLQ-09): Mapper parity tests are required for every slice that touches identity mapping semantics.
10. 2026-07-07 (DLQ-10): Prototype-mode mandatory compatibility target is in-repo sample projects and tests only.
11. 2026-07-07 (DLQ-11): Designer divergence/alignment decisions require rationale plus adapter contract note, with explicit per-step risk discussion.
12. 2026-07-07 (DLQ-12): Enforce no-new RoomId/ObjectId generic-runtime usage guardrails from Slice 3 onward.
13. 2026-07-07 (DLQ-13 amended): Default merge gate is build + focused runtime + smoke replay; full suite is mandatory before advancing from each of Slices 0-4.
14. 2026-07-07 (DLQ-14): DTO transition may be hybrid/staged, but final state must remove transitional identity adapters and retain only canonical identity naming.
15. 2026-07-07 (DLQ-15): Slice 6 cutover requires all DLQs locked, Slice 0-5 complete, and green full-suite validation.
16. 2026-07-07: ScopeNodeId is globally unique on its own; ScopeKind indicates node type and is not a uniqueness discriminator.
17. 2026-07-07: Room transitional invariant locked: RoomId == ObjectId == ScopeNodeId.
18. 2026-07-07: Contract version remains 1.0 during additive/migration slices.
19. 2026-07-07: Slice 5 decision package approved; final identity removals/renames require a versioned v2 cutover.
20. 2026-07-07: In-repo sample compatibility matrix automated; MapDemo1 is intentionally classified as authored-only until clean export artifacts are generated.

## Archive Checklist

1. Status set to Completed (Archived).
2. File moved from plans/active to plans/archived.
3. Historical execution and validation logs retained in place.
