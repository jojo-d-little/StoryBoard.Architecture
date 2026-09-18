# Room Display Name And Transition Presentation - Stage 4 Host Interface And Web Portal Runtime Consumption Handoff

Status: Complete
Stage: 4 of 7
Date: 2026-09-11
Owner Session: GitHub Copilot (GPT-5.3-Codex)

## Opening Prompt (Use To Start This Stage)

Start Stage 4 of plans/active/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_PLAN.md.
Read Stage 1 contracts/runtime handoff, Stage 2 designer handoff, and Stage 3 GameEngine handoff first, then complete only Stage 4 (Host Interface And Web Portal Runtime Consumption).
Do not begin Stage 5.
Honor Stage 4 boundary allowlists from the main plan; do not edit outside Stage 4 edit scope.
Update this handoff with files changed, validation results, behavioral notes, and explicit next-stage checklist.

## Stage Boundary Allowlist Snapshot (From Main Plan)

Default deny rule:
1. Any path not explicitly listed in allowed read/edit scope is out of scope for this stage.

Edit-implies-read rule:
1. Any path in allowed edit scope is automatically readable.
2. Allowed read scope lists only extra read-only dependencies.

1. Allowed read scope:
- plans/**
2. Allowed edit scope:
- Storyboard.Shared/**
- Storyboard.Shared.Contracts/**
- Storyboard.WebPortal/**
- Storyboard.WebPortal.Tests/**
- plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_04_HOST_WEBPORTAL_HANDOFF.md

## Compatibility Mode Declaration

1. Stage 4 compatibility mode: N/A (consumer adaptation stage).

## Scope Completed

1. Updated web portal room-transition cue resolution to consume runtime-authored traversal transition cues from host room-change payloads.
2. Implemented cue-first precedence in web portal transition selection:
- explicit dev/manual cue selection still wins when set,
- otherwise runtime-authored room-transition cue effect key is used when catalog-resolved,
- otherwise existing web defaults remain fallback.
3. Added Stage 4 regression tests to lock cue precedence and unresolved-key fallback behavior.
4. Added explicit dual-fade support with clear per-cue control:
- existing crossfade behavior remains available,
- new blackout-swap fade behavior is selectable by cue key.
5. Implemented renderer sequencing for blackout-swap fade so room-size changes can fade to full blackout, switch bounds/resolution, then fade the incoming room in.
6. Verified required Stage 4 validation gates for transport codegen and web portal test project.
7. Added explicit warning diagnostics when runtime requests a room-transition cue key that is not present in the loaded catalog, while preserving global fallback behavior.

## Files Changed

1. Storyboard.WebPortal/src/hooks/useRoomTransitionCueWorkflow.ts
2. Storyboard.WebPortal/src/hooks/useRoomTransitionCueWorkflow.test.tsx
3. Storyboard.WebPortal/src/gameRenderer/adapters/mapHostSessionScene.ts
4. Storyboard.WebPortal/src/gameRenderer/contracts/sceneTypes/GameRenderRoomTransition.ts
5. Storyboard.WebPortal/src/gameRenderer/presentationCue/resolveMovementCueDuration.ts
6. Storyboard.WebPortal/src/gameRenderer/presentationCue/resolveMovementCueDuration.test.ts
7. Storyboard.WebPortal/src/gameRenderer/pixi/PixiGameRenderer.ts
8. Storyboard.WebPortal/src/components/DevToolsPanel.tsx
9. Storyboard.GameEngine/Config/presentation-effects.catalog.json
10. Storyboard.WebPortal/src/hooks/useHostRendererSessionWorkflow.ts
11. plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_04_HOST_WEBPORTAL_HANDOFF.md

## Contract/Interface Impact

1. No shared schema or generated contract DTO files were edited in Stage 4.
2. No host/runtime interface signatures were changed.
3. Consumer behavior update only in web portal transition resolution:
- runtime room-change cue effect key is now consumed when provided and resolvable,
- settings defaults continue as fallback when traversal-level cue is missing/unresolved.
4. Added transition-mode interpretation for `BlackoutSwapFade` to support explicit cue-key selection between crossfade and blackout-swap fade.

## Validation Commands Executed

1. `dotnet test .\Storyboard.TransportCodegen.Tests\Storyboard.TransportCodegen.Tests.csproj`
2. `dotnet test .\Storyboard.WebPortal.Tests\Storyboard.WebPortal.Tests.csproj`
3. `npm test -- --run src/hooks/useRoomTransitionCueWorkflow.test.tsx` (from `Storyboard.WebPortal`)
4. `npm run build` (from `Storyboard.WebPortal`)
5. `npm test -- --run src/gameRenderer/presentationCue/resolveMovementCueDuration.test.ts src/hooks/useRoomTransitionCueWorkflow.test.tsx` (from `Storyboard.WebPortal`)
6. `dotnet test .\Storyboard.TransportCodegen.Tests\Storyboard.TransportCodegen.Tests.csproj` (post dual-fade update re-run)
7. `dotnet test .\Storyboard.WebPortal.Tests\Storyboard.WebPortal.Tests.csproj` (post dual-fade update re-run)
8. `npm run build` (post diagnostics warning hardening update, from `Storyboard.WebPortal`)

## Test Results

1. `Storyboard.TransportCodegen.Tests`: PASS (9 passed, 0 failed, 0 skipped).
2. `Storyboard.WebPortal.Tests`: PASS (13 passed, 0 failed, 0 skipped).
3. Web portal targeted hook tests (`useRoomTransitionCueWorkflow.test.tsx`): PASS (7 passed, 0 failed, 0 skipped).
4. Web portal production build (`npm run build`): PASS (TypeScript + Vite build succeeded).
5. Web portal targeted transition tests (`resolveMovementCueDuration.test.ts` + `useRoomTransitionCueWorkflow.test.tsx`): PASS (46 passed, 0 failed, 0 skipped).
6. Post-update Stage 4 required gate rerun: PASS (`Storyboard.TransportCodegen.Tests` 9/9, `Storyboard.WebPortal.Tests` 13/13).
7. Post-diagnostics hardening web portal build: PASS (`npm run build`).

## Behavioral Notes

1. Web portal transition selection now treats runtime room-change transition cue as authoritative authored intent when it resolves in the loaded presentation cue catalog.
2. If the runtime-authored transition cue is absent or unknown to the catalog, the resolver falls back to existing `roomTransitionDefaults` logic (including travel-direction overrides).
3. Room label consumption remains sourced from `roomChange.newRoom.name`, which is already player-facing after Stage 3 engine fallback semantics.
4. Type-level room-transition cue mapping in host session scene adapter was tightened to handle optional `presentationCues` safely, keeping npm build/tsc green.
5. Crossfade and blackout-swap fade are now distinct behaviors controlled by effect key through `roomTransitionPresentation.mode`:
- `CrossFade` keeps concurrent outgoing/incoming blend,
- `BlackoutSwapFade` performs outgoing fade to blackout, swaps bounds at midpoint, then fades incoming room in.
6. When runtime sends a room-transition cue effect key that does not resolve in the active catalog, web portal now emits a clear warning diagnostic and continues with configured fallback transition settings.

## Known Issues/Risks

1. If the host provides a runtime-authored cue key that is not in the currently loaded catalog, web portal intentionally falls back to configured defaults; catalog/runtime drift can therefore mask authored override intent.
2. Manual/dev cue override remains higher priority when selected, which is intentional for diagnostics but should be considered when visually validating authored cues.
3. The new blackout-swap cue must be present in the active presentation-effects catalog used by runtime content; if absent, selection falls back through existing default cue logic and now produces an explicit warning diagnostic.

## Boundary Compliance Report

1. Out-of-scope reads performed:
- ENHANCEMENT_GUIDELINES.md (required repository entry-point policy pre-read).
- .github/instructions/storyboard-shared-runtime.instructions.md (instruction file read for shared-scope compliance awareness).
- /memories/repo/build-notes.md (repository memory consultation).
2. Out-of-scope edits performed:
- Storyboard.GameEngine/Config/presentation-effects.catalog.json (user-directed addition of a new room-transition cue key for explicit dual-fade selection semantics).
3. Stage-boundary exceptions approved:
- User-directed implementation follow-up after Stage 4 completion to add explicit dual-fade cue control.
4. Session context scope notes:
- Web portal implementation/test edits remained in Stage 4 scope; one user-directed catalog edit was applied outside Stage 4 allowlist for cue-definition parity.

## Explicit Next-Stage Start Checklist

1. Confirm Stage 1, Stage 2, and Stage 3 handoffs were reviewed before making edits.
2. Consume room-change transition cues from `roomChange.presentationCues` (category `RoomTransition`) and avoid introducing new scalar transition fields.
3. Treat `roomChange.NewRoom.Name` as player-facing value already resolved by engine fallback logic.
4. Preserve web defaults as fallback when traversal-level cue is missing/unresolved.
5. Preserve Stage 4 cue precedence semantics in simulator parity work: manual/dev override (if used) > runtime-authored cue when resolvable > configured defaults.
6. Begin Stage 5 only in `Storyboard.Simulator/**`, `Storyboard.Simulator.Tests/**`, `Storyboard.Simulator.SmokeTests/**`, and its stage handoff file per Stage 5 boundary allowlist.
