# Pixi Game Renderer Plan

Purpose: establish a phased, testable migration path from the WPF simulator renderer to a Pixi 8-based web renderer while preserving runtime-host contract behavior.

## Implementation Principle

WPF is a functional reference only, not an implementation template. The Pixi renderer should be built using Pixi-native architecture, lifecycle, and rendering patterns while matching required gameplay outcomes and contract semantics.

## Goals

1. Preserve runtime ownership of room/command/session truth.
2. Keep host contract boundaries stable during renderer migration.
3. Deliver visual value early with directional room image overlays.
4. Avoid mixing large refactors with behavioral changes in the same phase.
5. Achieve functional parity without reproducing WPF-specific UI composition, control patterns, or rendering mechanics.

## Non-Goals (Initial Phases)

1. No runtime command grammar changes.
2. No host API contract rewrites.
3. No audio subsystem migration before visual baseline parity.
4. No browser-side parsing of runtime/export/project files in thin mode.
5. No one-to-one port of WPF view/viewmodel rendering patterns into web code.

## Pixi-Native Design Rules

1. Keep scene graph ownership in Pixi containers/sprites, not WPF-style view wrappers.
2. Use renderer-neutral state objects and project them into Pixi drawables.
3. Prefer texture/sprite lifecycle management, pooling, and z-index sorting idioms native to Pixi.
4. Use browser-appropriate asset loading/caching semantics while honoring host API boundaries.
5. Match behavior contracts and visual outcomes, not desktop implementation details.

## Renderer Architecture Charter

### 1) Module Map (WebPortal)

Create and keep all game-renderer code under one root so reviewers always know where to look.

Proposed structure:

1. src/gameRenderer/
2. src/gameRenderer/contracts/
3. src/gameRenderer/scene/
4. src/gameRenderer/pixi/
5. src/gameRenderer/adapters/
6. src/gameRenderer/interaction/
7. src/gameRenderer/diagnostics/
8. src/gameRenderer/testing/

Responsibilities:

1. src/gameRenderer/contracts/
  - Renderer-neutral TypeScript types (no DOM/Pixi/WPF types).
  - Examples: scene node primitives, room dimensions, overlay image payloads.
2. src/gameRenderer/scene/
  - Pure transform logic from host/orchestration state into renderer-neutral scene snapshots.
  - Deterministic sorting and compatibility semantics (for example Independent mode selection).
3. src/gameRenderer/pixi/
  - Pixi application/stage lifecycle, container composition, sprite creation/disposal, texture policy.
  - No host API calls here.
4. src/gameRenderer/adapters/
  - Boundary adapters only (asset URL/bytes adapter, scene-to-pixi mapping adapter).
5. src/gameRenderer/interaction/
  - Pointer/click/waypoint intent capture mapped to typed intents, not direct host mutation.
6. src/gameRenderer/diagnostics/
  - Renderer diagnostics events, frame timing markers, missing asset events.
7. src/gameRenderer/testing/
  - Shared fixtures/builders for renderer unit/integration tests.

### 2) Ownership Boundaries

1. App and hooks own runtime/session orchestration and host connectivity.
2. Scene layer owns functional mapping from runtime payloads to renderer-neutral state.
3. Pixi layer owns visual realization and lifecycle only.
4. Host API layer owns network/transport and retries.
5. No layer may take ownership of another layer's concerns.

### 3) Dependency Rules (Hard Gates)

1. src/gameRenderer/contracts has zero runtime dependencies beyond basic shared types.
2. src/gameRenderer/scene may depend on contracts and orchestration/host state types, but not on Pixi.
3. src/gameRenderer/pixi may depend on contracts/scene output, but not on hostApi or hooks.
4. src/hooks and src/hostApi must not import from src/gameRenderer/pixi internals.
5. Components outside renderer root may consume a single public renderer facade only.

### 4) Public Facade Rule

Expose one narrow renderer entrypoint, for example:

1. createGameRenderer(mountElement, options)
2. renderer.updateScene(sceneSnapshot)
3. renderer.resize(width, height)
4. renderer.dispose()

All other renderer modules stay internal to avoid cross-project coupling drift.

### 5) State and Update Model

1. Renderer input is immutable scene snapshots or explicit patch payloads.
2. Runtime ordering/watermark semantics are resolved before renderer ingestion.
3. Renderer applies updates serially and emits diagnostics on dropped/invalid updates.
4. No hidden side channels from Pixi internals back into host state.

