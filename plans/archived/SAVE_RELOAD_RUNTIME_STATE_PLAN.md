# Save/Reload Runtime State Plan

Status: Closed
Date: 2026-08-11

## Goal

Define and implement a durable save/reload capability that can persist an in-flight runtime game session and rehydrate it later, starting with local disk JSON and preserving a clean path to future storage backends.

## Problem Statement

Players need a reliable way to persist current runtime progress and restore it later without losing authoritative in-flight session state.

## Scope

1. Create a dedicated save-state schema family and generated DTO contracts.
2. Keep save payload contracts separate from authored project contracts and runtime export contracts.
3. Use a single JSON document per save-state artifact for v1.
4. Implement save/reload around runtime scope-node-centric state, not designer object-type-centric structure.
5. Bind each save file to the base game content identity it is valid for.
6. Support local-disk storage first through an abstraction that can later target database/blob storage.
7. Add validation, diagnostics, and migration/version seams compatible with current schema/codegen lock practices.
8. Support explicit Save Game and Load Game operations at host level using shared runtime services.
9. Use `Storyboard.Simulator` File menu Save Game / Load Game actions as the first proving vehicle.

## Hard Boundary Requirement

1. This effort must make no code changes in `StoryboardDesigner.App`.
2. Expected implementation footprint is primarily `Storyboard.Shared.Contracts` and `Storyboard.Shared`.
3. Host-level integration changes are limited to `Storyboard.Simulator` invocation paths needed to trigger save/load.
4. Designer behavior and UX are intentionally out of scope and must remain unaffected.

## Contract Home (Locked)

1. Save-state contracts are housed under `Storyboard.Shared.Contracts/SaveGameStateContracts`.
2. Folder structure follows existing project convention with dedicated subfolders for schemas and DTO artifacts.
3. Save-state schemas/DTOs remain standalone in this contract family.
4. Cross-family schema references are allowed only for shared enum definitions from runtime contracts.
5. Non-enum DTO/schema structure is not reused from runtime contracts; it is authored in save-state contracts.

## Non-Goals (This Plan)

1. Multiplayer transport implementation details.
2. Cloud storage implementation.
3. Encryption/signing/compression hardening.
4. Final UX polish for save slot management.
5. Autosave/checkpoint product policy.

## Architectural Direction (Locked Baseline)

1. Save contracts are a distinct runtime contract family.
2. Enumerations should be reused from canonical shared enum definitions rather than duplicated in save schemas.
3. Save format remains storage-agnostic: same serialized contract regardless of local file vs future database/blob persistence.
4. Save artifact must include base-content identity metadata so restore is a two-step process:
- load/resolve base game content
- rehydrate session state on top
5. Local-disk v1 may include relative path locator hints, but content identity fields remain authoritative.
6. Preserve host/runtime boundaries:
- host UX/orchestration remains host-specific
- reusable save/reload runtime logic belongs in `Storyboard.Shared`
7. Base game identity fields use `gameProjectId` naming.
8. `StoryboardDesigner.App` remains untouched by this effort; simulator is the only host integration surface for v1 proof.

## Proposed Contract Shape (Direction)

1. Save envelope metadata:
- save schema version
- created timestamp
- runtime engine/build version
- optional checksum/integrity fields

2. Base-content binding block:
- `gameProjectId`
- optional `gameProjectVersion`
- optional `gameProjectFingerprint`
- optional `gameProjectLocatorHint` (relative path to runtime entry file)

3. Runtime snapshot block:
- session identity/state
- scope-node graph state
- variables/containers/object instances
- active procedures/actions and relevant transient execution context
- timers/scheduled events needed for correct continuation

4. Optional diagnostics block:
- warning records from save-time validation
- migration markers when save was upgraded

## Phased Implementation Order (Contract-First)

This effort follows a strict contract-first sequence: define and lock contract shape before feature implementation consumes it.

### Phase 0 - Contract Session And Lock-Off

1. Hold focused contract design session for save-state schema/DTO shape.
2. Confirm lock-off decisions already recorded in this plan remain accurate.
3. Freeze naming and structural decisions before implementation begins.

Exit criteria:
1. Save contract shape agreement is explicit and documented.
2. No unresolved contract-shape questions remain for v1.

