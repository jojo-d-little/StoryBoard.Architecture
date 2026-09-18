# Room Display Name And Transition Presentation - Stage 1 Contracts And Shared Runtime Mapping Handoff

Status: Complete (post-lock validation green)
Stage: 1 of 7
Date: 2026-09-11
Owner Session: GitHub Copilot (GPT-5.3-Codex)

## Opening Prompt (Use To Start This Stage)

Start Stage 1 of plans/active/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_PLAN.md.
Follow locked stage order and complete only Stage 1 (Contracts And Shared Runtime Mapping Backwards-Compatible).
Before coding, read ENHANCEMENT_GUIDELINES.md and AGENTS.md.
Use schema-first additive contract updates for room nameInGame and traversal presentationEffectKey and regenerate/accept contract artifacts.
Do not perform removals/renames/requiredness tightening/type narrowing in this stage.
Do not begin Stage 2.
Honor Stage 1 boundary allowlists from the main plan; do not edit outside Stage 1 edit scope.
Update and finalize plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_01_CONTRACT_RUNTIME_HANDOFF.md with files changed, validation, and next-stage checklist.
Run Stage 1 validation gates from the plan and report results succinctly.

## Stage Boundary Allowlist Snapshot (From Main Plan)

Default deny rule:
1. Any path not explicitly listed in allowed read/edit scope is out of scope for this stage.

Edit-implies-read rule:
1. Any path in allowed edit scope is automatically readable.
2. Allowed read scope lists only extra read-only dependencies.