### 6) Testing Matrix

1. Unit: scene transforms (anchor math, ordering, Independent mode compatibility rules).
2. Unit: pixi mapping (sprite props, transform sequence, z-index assignment, clipping).
3. Integration: host payload -> scene snapshot -> renderer update lifecycle.
4. Smoke: fixture room render snapshot for directional overlays.
5. Regression: preserve existing non-render behavior tests for commands/session flow.

### 7) Naming and File Conventions

1. Use feature-first names: DirectionalOverlaySceneBuilder.ts, PixiRoomOverlayLayer.ts.
2. Keep one primary responsibility per file.
3. Keep index.ts files as export barrels only.
4. Keep renderer docs near code: src/gameRenderer/README.md.

### 8) PR Review Checklist (Required)

1. Does this PR keep host/runtime ownership boundaries intact?
2. Does it avoid importing Pixi outside renderer modules?
3. Are scene semantics deterministic and test-covered?
4. Are lifecycle/disposal concerns covered to prevent leaks?
5. Are changes scoped to one migration phase/slice?
6. Is there any hidden WPF behavior replication instead of explicit Pixi-native logic?

## Baseline Functional Inventory (From WPF Simulator)

### Rooms and Scene Hydration

- User-visible behavior:
  - Active room metadata and scene payload hydrate at attach/start.
  - Render surface uses authored width/height when present, defaulting to 800x600.
- Source-of-truth data:
  - HostSessionDataEnvelope
  - HostRuntimePresentationResult
  - HostCommandNewRoomSummary
- Key simulator/runtime seams:
  - Storyboard.Simulator/ViewModels/SimulatorViewModel.cs
  - Storyboard.GameEngine/GameServices/Commands/GameCommandProcessorService.cs
  - Storyboard.Shared.Contracts/HostContracts/HostCommandDtos/HostSessionDataEnvelope_contract.cs
  - Storyboard.Shared.Contracts/HostContracts/HostCommandDtos/HostRuntimePresentationResult_contract.cs

### Directional Room Image Overlay

- User-visible behavior:
  - Directional overlays anchor to edges/corners with offsets.
  - Scale then rotate transform behavior.
  - Clipped to room bounds.
  - Ordered rendering from producer payload.
  - Independent display mode compatibility rule:
    - use default slot image when present; else first valid image.
- Source-of-truth data:
  - HostCommandNewRoomSummary.DirectionalRenderableImages
  - HostCommandRoomDirectionalImage
  - HostCommandRenderableImage
- Key simulator/runtime seams:
  - Storyboard.Simulator/ViewModels/SimulatorRenderablePreviewViewModels.cs
  - Storyboard.GameEngine/GameServices/Commands/GameCommandProcessorService.cs
  - Storyboard.Shared.Contracts/HostContracts/HostCommandDtos/HostCommandNewRoomSummary_contract.cs
  - Storyboard.Shared.Contracts/HostContracts/HostCommandDtos/HostCommandRoomDirectionalImage_contract.cs
  - Storyboard.Shared.Contracts/HostContracts/HostCommandDtos/HostCommandRenderableImage_contract.cs
- Existing behavior locks:
  - Storyboard.Simulator.Tests/SimulatorDirectionalImageAnchoringTests.cs
  - Storyboard.Simulator.Tests/SimulatorRoomImageTemplateTransformParityTests.cs

### Objects and Render Order

- User-visible behavior:
  - Objects render above room imagery.
  - Deterministic ordering from RenderZOrder semantics.
  - Selection cues align with object transforms.
- Key seams and locks:
  - Storyboard.Simulator/ViewModels/SimulatorRenderablePreviewViewModels.cs
  - Storyboard.GameEngine/GameServices/Commands/GameCommandProcessorService.cs
  - Storyboard.Simulator.Tests/SimulatorRoomObjectRenderOrderTests.cs
  - Storyboard.Simulator.Tests/SimulatorRenderablePlacementMathTests.cs

### Movement and Waypoint Navigation

- User-visible behavior:
  - Point intent and waypoint movement submit through host APIs.
  - Delta updates reconcile movement and room state.
  - Replay telemetry affects visualization timing.
