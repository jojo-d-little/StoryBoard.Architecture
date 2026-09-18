# Portrait Room Orientation - Stage 1 Contract Handoff

Status: Complete
Stage: 1 of 5
Date: 2026-09-09
Owner Session: GPT-5.3-Codex

## Scope Completed
- Updated room core schema contract to support room-level render dimensions via `roomImageCanvasWidth` and `roomImageCanvasHeight`.
- Removed room-level cell-size override contract property (`roomGridCellSizeOverride`) from room core schema.
- Updated host room-change new-room summary schema to carry optional room-effective width/height for host client hydration.
- Kept project-level defaults (`roomImageCanvasWidth`, `roomImageCanvasHeight`) and project-level cell size (`roomDesignerGridCellSize`) unchanged.

## Files Changed
- Storyboard.Shared.Contracts/RuntimeContracts/Schemas/DtoContracts/Core/Room.core.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/HostCommandDtos/HostCommandNewRoomSummary_contract.schema.json
- Storyboard.Shared.Contracts/RuntimeContracts/Dtos/RuntimeRoomDto_contract.cs
- Storyboard.Shared.Contracts/HostContracts/HostCommandDtos/HostCommandNewRoomSummary_contract.cs

## Contract/Interface Impact
- Any contract composed from `Room.core.schema.json` now carries optional room-level width/height properties.
- Any contract composed from `Room.core.schema.json` no longer carries `roomGridCellSizeOverride`.
- This affects both Designer and Runtime room DTO schemas through schema composition.
- Host room-change payload contracts now include optional `roomImageCanvasWidth` and `roomImageCanvasHeight` on `newRoom` summary payload.

## Validation Commands Executed
- Regeneration and lock workflow completed by user review session.
- Observed terminal execution includes lock step: `Storyboard.SchemaCodegen lock-designer-contract` (exit code 0).
- Full lock command transcript and any additional codegen commands were not captured in this handoff session.

## Test Results
- No new Stage 1 test execution captured in this handoff session.
- Stage 2 kickoff should run normal build/tests before implementation continues.

## Behavioral Notes
- Contract shape now supports per-room bounds as the room-level source of truth target.
- Project-level cell size remains the only supported cell-size source in contract shape.

## Known Issues/Risks
- No blocking Stage 1 contract issues identified after regeneration review.
- Downstream compile/test fallout may still exist until Stage 2 and Stage 3 consumer mappings are updated.

## Next-Stage Start Checklist (Designer)
1. Review contract field additions/removals.
2. Confirm Designer room model reads/writes room-level `roomImageCanvasWidth` and `roomImageCanvasHeight`.
3. Remove any remaining Designer assumptions about room-level cell-size override.
4. Implement validation behavior per design lock: save allowed; run/export blocked on validation errors.
5. Run build + targeted tests and append results to Stage 2 handoff.