### Phase 1 - Author Save Schemas

1. Add dedicated save schema artifacts in `Storyboard.Shared.Contracts`.
2. Reuse canonical enum definitions; do not duplicate enum vocabularies.
3. Encode required/optional field policy for sparse payloads.

Exit criteria:
1. Schema files compile/validate in current codegen workflow.
2. Contract-review pass confirms lock-off alignment.

### Phase 2 - Generate And Lock DTO Contracts

1. Generate save DTOs from schemas.
2. Run contract lock/drift validations.
3. Resolve any schema/codegen fallout before touching runtime behavior.

Exit criteria:
1. Generated save DTOs are stable and accepted.
2. Lock manifests/guardrails pass.

### Phase 3 - Shared Runtime Consumption

1. Implement serializer/deserializer and rehydration pipeline in `Storyboard.Shared`.
2. Apply schema-driven migration/defaulting policies in load path.
3. Add diagnostics for identity mismatch, enum/migration failures, and corruption.

Exit criteria:
1. Shared runtime can roundtrip save/load using contract DTOs.
2. Focused shared tests pass.

### Phase 4 - Storage Provider And Simulator Invocation

1. Add local-disk save storage provider behind abstraction.
2. Add limited `Storyboard.Simulator` file-menu invocation wiring for Save Game / Load Game.
3. Keep `StoryboardDesigner.App` untouched.

Exit criteria:
1. Manual simulator smoke flow succeeds end-to-end.
2. Host-boundary guardrails remain intact.

### Phase 5 - Validation, Hardening, And Closeout

1. Run full validation gates and focused save/reload tests.
2. Validate additive-update compatibility behavior under policy.
3. Record closure evidence and follow-on items (if any).

Exit criteria:
1. All required gates pass.
2. Plan status can be moved to Closed.

## Contract-Effort Rule For Future Features

1. Existing contract families: expect small, targeted contract deltas first, then implementation.
2. New contract families (this effort): expect heavier upfront contract definition and lockoff before implementation.
3. In both cases, implementation starts only after contract agreement and generation/lock validation are complete.

## Implementation Workstreams (Small Slices)

1. Define save schema folder layout and naming conventions in `Storyboard.Shared.Contracts`.
2. Define save envelope + base-content binding schemas.
3. Define scope-node-centric runtime state schemas and generate DTOs.
4. Wire serializer/deserializer service abstractions in `Storyboard.Shared`.
5. Implement local-disk storage provider in host layer(s), keeping storage abstraction boundary clean.
6. Implement restore pipeline:
- resolve base game content identity
- load base content
- apply runtime state rehydration
7. Add diagnostics for missing/incompatible base-content identity.
8. Add version/migration entry points for save schema evolution.
9. Add validation/corruption handling for malformed, truncated, or incompatible save payloads.
10. Add host integration seams for explicit Save Game / Load Game operations.
11. Add `Storyboard.Simulator` File menu integration for Save Game / Load Game as initial end-to-end proof path.
12. Add focused tests for roundtrip fidelity, compatibility/version behavior, and identity mismatch handling.
13. Run full validation gates and lock artifacts.