- Key seams and locks:
  - Storyboard.Simulator/ViewModels/SimulatorViewModel.cs
  - Storyboard.Simulator/ViewModels/GameStateTreeProjectionBuilder.cs
  - Storyboard.Simulator.Tests/SimulatorReplaySpeedSemanticsTests.cs
  - Storyboard.Simulator.Tests/SimulatorScopeTreeQuantifiableProjectionTests.cs

### Commands and Clarification Chains

- User-visible behavior:
  - Command entry with ambiguity follow-up.
  - Correlation and raw command continuity across clarification.
- Key seams and locks:
  - Storyboard.Simulator/ViewModels/SimulatorViewModel.cs
  - Storyboard.GameEngine/GameServices/Commands/GameCommandProcessorService.cs
  - Storyboard.Simulator.Tests/SimulatorReplaySpeedSemanticsTests.cs

### Presentation Cues and Narrative

- User-visible behavior:
  - Ordered presentation steps (console/hud/archive behaviors).
  - Catalog-controlled effect behavior.
- Key seams and locks:
  - Storyboard.Simulator/Services/SimulatorPresentationEffectsCatalogService.cs
  - Storyboard.Simulator/ViewModels/SimulatorViewModel.cs
  - Storyboard.Simulator.Tests/SimulatorNarrativeCuePresentationTests.cs

### Sounds and Effects

- User-visible behavior:
  - Lane-based play/cancel/repeat/fade semantics.
- Key seams and gaps:
  - Storyboard.Simulator/ViewModels/SimulatorViewModel.cs
  - Storyboard.Shared.Contracts/HostContracts/HostCommandDtos/HostCommandSoundCue_contract.cs
  - Note: dedicated audio behavior tests are sparse compared with rendering tests.

### Identity, Discovery, Sessions, and Asset Cache

- User-visible behavior:
  - Thin/local mode switching, session attach/list/end, game discovery, preview assets.
  - Asset cache partitioning and warmup behavior.
- Key seams and locks:
  - Storyboard.Simulator/App.xaml.cs
  - Storyboard.Simulator/Services/SimulatorRuntimeConnectionSettings.cs
  - Storyboard.Simulator/Services/SimulatorDiscoveredAssetCacheService.cs
  - Storyboard.GameClient/Session/HostSessionAndDeltaHttpClient.cs
  - Storyboard.Simulator.Tests/SimulatorAuthenticationFlowTests.cs
  - Storyboard.Simulator.Tests/SimulatorSessionManagementStartFlowTests.cs
  - Storyboard.Simulator.Tests/SimulatorAssetCacheControlTests.cs
  - Storyboard.Simulator.Tests/SimulatorDiscoveryWarmupScopeDepthTests.cs

## Required Architecture Seams To Preserve

1. Keep host contracts as the renderer input boundary.
2. Keep command parsing/execution in runtime services.
3. Keep delta polling and watermark ordering semantics unchanged.
4. Keep renderer independent from direct file probing in thin mode.
5. Keep presentation effect interpretation owned by catalog/runtime policies.

## Risk Register

1. Medium: WPF-specific scene models (ImageSource, brush/visibility semantics) are not portable.
2. High: movement/waypoint and replay timing are stateful and easy to desync.
3. High: audio parity has lifecycle differences and limited current regression coverage.
4. Medium: asset caching policy differs between desktop and browser environments.
5. Medium: hidden compatibility behavior in directional display mode can be lost if not codified.

## Phased Execution Plan

### Phase 0: Baseline Lock and Migration Guardrails

Deliverables:

1. Capability matrix with source contracts and owning services.
2. Compatibility notes for special semantics (Independent directional mode, watermark ordering).
3. Test inventory map linking each capability to current lock tests.

Exit criteria:

1. Team sign-off on boundaries and non-goals.
2. No pending ambiguity on renderer-owned vs runtime-owned logic.

### Phase 1: Web Host Runtime Pipeline Parity (No Pixi Visuals Yet)

Deliverables:

1. Session attach/bootstrap flow in web host.
2. Delta poll loop with ordering and watermark safeguards.
3. Renderer-neutral scene state store fed by host DTOs.

Exit criteria:

1. Baseline room payload captured and observable in web state.
2. Delta updates apply in strict order with no stale overwrite behavior.

### Phase 2: Pixi 8 Foundation

Deliverables:

