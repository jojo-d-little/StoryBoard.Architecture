# Portrait Room Orientation - Stage 4 WebPortal Handoff

Status: Complete
Stage: 4 of 5
Date: 2026-09-10
Owner Session: GPT-5.3-Codex

## Stage 4 Inputs (Verified from Stage 1-3)
1. Stage 1 contracts establish room-level bounds fields and host new-room payload fields:
- roomImageCanvasWidth
- roomImageCanvasHeight
2. Stage 2 producer behavior establishes authored-source truth and fallback expectations:
- room-level width/height are active-room truth once explicit.
- project-level width/height are defaults for creation/fallback only.
3. Stage 3 runtime behavior confirms host payload production and mutation parity:
- room-change new-room summary now emits roomImageCanvasWidth/roomImageCanvasHeight.
- mutation summary clone/capture preserves roomImageCanvasWidth/roomImageCanvasHeight.
- movement bounds resolution prioritizes canonical room dimension variable names.

## Stage 4 Goal
Use roomImageCanvasWidth/roomImageCanvasHeight from runtime host payloads as WebPortal active-room bounds truth for rendering and interaction mapping, including mixed-aspect room transitions.
This includes correct click/tap translation into normalized room-space coordinates used by host/game engine point-intent flows.

## Scope Completed
1. Added WebPortal host contract typing for room-level bounds on new-room summary (`roomImageCanvasWidth`, `roomImageCanvasHeight`).
2. Added WebPortal host API client parsing for room-level bounds with positive-int normalization.
3. Updated scene snapshot mapping to use room-level bounds first, authored defaults second, and hardcoded fallback last.
4. Added adapter regression coverage for bounds precedence and invalid/missing room-dimension fallback.
5. Added contain-scaling regression coverage for click/tap mapping in mixed-aspect (portrait room in landscape viewport) scenarios.
6. Updated host API client tests to verify round-trip parsing of new-room room-level bounds in delta and baseline payloads.
7. Added sequential mixed-aspect transition regression coverage (800x600 -> 600x800 -> 800x600) to assert bounds refresh and prevent stale carryover.

## Stage 4 Implementation Checklist
1. Consume new-room summary roomImageCanvasWidth/roomImageCanvasHeight in the WebPortal room-state hydration pipeline.
2. Ensure active room dimensions are updated on each room transition before render and hit-test computations.
3. Update renderer sizing/layout logic to use active-room dimensions, not project defaults, when room values are present.
4. Update interaction coordinate mapping (click/tap to room coordinates) to use active-room dimensions.
5. Confirm interaction payload coordinates emitted to host/game engine match expected normalized room-space values after contain scaling and letterbox offsets.
6. Preserve deterministic fallback behavior when payload room dimensions are missing:
- fall back to project defaults provided by runtime/host contract paths.
7. Verify mixed-dimension transitions (for example, 800x600 to 600x800 and reverse) do not retain stale bounds.
8. Validate directional overlay/slot alignment under quarter-turn rotations (0/90/180/270) to prevent anchor drift.
9. Add a click-mapping matrix that asserts normalized room-space coordinate correctness for representative points (corners, center, near letterbox edges) across the MVP mixed-size transition matrix.

## Files Changed
1. Storyboard.WebPortal/src/hostApi/HostContracts/HostSessionDataEnvelope.ts
2. Storyboard.WebPortal/src/hostApi/client.ts
3. Storyboard.WebPortal/src/gameRenderer/adapters/mapHostSessionScene.ts
4. Storyboard.WebPortal/src/gameRenderer/adapters/mapHostSessionScene.test.ts
5. Storyboard.WebPortal/src/hostApi/client.test.ts
6. Storyboard.WebPortal/src/gameRenderer/scaling/containScaling.test.ts
7. Storyboard.WebPortal/visual-tests/directional-room-layout.spec.ts-snapshots/directional-room-layout-win32.png

## Contract/Interface Impact
1. No new contract changes are expected in Stage 4.
2. Stage 4 should consume existing Stage 1 host payload fields:
- roomImageCanvasWidth (optional int)
- roomImageCanvasHeight (optional int)
3. Do not reintroduce roomGridCellSizeOverride semantics in host UI bounds logic.

