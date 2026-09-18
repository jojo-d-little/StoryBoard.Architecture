# Command/Action Scar Tissue Retirement Plan

## Goal
Retire legacy room-command DTO/schema artifacts that predate the current action-first runtime model, while preserving modern action behavior and avoiding contract regressions.

## Why
- Command/action overlap currently creates confusing contract shape and maintenance risk.
- Legacy command DTOs appear to persist mostly placeholder payloads and are weakly connected to modern action execution.
- Cleanup will simplify runtime and designer contract surfaces.

## Confirmed Active vs Legacy Boundary
### Keep (active, action-first)
- CommandActionDto family (designer contracts).
- RuntimeCommandActionDto family (runtime contracts).
- AvailableGameActions runtime/designer mapping and execution paths.

### Candidate removal (legacy command-era)
- RoomCommandDto family.
- RuntimeRoomCommandDto family.
- RoomCommand core schema.
- RuntimeCommandActionLinkDto family.
- Room commandPhrases contract fields and related mapper glue.

## Scope
### In scope
- Schema and generated contract cleanup for legacy command DTOs.
- Export/import/runtime mapper cleanup tied to removed command DTOs.
- Runtime command-processing cleanup for explicit commandPhrase linkage path if no longer needed.
- Snapshot/test updates required by contract changes.

### Out of scope
- New gameplay command features.
- Vocabulary/grammar redesign.
- Broad UI redesign unrelated to command/action contract cleanup.

## Risks
- Breaking legacy project/runtime JSON load paths.
- Removing explicit phrase matching behavior if any scenarios still depend on it.
- Contract lock drift across runtime/designer manifests.

## Lock-Off Questions (must answer before deletion slice)
Question count: 10

1. Do we keep a temporary read-compatibility shim for legacy `commandPhrases` payloads?
2. If compatibility is kept, what is the exact deprecation window and required diagnostic message contract?
3. Is explicit room commandPhrase -> linked action dispatch intentionally removed, or retained behind compatibility mapping?
4. Should writer output drop `commandPhrases` immediately or after one transition window?
5. Are `RuntimeRoomCommandDto` and `RuntimeCommandActionLinkDto` both approved for full retirement from runtime contracts in the same slice?
6. Is `RoomCommandDto` approved for full retirement from designer contracts, including removal from `RoomDto` schema containment?
7. Do we require loader compatibility for previously exported runtime JSON containing legacy `commandPhrases`/`actionLinks`, or is re-export from authored `.sbe` the only supported migration path?
8. If loader compatibility is required, should legacy commandPhrase records be ignored with warning, transformed into action triggers, or treated as hard errors?
9. Are sample/runtime snapshot artifacts expected to remove `commandPhrases` shape immediately once contracts change, or do we preserve placeholders for one transition cycle?
10. What is the explicit cutover signal to start lock workflow updates (runtime contract lock, designer contract lock, enum lock if affected) for this plan?

## Lock-Off Decisions To Date
1. Keep temporary read-only compatibility handling for legacy `commandPhrases` only as needed to complete cutover safely; no long-lived legacy support track.
2. Deprecation window is hard-cut oriented: once migration-ready code lands and sample regeneration is complete, legacy support is removed immediately.
3. Explicit room commandPhrase -> linked-action dispatch is removed; runtime command execution is action-first only.
4. Writer output drops `commandPhrases` immediately in the same cutover change set.
5. `RuntimeRoomCommandDto` and `RuntimeCommandActionLinkDto` are retired together in one runtime-contract slice.
6. `RoomCommandDto` is retired fully, including removal from `RoomDto` schema containment.
7. Migration path is one-time re-export from authored `.sbe` sources; legacy runtime JSON artifacts are not an ongoing compatibility target.
8. Because loader compatibility is not retained, encountering legacy commandPhrase/actionLinks payloads after cutover is treated as a hard migration-required error.
9. Sample/runtime snapshot artifacts remove `commandPhrases` shape immediately; stale artifacts must be regenerated/updated in the same change wave.
10. Cutover signal for lock workflow updates is: schema + mapper/runtime code changes complete, sample regeneration complete, then run lock workflow updates and guardrail/full validation in the same slice.

## Current Status
- Lock-off: Complete (10/10 questions answered).
- Slice 0 baseline: Complete (2026-08-08).
- Slice 2 schema + DTO retirement: Complete (2026-08-08).
- Slice 3 mapper/runtime cleanup: Complete (2026-08-08).
- Slice 4 regression + snapshot closure: Complete (2026-08-08).
- Baseline evidence:
	- `dotnet build .\StoryboardDesigner.slnx` -> pass.
	- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "SchemaEmittedContractDriftGuardrailsTests"` -> pass (4/4).
	- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"` -> pass (62/62).
	- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"` -> pass (8/8).
- Closure evidence:
	- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~JsonExportServiceRuntimeExportValidationTests"` -> pass (2/2).
	- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "SchemaEmittedContractDriftGuardrailsTests"` -> pass (4/4).
	- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"` -> pass (8/8).
	- `dotnet build .\StoryboardDesigner.slnx` -> pass.
	- `dotnet test .\StoryboardDesigner.slnx` -> pass (1156/1156).

## Proposed Slices
### Slice 0: Safety Baseline
- Capture baseline validation and confirm all guardrail tests are green before cleanup.
- Freeze an artifact inventory of all files/schemas referencing legacy command DTOs.

### Slice 1: Contract Shape Decision
- Finalize lock-off answers above.
- Decide immediate removal vs compatibility-window behavior.

### Slice 2: Schema + DTO Retirement
- Remove legacy schemas and generated DTO artifacts selected in Slice 1.
- Update room schemas/wrappers to drop commandPhrases if approved.
- Update lock manifests after regeneration/lock workflow.

### Slice 3: Mapper/Runtime Cleanup
- Remove export/import conversion helpers for legacy command DTOs.
- Remove runtime explicit commandPhrase dispatch path if approved.
- Keep action-trigger matching path intact and validated.

### Slice 4: Regression + Snapshot Closure
- Refresh affected snapshots/fixtures intentionally.
- Run focused and full test gates.
- Confirm no stale lock-manifest rows or orphan generated files remain.

## Validation Gates
1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "SchemaEmittedContractDriftGuardrailsTests"
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"
4. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
5. dotnet test .\StoryboardDesigner.slnx

## Exit Criteria
- Legacy command DTO/schema artifacts are removed or explicitly compatibility-scoped with deprecation plan.
- Runtime command execution relies on action-first pathways with no regression.
- Contract lock guardrails pass for runtime/designer/enum outputs.
- Full solution test suite passes.