1. Pixi stage lifecycle wrapper (create, resize, dispose) using Pixi-idiomatic container ownership.
2. Asset loading adapter that consumes host-resolved URLs/bytes.
3. Diagnostics hooks for scene updates and asset failures.

Exit criteria:

1. Stable mount/unmount with no leaks in repeated navigation.
2. Stage sizing honors authored room dimensions and default fallback.

### Phase 3: First Feature Slice - Directional Room Image Overlay

Scope (strict):

1. Render directional room overlays only.
2. No objects, movement, waypoint visuals, HUD cues, or audio rendering yet.

Implementation targets:

1. Slot anchor mapping parity (Down + N/NE/E/SE/S/SW/W/NW).
2. Offset/scale/rotation parity.
3. Clipping to room bounds.
4. Stable producer-order rendering.
5. Independent display mode compatibility behavior.

Acceptance criteria:

1. Matches existing anchoring semantics from simulator tests.
2. Directional placement semantics are codified and test-locked: slot selects anchor, anchor point is positioned on room edge/corner, then offsets are applied in room space.
3. Missing asset handling is non-fatal and diagnostic.
4. Scene remains responsive under repeated room changes.

Test strategy:

1. Unit tests for anchor resolution and display-mode selection.
2. Unit tests for transform/parity semantics.
3. Integration test with fake baseline + delta payload feed.
4. Smoke snapshot for known room fixture with directional overlays.

### Phase 4: Room Objects and Static Interaction Cues

Deliverables:

1. Object sprite layer on top of directional overlays.
2. Deterministic z-order parity.
3. Selection outline cue parity for static states.

Exit criteria:

1. Object ordering and placement tests pass.
2. No regressions in directional overlay behavior.

### Phase 5: Movement, Waypoints, and Command-Driven Visual State

Deliverables:

1. Coordinate intent mapping for click/waypoint flow.
2. Movement visualization tied to host telemetry and delta reconciliation.
3. Command clarification continuity preserved end-to-end.

Exit criteria:

1. Replay timing semantics hold across speed multipliers.
2. Movement path visuals match host-acknowledged state changes.

### Phase 6: Presentation Cues and HUD Layer

Deliverables:

1. Catalog-routed cue rendering (HUD/console/archive targets).
2. Queueing/ordering parity for sequential cues.

Exit criteria:

1. Narrative cue tests and manual parity checks pass.
2. Unknown effect keys preserve current explicit behavior.

### Phase 7: Audio Semantics

Deliverables:

1. Web audio lane model for play/cancel/repeat/fade.
2. Explicit regression tests for lane interruption/overlap semantics.

Exit criteria:

1. Audio cues remain deterministic under replay and rapid delta batches.

### Phase 8: Hardening, Performance, and Retirement Path

Deliverables:

1. Perf and memory profiling for long-running sessions.
2. Golden-scene comparison checks for key fixtures.
3. WPF renderer coexistence and deprecation checklist.

Exit criteria:

1. Target parity reached for critical gameplay visualization paths.
2. Operational readiness documented for default web renderer usage.

## Design Lock-Off Questions

1. What is the single authoritative renderer input contract per frame: full scene snapshot only, or snapshot plus additive patch events?
2. Should the renderer reject out-of-order deltas hard (drop with diagnostics) or soft (queue and reconcile)?
3. Which layer owns watermark continuity enforcement before render updates are applied?
4. What exact fallback dimensions apply when authored render width/height are missing or invalid?
5. Which directional overlay anchors are mandatory for parity in Phase 3 (N, S, E, W, and corner variants)?
6. Is transform order contractually fixed to scale then rotate for all directional overlays?
7. For Independent room display mode, is selection precedence exactly default image first, then first valid image?
8. When a directional image asset fails to load, do we render nothing and emit diagnostics, or render a placeholder?
9. What is the canonical z-order rule between room overlays, room objects, and selection cues?
10. Do object selection cues belong to scene data, or are they derived in renderer interaction state?
11. Should pointer intent capture produce only typed intents, with zero direct host mutation from renderer modules?
12. What is the max allowed frame-time budget for update application before we emit performance diagnostics?
13. What is the disposal contract for scene transitions: full stage rebuild, or layer-level incremental teardown?
14. Which asset cache keys are canonical in browser mode to avoid duplicate texture loads across room transitions?
15. What are the explicit no-go dependencies from renderer modules (host API, hooks, persistence, config file probing)?
16. Which exact simulator tests are treated as parity locks for directional anchoring and transform semantics?
17. What are the acceptance thresholds for visual parity checks: pixel snapshot tolerance, semantic assertions, or both?
18. For IsNoOp delta results, should the renderer skip update application entirely and only advance internal diagnostics state?
19. What diagnostics events are mandatory for v1 (asset missing, dropped update, invalid ordering, render lifecycle faults)?
20. What objective criteria marks each phase as complete and eligible for merge without reopening prior phase scope?