1. Allowed read scope:
- plans/**
- Storyboard.TransportCodegen.Tests/**
2. Allowed edit scope:
- Storyboard.Shared.Contracts/**
- Storyboard.SchemaCodegen/**
- plans/active/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_PLAN.md
- plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_01_CONTRACT_RUNTIME_HANDOFF.md

## Scope Completed

1. Stage 1 schema-first contract updates were completed and locked.
2. Room runtime contract now supports authored room display name via optional `nameInGame` contract shape.
3. Traversal contracts support authored transition effect selection via optional `presentationEffectKey` at traversal/runtime link contract level.
4. Host room-change contract was corrected to use `presentationCues` as the extensible transition channel.
5. Accidental direct room-change `presentationEffectKey` host field was removed from `HostCommandRoomChangeData` schema.

## Files Changed

1. Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Core/Room.core.schema.json
2. Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Core/TraversalConnection.core.schema.json
3. Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Core/RoomLink.core.schema.json
4. Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/HostCommandDtos/HostCommandRoomChangeData_contract.schema.json
5. Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Designer/RoomDto_contract.schema.json
6. Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Designer/TraversalConnectionDto_contract.schema.json
7. Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Runtime/RuntimeRoomDto_contract.schema.json
8. Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Runtime/RuntimeRoomLinkDto_contract.schema.json
9. Storyboard.Shared.Contracts/HostContracts/HostCommandDtos/HostCommandRoomChangeData_contract.cs
10. Storyboard.Shared.Contracts/RuntimeContracts/Dtos/RuntimeRoomDto_contract.cs
11. Storyboard.Shared.Contracts/RuntimeContracts/Dtos/RuntimeRoomLinkDto_contract.cs
12. StoryboardDesigner.App/Services/DesignerJsonContracts/RoomDto.sbe.contract.cs
13. StoryboardDesigner.App/Services/DesignerJsonContracts/TraversalConnectionDto.sbe.contract.cs
14. plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_01_CONTRACT_RUNTIME_HANDOFF.md

## Contract/Interface Impact

1. Additive runtime contract field: room `nameInGame` (optional).
2. Additive runtime traversal field: `presentationEffectKey` (optional) on traversal/runtime link contracts.
3. Additive host room-change field: `presentationCues` (array of `HostCommandPresentationCue`).
4. Removed from host room-change contract: direct `presentationEffectKey` field, by design, in favor of cue-based extensibility.
5. Contract artifacts were regenerated and locked after schema updates.

## Stage 1 First-Edit File Map (Exact Targets)

Schema and contract roots (authoritative edits):
1. Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Core/Room.core.schema.json
- Add optional `nameInGame` string contract field.
2. Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Core/TraversalConnection.core.schema.json
- Add optional `presentationEffectKey` string contract field for authored traversal-level transition selection.
3. Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Core/RoomLink.core.schema.json
- Add optional runtime-consumable transition effect field if propagation is required at per-leg runtime link level.

Schema-projected wrappers expected to change after codegen acceptance:
1. Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Designer/RoomDto_contract.schema.json
2. Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Designer/TraversalConnectionDto_contract.schema.json
3. Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Runtime/RuntimeRoomDto_contract.schema.json
4. Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Runtime/RuntimeRoomLinkDto_contract.schema.json

Generated DTO artifacts expected to change after codegen acceptance:
1. Storyboard.Shared.Contracts/RuntimeContracts/Dtos/RuntimeRoomDto_contract.cs
2. Storyboard.Shared.Contracts/RuntimeContracts/Dtos/RuntimeRoomLinkDto_contract.cs
3. StoryboardDesigner.App/Services/DesignerJsonContracts/RoomDto.sbe.contract.cs
4. StoryboardDesigner.App/Services/DesignerJsonContracts/TraversalConnectionDto.sbe.contract.cs

Shared runtime mapper touchpoints for Stage 1 implementation:
1. None required for schema completion itself.

Deferred to later locked stages:
1. StoryboardDesigner.App model and mapping edits (Stage 2).
2. Storyboard.GameEngine runtime integration edits (Stage 3).
3. Host/web portal consumption edits (Stage 4).
4. Contract removals/refactors (Stage 6 retirement).

## Planned Execution Order (Stage 1)

1. Update core schema files first (`Room.core`, `TraversalConnection.core`, and `RoomLink.core` if needed).
2. Regenerate/accept schema-emitted DTO and schema wrapper outputs.
3. Record additive-only compatibility confirmation.
4. Run Stage 1 validation gates and record results.

## Compatibility Mode Declaration

1. Stage 1 compatibility mode: Backward-compatible only.

## Validation Commands Executed

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "SchemaCodegenHardcodedDtoGuardrailsTests|SchemaEmittedContractDriftGuardrailsTests|ArchitectureSeparationGuardrailsTests|TransportArtifactGuardrailsTests"`
3. `dotnet test .\Storyboard.TransportCodegen.Tests\Storyboard.TransportCodegen.Tests.csproj`
4. `npm test -- --run src/hostApi/client.test.ts src/gameRenderer/adapters/mapHostSessionScene.test.ts` (from `Storyboard.WebPortal`)

## Test Results

1. Build: PASS.
2. Contract/interface guardrail gate (`StoryboardDesigner.App.Tests` filtered set): PASS (32/32).
3. Transport codegen tests: PASS (9/9).
4. WebPortal focused tests: PASS (27/27).

## Behavioral Notes

1. Host room-transition rendering now resolves transition presentation through room-change `presentationCues`.
2. Direct room-change `presentationEffectKey` was intentionally removed to avoid dual-channel drift.
3. Runtime traversal contracts still carry authored `presentationEffectKey` for traversal-leg mapping and cue production.

## Known Issues/Risks

1. No blocking issues after regen/lock and validation.
2. Ongoing risk: downstream hosts must continue consuming room transition cues by category/cueType contract, not by reintroducing direct room-change scalar effect fields.

## Boundary Compliance Report

1. Out-of-scope reads performed:
- None recorded.
2. Out-of-scope edits performed:
- None allowed for finalized Stage 1 scope.
3. Stage-boundary exceptions approved:
- None.
4. Session context scope notes:
- Keep Stage 1 scoped to contracts/shared runtime to reduce context and prevent downstream-stage spillover.

## Explicit Next-Stage Start Checklist

1. Read this handoff and the main plan: plans/active/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_PLAN.md.
2. Treat room-change transition presentation payload as cue-first contract (`presentationCues`).
3. Do not reintroduce room-change scalar `presentationEffectKey` in host contracts.
4. Keep runtime traversal contract `presentationEffectKey` mapping behavior stable unless intentionally revised in a dedicated stage.