## Validation Commands Executed
1. npm test -- --run src/hostApi/client.test.ts src/gameRenderer/adapters/mapHostSessionScene.test.ts src/gameRenderer/scaling/containScaling.test.ts
- PASS (36 passed, 0 failed)
2. npm run build
- PASS
3. npx playwright test visual-tests/directional-room-layout.spec.ts --update-snapshots
- PASS (baseline snapshot updated)
4. npx playwright test visual-tests/directional-room-layout.spec.ts
- PASS
5. Upstream evidence (already complete):
- Stage 3 focused GameEngine suite: pass (55/55).
- Runtime-focused app gate: pass (66/66).
- Transport codegen tests: pass (9/9).

## Stage 4 Validation Plan
1. Build:
- npm run build (Storyboard.WebPortal)
2. WebPortal tests (targeted first, then broader suite as needed):
- run room transition and interaction-mapping tests that exercise mixed room dimensions.
- include explicit click/tap -> normalized room-space assertions for each mixed-dimension case.
3. Cross-stage confidence checks:
- verify room-change payload handling reflects roomImageCanvasWidth/roomImageCanvasHeight.
- verify no stale-dimension behavior after consecutive room transitions.
4. Manual/visual smoke for mixed-aspect rooms:
- transition landscape -> portrait -> landscape.
- confirm render bounds and clickable regions remain aligned in each room.
- confirm host command point intents receive expected normalized room-space x/y values for sampled clicks.

## Test Results
1. Host API client tests now verify room-level bounds parsing on `newRoom` for both session delta and baseline payloads.
2. Scene adapter tests now verify room-level bounds precedence over authored defaults and fallback behavior when room values are invalid/missing.
3. Contain-scaling tests now verify mixed-aspect click/tap mapping center-point correctness and side-letterbox clamping semantics.
4. Focused Stage 4 test command pass: 36/36.
5. Directional visual baseline snapshot comparison now passes after baseline acceptance/update.
6. Scene adapter tests now include a sequential transition assertion that room bounds rebind correctly across landscape -> portrait -> landscape room changes.

## Behavioral Notes
1. Stage 4 source of truth for active-room render/interaction bounds is room-level dimensions carried in runtime host payloads.
2. Project-level room dimensions are defaults/fallbacks only and must not override explicit room-level values.
3. Orientation remains derived from width/height, not a separately persisted runtime flag.
4. Mixed-aspect transitions are a primary Stage 4 regression axis and require explicit verification.
5. Click/tap translation into normalized room-space coordinates is a primary Stage 4 regression axis and must be verified under mixed-aspect transitions.
6. Manual POV review found no perceptible visual difference between prior expected vs actual directional baseline images; visual drift was accepted as snapshot noise and the baseline was refreshed.

## Known Issues/Risks
1. Primary risk is stale or partially updated bounds across room transitions causing render-hit-test mismatch.
2. Primary visual risk is directional overlay anchor drift if rotation is applied at render-time without layout-aware sizing.
3. Legacy compatibility assumptions around roomGridCellSizeOverride can cause accidental fallback coupling if reused in WebPortal bounds paths.
4. Primary interaction risk is using stale bounds for contain-transform inversion, producing incorrect normalized room-space click coordinates after room transitions.
5. Mixed-size room slide transition aesthetics can still look odd in some transitions; this is intentionally deferred to a separate follow-up effort (transition-mode selection/polish).

## Stage 4 Acceptance Criteria (Exit Gate)
1. WebPortal consumes roomImageCanvasWidth/roomImageCanvasHeight from new-room payloads for active-room bounds.
2. Rendered room bounds update correctly across mixed-dimension transitions without stale dimensions.
3. Interaction mapping (click/tap targeting) remains correct across mixed-dimension transitions.
4. WebPortal emits expected normalized room-space x/y values to host/game engine point-intent flows across mixed-dimension transitions.
5. No contract changes are required to complete Stage 4.
6. Stage 4 validation commands and results are recorded in this file.

## Stage Closure Decision
1. Decision: Stage 4 is closed as Complete on 2026-09-10.
2. Rationale: adapter bounds precedence, host payload parsing, mixed-aspect interaction mapping, sequential stale-bounds regression coverage, and directional visual baseline verification all passed.
3. Deferred item (non-blocking): mixed-size transition aesthetic polish remains tracked as a separate follow-up and is not part of this MVP correctness gate.
4. Readiness for Stage 5: WebPortal now presents a stable contract-consumer baseline for simulator parity checks against room-effective bounds and interaction mapping behavior.

## Next-Stage Start Checklist (Simulator)
1. Confirm expected mixed-aspect visual behavior alignment.
2. Confirm interaction bounds assumptions.
3. Confirm remaining parity items for simulator MVP.
4. Reuse Stage 4 behavioral expectations for normalized room-space mapping as parity assertions where simulator click/hit-test interactions are enabled.