## Design Lock-Off Decisions (2026-08-29)

1. Question 1 locked: Host baseline plus ordered delta polling is the authoritative update model. Renderer consumes derived immutable scene snapshots.
2. Question 2 locked: Keep continuity/recovery smarts in runtime and host orchestration. Client passes last watermark, consumes returned watermark, handles resync-required by fetching baseline.
3. Question 3 locked: Watermark continuity ownership is in host polling/orchestration, not renderer internals.
4. Question 4 locked: Missing or invalid authored dimensions is a contract error and must be logged as error. Do not silently render against guessed room bounds for runtime scene rendering.
5. Question 5 locked: Required directional room image set is Down plus all 8 directions (North, NorthEast, East, SouthEast, South, SouthWest, West, NorthWest). Down is foundational.
6. Question 6 locked: Directional placement uses a Pixi-native anchor model. The slot selects the sprite anchor point, position places that anchor on the room edge/corner, and offsets are applied in room space. Transform behavior must be explicit and parity-tested.
7. Question 7 deferred lane: Keep independent mode support as a tracked lane, but do not make full independent-mode parity an immediate implementation focus.
8. Question 8 locked: Missing directional asset renders as absent with diagnostics. Optional diagnostics-only location indicator may be enabled in debug mode.
9. Question 9 locked: Z ordering is backend-engine authoritative; renderer obeys provided ordering and reports invalid/missing ordering via diagnostics.
10. Question 10 locked: Engine owns cue semantics and timing intent; renderer owns visual realization details. Catalog mapping may be shared or renderer-specific, but semantic ownership stays in engine.
11. Question 11 deferred: Pointer-intent command routing work is deferred pending refactor to issue text commands through the unified command interface.
12. Question 12 locked: Monitor poll-ingest latency, frame/update latency, and backlog pressure separately. Thresholds are configuration-driven in WebPortal settings.
13. Question 13 locked: In-room updates stay incremental/lightweight; room transitions may perform deeper off-screen rebuild and swap with transition modes (atomic, slide, fade).
14. Question 14 locked: Cache identity key is canonical asset path. Same path across rooms must reuse the same cached resource. Use TTL/ETag metadata when available.
15. Question 15 locked: Renderer no-go dependencies include direct host API calls, hooks/workflow imports, persistence/auth ownership, command grammar/dispatch logic, and file probing in thin mode.
16. Question 16 locked: Simulator tests are parity guidance; WebPortal-specific tests are required and built incrementally as each slice lands.
17. Question 17 locked: Acceptance uses both semantic assertions and pixel snapshots, with semantic correctness as primary and snapshot tolerance as secondary smoke coverage.
18. Question 18 locked: IsNoOp is convenience/optimization, not protocol correctness dependency.
19. Question 19 locked: Mandatory diagnostics set remains required; no-op poll diagnostics are sampled and emitted every configurable heartbeat interval, not per poll.
20. Question 20 locked: Phase completion requires passing scope, boundary, contract, test, diagnostics, and validation gates.

### Directional Placement Contract (Pixi-Native)

1. Renderer-neutral scene data expresses directional placement intent (slot + offsets), not Pixi-specific coordinates.
2. Pixi layer resolves slot to anchor fractions and room-edge/corner position.
3. `sprite.anchor` identifies the point inside the image that is attached to the room coordinate.
4. `sprite.position` sets where that anchor point lands in room space.
5. Offsets are additive adjustments from the resolved room anchor coordinate.
6. Room space and screen/canvas space remain distinct; room coordinates are authoritative for placement semantics.
7. Missing directional assets render as absent and emit diagnostics.

### Visual Guardrail Priority: Room and Door Alignment

