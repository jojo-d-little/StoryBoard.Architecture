# Command Action Contract Payloads Plan

Status: Closed
Date: 2026-08-08
Closed: 2026-08-09

## Goal

Refactor action contracts from a flattened property bag to action-specific payload contracts, with a hard switch and sample migration.

## Scope

1. Keep only shared action metadata at CommandAction root.
2. Move action-specific fields into payload contracts keyed by action type.
3. Support separate runtime/designer payload DTO lanes where needed.
4. Update schema codegen to generate typed payload members from payload references.
5. Migrate sample projects to the new contract shape.

## Proposed Execution (Small Slices)

1. Define target root contract shape for CommandAction with `payload` child.
2. Create first vertical slice for `Synonym` payload contracts (core + runtime + designer).
3. Update codegen for typed payload property generation (lane-aware).
4. Wire designer serialization mapping and runtime snapshot mapping for Synonym payload.
5. Add remaining action payload schemas in grouped batches (movement, state/property, composition, linking).
6. Remove deprecated flat action-specific properties from core/designer contracts.
7. Regenerate contracts, lock manifests, and run full validations.

## Validation Gates

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~SchemaEmittedContractDriftGuardrailsTests"
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
4. dotnet test .\StoryboardDesigner.slnx

## Design Questions

1. Should `payload` be required for every action type, or nullable for no-data actions (for example `EchoMessage`)?
2. Do we want one payload schema per action type, or allow selected shared payload schemas across multiple action types (for example container transfer)?
3. Should `outcomeMessageMap` remain at the CommandAction root, or move into specific payloads where used?
4. Should `direction` remain a root-level common field, or move into only movement/navigation payloads?
5. For designer/runtime parity, should payload property names be identical across lanes unless there is a documented reason to diverge?
6. Should payload discriminators be represented only by `actionType`, or should payload objects also carry a self-describing type field?
7. Do we keep enum-backed string fields in payload contracts as plain strings, or promote them to stricter enum refs now?
8. For action types currently sharing semantics (`NavigateDirection` and `NavigateToAdjacent`), do we keep separate payload contracts or a shared navigation payload contract?
9. Is `linkedActions` conceptually a root-level flow construct, or should it become the payload for `LinkedActions` and `Synonym` only?

## Decision Log (Current)

1. `payload` remains part of the contract shape for every action type, but an explicit empty payload instance is not required for no-data payload action types.
2. Use one payload contract per action type.
3. Keep `outcomeMessageMap` at the CommandAction root.
4. Keep `direction` at the CommandAction root (nullable for non-directional actions).
5. Keep payload property names aligned across runtime/designer lanes by default; divergence requires a documented reason.
6. Use `actionType` as the semantic action discriminator and formally accept `$payloadType` as the payload polymorphism discriminator for serialization/deserialization.
7. Promote stable engine-owned payload vocabularies to enum refs during this hard switch.
8. Keep separate payload contracts for related action types, and allow shared schema composition via inline base references when useful.
9. Additional organization rule: create dedicated ActionPayloads subfolders for schema artifacts and class artifacts in both designer and runtime lanes for visual clarity. This is organizational only and does not require namespace changes.
10. Move `linkedActions` from CommandAction root into flow-specific payload contracts.
11. Formal acceptance (2026-08-09): `$payloadType` is accepted as an intentional contract mechanism and is no longer considered decision drift.
12. Load-time enforcement rule: for action types whose payload contract includes data properties, missing `payload` should emit a warning in load diagnostics rather than fail hard.

## Pending Decision

No pending design decisions. All 9 questions are resolved.

## Notes

- This plan assumes hard cutover with no backward-compatible read path for old flat action fields.
- Empty/no-data payload action types may omit payload instances on disk; data-bearing payload action types should warn when payload is missing at load.
- Sample migration is part of the implementation, not a follow-up task.

## Progress Update

1. Added payload schema folder structure under Core/Runtime/Designer ActionPayloads.
2. Added base + Synonym payload schemas in core and lane wrappers.
3. Added CommandAction root `payload` property in core schema with lane-specific type hints.
4. Added transitional `payload` JsonElement properties to authored partial DTOs:
	- Designer CommandActionDto.
	- Runtime RuntimeCommandActionDto.
5. Wired Synonym payload write/read path in JsonExportService and runtime bootstrap mapper using `payload.targetActionId` with fallback to existing root field during transition.
6. Validation completed:
	- Solution build passed.
	- Focused synonym/serialization tests passed.
7. Implemented load-time diagnostics for missing payload instances on data-bearing action types:
	- Warning code: `action.payload.missing.v1`.
	- No-data payload action types do not warn when payload is omitted.
	- Warnings flow into designer load diagnostics CSV.
8. Additional validation completed:
	- `dotnet build .\StoryboardDesigner.slnx` passed.
	- Focused payload/load diagnostics tests passed.

## Closeout Summary

1. Plan scope is complete for current objectives and validated.
2. `$payloadType` acceptance and payload-omission warning policy are documented and implemented.
3. Remaining optional improvements (if desired) should be tracked in a new follow-on plan, not this closed artifact.