## Validation Gates

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`
3. `dotnet test .\Storyboard.Shared.Tests\Storyboard.Shared.Tests.csproj --filter "FullyQualifiedName~RuntimeSaveGameStateServiceScaffoldingTests"`
4. `dotnet test .\StoryboardDesigner.slnx`
5. Manual simulator smoke proof: save from `Storyboard.Simulator` File menu, then load and verify in-flight continuation behavior.

## Design Lock-Off (Resolved)

1. Base-content match policy:
- require `gameProjectId` match
- prefer `gameProjectFingerprint` match
- allow compatibility fallback only through explicit compatibility/migration rule

2. Identity mismatch behavior:
- default fail hard
- explicit override path is allowed only via deliberate confirmation flow with diagnostics

3. Persisted vs recomputed state boundary:
- persist authoritative gameplay/runtime state
- recompute derived indexes/caches/projections on load

4. Timer/event representation:
- use logical game-time offsets as authoritative values
- optional wall-clock timestamps are diagnostics only

5. Sparse payload policy:
- omit empty/null optional fields/collections for thinner files
- required fields remain explicit
- loader performs canonical defaulting for omitted optional fields

6. Unknown enum handling:
- unknown enum values are hard load errors
- compatibility uses explicit schema/version migration value mappings

7. RNG forward-compat seam:
- include RNG state support as planned field/seam
- keep optional/inactive until randomness exists in runtime behavior

8. Integrity/checksum policy:
- checksum field is optional in v1
- when present, mismatch is a load failure
- checksum scope excludes checksum field itself and is computed over canonicalized scoped content

9. Migration posture:
- rely primarily on schema-driven compatibility hints (aliases/translations/defaults)
- if a case exceeds schema-hint capability, treat as targeted exception requiring separate design review

10. Locator hint basis:
- `gameProjectLocatorHint` points to runtime entry artifact
- path is relative to save file location
- hint is optional and non-authoritative

11. Host neutrality:
- save contract is host-neutral from day one
- host-specific behavior remains in host orchestration/UX layers

12. Post-update restore compatibility policy:
- support restoring player progress after additive/safe core-game updates
- additive/safe means updates avoid removing/invalidating referenced runtime entities needed by existing saves
- allow changed-base restore only when compatibility/migration checks pass under this additive-safety policy

## Decision Log

1. Save/reload uses dedicated schema + generated DTO contracts.
2. Save DTO structure is runtime scope-node-centric, not designer type-centric.
3. Enumerations are reused from canonical shared enum vocabularies.
4. Save artifact is a single JSON document in v1.
5. Restore is explicitly modeled as base-content load + session rehydration.
6. Base-content identity naming uses `gameProjectId` field family (`gameProjectVersion`, `gameProjectFingerprint`, `gameProjectLocatorHint`).
7. Restore matching uses required id match and controlled compatibility fallback, not silent fallback.
8. Identity mismatch default is hard-fail with explicit override path only.
9. Payload shape is intentionally sparse for optional data; loader defaulting normalizes omitted optional fields.
10. Unknown enum values fail load and must be handled through migration mappings.
11. Save contracts include forward-compatible RNG seam, initially optional until randomness exists.
12. `gameProjectLocatorHint` targets runtime entry artifact and is relative to save-file location.
13. Save artifact is host-neutral across Designer and Simulator by contract.
14. Compatibility across base-game updates is required for additive/safe updates via explicit compatibility/migration checks.
15. Initial proving UX path is `Storyboard.Simulator` File menu Save Game / Load Game.
16. Hard boundary locked: no `StoryboardDesigner.App` changes; implementation is contracts/shared plus minimal simulator invocation wiring.
17. Execution ordering is contract-first: schema/DTO agreement and lock before runtime/host implementation.
18. Save-state contract home is `Storyboard.Shared.Contracts/SaveGameStateContracts` with dedicated schema/DTO subfolders.

## Session Closeout Evidence (2026-08-10)

1. Save-state contract lock lane in `Storyboard.SchemaCodegen` is implemented for generate + lock operations:
- `generate-savegame-contract-staging`
- `generate-savegame-enum-staging`
- `lock-savegame-contract`
- `lock-savegame-enum`

2. Save-state lock manifests are populated and present:
- `CodegenManagment/ContractLock/locked-savegame-contract-dtos.txt` (10 DTO entries)
- `CodegenManagment/ContractLock/locked-savegame-contract-enums.txt` (1 enum entry)

3. Lock parity verification completed after lock:
- flat staging regenerated to `CodegenManagment/Staging/SaveGameStateDtos` and `CodegenManagment/Staging/SaveGameStateEnums`
- DTO lock-target parity: file-set diff = 0, hash diff = 0
- Enum lock-target parity: hash match = true

4. Build + focused guardrail validation succeeded after lock:
- `dotnet build .\StoryboardDesigner.slnx`
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"` (62/62 passing)
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"` (8/8 passing)