1. As soon as initial room render parity is achieved for room/wall imagery plus related door alignment, add a dedicated visual regression fixture immediately.
2. Treat this fixture as a high-priority hardening gate due to known historical instability and repeat regressions in this area.
3. The fixture must verify room base imagery and door overlays align correctly under the canonical authored dimensions and transform pipeline.
4. Include this fixture in routine renderer regression runs so drift is caught early during unrelated feature work.
5. If this guardrail fails, block phase progression until alignment parity is restored.

## Working Cadence Recommendation

1. Ship one phase per PR sequence where practical.
2. Keep behavior-additive changes separate from refactor-only changes.
3. Require explicit acceptance checklist pass before phase advancement.

## Initial Next Actions (Ready Now)

1. Create a renderer-neutral directional overlay scene DTO in web host code.
2. Implement Pixi stage wrapper and directional overlay layer only.
3. Port anchoring and transform parity tests first, then render implementation.
4. Run existing web test/build gates and add a focused directional overlay smoke fixture.

## Session Resume Handoff

Use this section to restart execution in a new session without re-discovery.

### Locked, Deferred, and Open Status

1. Locked decisions: Questions 1-5, 8-10, 12-20.
2. Deferred decisions: Question 6 (transform deep-dive), Question 7 (independent-mode full support lane), Question 11 (pointer-intent routing after command-interface refactor).
3. Open risk to watch continuously: room and door alignment drift (hard guardrail).

### Immediate Restart Slice (First PR Scope)

1. Scope: Phase 1 through minimal Phase 3 vertical slice for directional room overlays only.
2. Must include: renderer-neutral scene DTO, Pixi stage lifecycle wrapper, directional overlay layer, deterministic ordering, and diagnostics hooks.
3. Must exclude: object rendering, movement visuals, waypoint visuals, HUD cues, and audio rendering.

### First Validation Gate on Re-entry

1. Reconfirm boundaries before coding:
  - Renderer modules do not call host APIs directly.
  - Hooks/orchestration own transport, watermark, and resync behavior.
  - Renderer consumes immutable scene snapshots only.
2. Create the room-plus-door alignment visual regression fixture immediately after first parity render.
3. Treat that fixture as blocking for phase progression.

### Re-entry Command Checklist

1. Build solution: dotnet build .\\StoryboardDesigner.slnx
2. Run app test suite: dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj
3. Run runtime-focused regression filter: dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"

### Definition of Done for Next Session

1. A small end-to-end directional overlay path renders in Pixi from host-driven scene data.
2. New tests cover anchoring/transform semantics and include at least one overlay smoke fixture.
3. The high-priority room-and-door alignment guardrail fixture exists and is wired into routine regression runs.
4. No ownership boundary violations are introduced.

## Session Log

Use one entry per work session to keep restart context concise and verifiable.

### Entry Template

Date: YYYY-MM-DD
Owner: <name>
Scope: <phase/slice worked>

Changes Completed:
1. <what changed>
2. <what changed>

Decisions or Locks Updated:
1. <question or rule>

Tests and Validation Run:
1. <command>
2. <result summary>

Risks or Regressions Observed:
1. <risk or none>

Deferred Follow-ups:
1. <deferred item>

Next Session Start Here:
1. <first actionable step>

### Latest Entry

Date: 2026-08-29
Owner: Copilot + user lock-off
Scope: Governance lock-off hardening and resume readiness

Changes Completed:
1. Finalized Question 20 as locked with explicit phase completion gates.
2. Added high-priority visual guardrail requirements for room and door alignment.
3. Added Session Resume Handoff with deferred lanes, restart slice, and re-entry validation checklist.

Decisions or Locks Updated:
1. Question 20 locked: phase completion requires scope, boundary, contract, test, diagnostics, and validation gates.
2. Room and door alignment fixture is a blocking guardrail for phase progression.

Tests and Validation Run:
1. Plan/documentation update only in this step; no new build or test command executed.

Risks or Regressions Observed:
1. Historical drift risk remains concentrated in room imagery and door overlay alignment until fixture is implemented.

Deferred Follow-ups:
1. Question 6 transform deep-dive.
2. Question 7 independent-mode full-support lane.
3. Question 11 pointer-intent routing refactor dependency.

Next Session Start Here:
1. Implement the first directional overlay vertical slice with test-first anchoring and transform assertions.
2. Add the dedicated room-and-door alignment visual fixture immediately after first parity render.