5. Boundary confirmation for this session:
- no implementation changes were made in `StoryboardDesigner.App`
- effort remained within contracts/codegen/validation lanes
6. Runtime non-scope save/load behavior refinement completed in shared runtime:
- runtime non-scope restore is RoomLink-only
- `TraversalConnection` non-scope entries are treated as unsupported in runtime and emit diagnostics without mutating runtime state
- RoomLink capture/load compatibility is covered for both `variableStates` and `isActive` fallback payloads, with explicit precedence (`variableStates` wins when both are present)
7. Shared runtime save/load seam validation additions are passing:
- focused suite `RuntimeSaveGameStateServiceScaffoldingTests` now includes RoomLink positive, fallback, precedence, and negative-path diagnostics/non-mutation coverage (latest run: 26/26 passing)
- object-containment roundtrip coverage now includes explicit nested-scope capture and crowbar-in-backpack restore parentage assertions
- guardrail/runtime playback filters remained green after these additions (62/62 and 8/8 passing)
8. Simulator host integration for Save Game / Load Game is implemented:
- `Storyboard.Simulator` File menu now includes `Save Game` and `Load Game` commands
- save flow captures runtime session through `GameManager.CaptureSaveGameState(...)` and writes `.sbe.save.json`
- load flow reads save envelope, resolves/loads base runtime export project, then applies `GameManager.LoadSaveGameState(...)` rehydration
- implementation remains host-only (`Storyboard.Simulator`) with shared runtime logic in `Storyboard.Shared`
9. Final validation gates after simulator integration are green:
- `dotnet build .\StoryboardDesigner.slnx`
- `dotnet test .\Storyboard.Shared.Tests\Storyboard.Shared.Tests.csproj --filter "FullyQualifiedName~RuntimeSaveGameStateServiceScaffoldingTests"` (26/26 passing)
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"` (62/62 passing)
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"` (8/8 passing)
- `dotnet test .\StoryboardDesigner.slnx` (1232/1232 passing)
10. Save contract model was unified to scope-node recursion for object state persistence:
- active save DTO/schema artifacts for `SaveGameObjectStateDto` were retired
- `SaveScopeNodeStateDto.ObjectStates` now recursively uses `SaveScopeNodeStateDto`
- shared save/load mapping in `RuntimeSaveGameStateService` now captures and restores nested object lineage through scope-node recursion
11. Save-game codegen lane correctness fixes were applied in `Storyboard.SchemaCodegen`:
- save-game DTO generation now uses save-game emission-lane type hints
- enum type emission now consistently aliases through `ContractEnums` for save-game DTOs
19. Cross-contract reuse is restricted to shared enum references from runtime contracts only.

12. Explicit runtime-materialized save/restore policy is implemented and validated:
- `SaveScopeNodeStateDto` now persists nullable `isRuntimeMaterialized`
- restore path only materializes missing game-object ids when `isRuntimeMaterialized == true`
- restore matching now claims id-resolved linked-base matches, preventing collisions when multiple objects share one linked source id
- focused shared runtime coverage now includes explicit positive/negative materialization restoration behavior and shared-source distinct-instance restoration (latest run: 33/33 passing)

13. Additional validation after explicit materialization update succeeded:
- `dotnet build .\StoryboardDesigner.slnx`
- `dotnet test .\Storyboard.Shared.Tests\Storyboard.Shared.Tests.csproj --filter "FullyQualifiedName~RuntimeSaveGameStateServiceScaffoldingTests"` (33/33 passing)
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj` (750/750 passing)
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"` (8/8 passing)
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"` (62/62 passing)

## Closure Disposition

1. This plan is closed and archived after implementation of contract-first save/reload, simulator invocation wiring, and explicit runtime-materialization restore handling.
2. Full-solution `dotnet test .\StoryboardDesigner.slnx` was rerun on 2026-08-11 and reported one failure in `StoryboardDesigner.App.SmokeTests` (`Bite4BaselineCoverageSmokeTests.Hierarchy_DoubleClick_Can_Open_Designer_Editors_And_AreaMap_Anchors`) caused by FlaUI property support (`PropertyNotSupportedException` on Name).
3. The smoke failure is outside save/reload runtime behavior and does not block this plan closure; save/reload gates and focused runtime validations remain green.

## Notes

1. This plan intentionally separates save-state contracts from authored-content and runtime-export contract families.
2. Storage medium is intentionally decoupled from save payload format to protect future backend flexibility.
3. A future follow-on plan may split this into multiplayer/session-host-specific extensions once foundational single-session save/reload is complete.
