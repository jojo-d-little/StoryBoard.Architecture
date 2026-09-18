# Portrait Room Orientation Handoff Plan (Initial Rough Draft)

Last updated: 2026-09-09
Status: Draft (handoff-ready)

Purpose: capture an initial implementation path for supporting per-room portrait authoring/presentation while preserving current host/runtime boundaries and minimizing regression risk.

## Problem Statement

Today, authored room render dimensions are effectively global defaults (typically 800x600 landscape), which limits visual design choices for long/narrow spaces like hallways. We want some rooms to be authored and presented as portrait (for example 600x800), while still rendering into the same physical host viewport using contain scaling.

Expected outcome: portrait-authored rooms should appear narrower/taller in the same landscape viewport, reinforcing hallway depth illusion without changing physical monitor dimensions.

## Non-Goals (Phase 1)

1. No command grammar changes.
2. No runtime-side implicit directional alias behavior.
3. No host-side direct reads of runtime/export/project files in thin mode.
4. No broad renderer architecture refactor unrelated to per-room bounds.

## Current Baseline (Observed)

1. WebPortal renderer already scales and click-maps using scene bounds per snapshot.
2. WebPortal host adapter already hydrates scene bounds from authored width/height.
3. Designer room tabs currently receive global project canvas width/height.
4. Project model currently owns global room canvas width/height defaults.
5. Room model does not currently expose room-level render width/height or orientation.
6. Runtime/host contracts currently surface authored render width/height at project/session level.

## Proposed Shape

Use explicit per-room render dimensions as source of truth, with orientation derived from aspect ratio and exposed in UX as a preset convenience.

1. Data truth:
- Add room-level render width/height fields (effective bounds for that room).
- Preserve project-level width/height as defaults for newly created rooms.
- Treat width/height as the only authoritative room geometry values in contracts and runtime.
- Do not persist a room orientation enum in runtime contracts for Phase 1.

2. UX convenience:
- Add a room setting for orientation preset:
  - Landscape preset -> width 800, height 600 (or current project defaults)
  - Portrait preset -> width 600, height 800 (or swapped project defaults)
- Keep width/height directly editable for advanced layouts.
- Orientation label/state is derived from current width/height, not separately persisted.

3. Runtime/host output:
- Include room-effective render bounds in room transition payloads (new-room summary), so host rendering uses room-specific truth directly.

4. Cell size policy (Phase 1):
- Keep cell size project-global (no per-room cell size overrides).
- Remove unused room-level cell size deviation support from schema/contracts during this update.

## Why This Shape

1. It avoids fragile host heuristics.
2. It keeps producer ownership explicit.
3. It remains extensible for non-standard room sizes later.
4. It aligns with current bounds-driven WebPortal scaling and pointer mapping logic.

## Initial Scope Breakdown

### A) Designer Authoring + UX

1. Add room-level render width/height (orientation preset is UX-only and derived from dimensions).
2. Room settings dialog/editor updates for per-room bounds.
3. Room editor tab initialization uses selected room effective bounds.
4. Migration/defaulting logic:
- Existing rooms inherit current project defaults when fields are absent.
5. Validation:
- Ensure width/height > 0.
- Ensure width/height are evenly divisible by project cell size.
- Keep room cell counts within agreed min/max limits (for example 4-200 columns/rows).
- Guard nonsensical values with friendly diagnostics.

Estimated effort: Medium.

### B) Runtime + Host Contracts

1. Contract update to carry room-effective bounds in room change/new-room payload.
2. Contract cleanup: remove unused room-level cell size deviation fields/paths from schema/contracts.
3. Schema + codegen regeneration and guardrail updates.
4. Runtime bootstrap and session projection wiring:
- Room-level bounds flow from authored room to runtime snapshot.
- GameManager session data envelope and presentation baseline include room-effective bounds for active room.
5. Backward compatibility:
- For older projects with missing room bounds, apply deterministic load-time defaulting from project-level defaults, then persist explicit room bounds.

Estimated effort: Medium to Medium-High.

### C) WebPortal Renderer

1. Adapter consumes room-effective bounds from payload when present.
2. Keep contain scaling and click mapping bounds-driven.
3. Treat click/tap -> game engine normalized point translation as a Stage 4 correctness gate.
4. Ensure normalized point translation always uses active-room effective bounds (not stale or project-default bounds) after every room transition.
5. MVP transition decision: keep current directional slide transition behavior even across mixed aspect ratios.
6. Accept visible letterbox/background exposure during mixed-aspect slide transitions as MVP behavior.
7. Verify directional overlay slot placement across mixed aspect-ratio transitions.
8. Add focused tests for mixed landscape <-> portrait room transitions.
9. Add focused tests that validate click/tap mapping to expected normalized room-space coordinates across mixed room sizes/aspect ratios.

Estimated effort: Low to Medium.

### D) Simulator Host (MVP)

1. Consume active-room effective width/height from runtime payload/session state as display bounds source of truth.
2. Render room content using basic contain/center behavior within simulator viewport.
3. Accept visible empty space (letterbox/pillarbox) when room aspect ratio differs from host viewport.
4. Keep transition behavior simple for MVP (no advanced mixed-aspect choreography requirement).
5. Ensure any simulator interaction mapping continues to use active-room bounds correctly.

Estimated effort: Low to Medium.

## Structured Delivery Order And Session Handoffs (Locked)

Implementation order is fixed for this workstream:

1. Contract schema adjustments
2. Designer support
3. GameEngine support
4. WebPortal support
5. Simulator support

Each stage must end with a formal handoff markdown document before downstream work begins.

Current progress:
- Stage 1 (Contract schema adjustments): Complete.
- Stage 2 (Designer support): Complete.
- Stage 3 (GameEngine support): Complete.
- Stage 4 (WebPortal support): Complete.
- Stage 5 (Simulator support): Complete.
- Workstream status: Stage 1-5 MVP complete; handoff package finalized.

### Stage 1: Contract Schema Adjustments

Goal:
- Apply schema/contract changes for room-effective dimensions and remove room-level cell-size deviation support.

Primary output handoff document:
- plans/active/handovers/PORTRAIT_ROOM_ORIENTATION_01_CONTRACT_HANDOFF.md

Required handoff contents:
- Contract/schema files changed.
- Exact fields added/removed/renamed.
- Codegen artifacts regenerated and accepted.
- Guardrail test results (pass/fail and command list).
- Consumer impact notes for Designer, GameEngine, WebPortal, and Simulator.

Downstream usage:
- Stage 2 (Designer) starts from this handoff.
- Stage 3 (GameEngine) also starts from this same handoff.

### Stage 2: Designer Support

Goal:
- Add authoring UX and validation behavior for room-level dimensions and defaults.

Primary output handoff document:
- plans/active/handovers/PORTRAIT_ROOM_ORIENTATION_02_DESIGNER_HANDOFF.md

Required handoff contents:
- UX changes (dialogs, labels, presets, validation messaging).
- Save/validation behavior confirmation (save allowed, run/export blocked on errors).
- Legacy load/default/write-back behavior confirmation.
- Tests added/updated and results.
- Any known UX caveats for runtime hosts.

Downstream usage:
- Stage 3 may reference this for authoring/runtime alignment checks.

### Stage 3: GameEngine Support

Goal:
- Ensure runtime session projection, bounds logic, and transition payloads carry room-effective dimensions correctly.

Primary output handoff document:
- plans/active/handovers/PORTRAIT_ROOM_ORIENTATION_03_GAMEENGINE_HANDOFF.md

Required handoff contents:
- Runtime mapping and payload path changes.
- Bounds/movement correctness verification summary.
- Backward compatibility behavior for legacy content.
- Tests added/updated and results.
- Host-consumer notes for WebPortal and Simulator.

Downstream usage:
- Stage 4 (WebPortal) starts from Contract + GameEngine handoffs.
- Stage 5 (Simulator) starts from Contract + GameEngine handoffs.

### Stage 4: WebPortal Support

Goal:
- Consume room-effective bounds and keep MVP slide behavior for mixed-aspect transitions.
- Preserve correct click/tap -> normalized room-space translation for host/game engine intents across mixed-aspect transitions.

Primary output handoff document:
- plans/active/handovers/PORTRAIT_ROOM_ORIENTATION_04_WEBPORTAL_HANDOFF.md

Required handoff contents:
- Adapter/renderer changes for bounds consumption.
- Mixed-aspect transition behavior confirmation (letterbox/background expected).
- Interaction/click mapping verification.
- Normalized point translation verification (viewport click/tap to room-space x/y) across mixed room dimensions.
- Test matrix coverage and results.
- Known limitations deferred past MVP.

Downstream usage:
- Stage 5 may reference this for behavior parity notes.

### Stage 5: Simulator Support

Goal:
- Consume room-effective bounds with simple MVP presentation behavior and correct interaction bounds.

Primary output handoff document:
- plans/active/handovers/PORTRAIT_ROOM_ORIENTATION_05_SIMULATOR_HANDOFF.md

Required handoff contents:
- Bounds-driven presentation behavior confirmation.
- Mixed-aspect letterbox/pillarbox acceptance confirmation.
- Interaction and movement bounds verification.
- Tests added/updated and results.
- Final MVP completion notes and residual risks.

### Handoff Document Template (Apply To Every Stage)

1. Scope completed
2. Files changed
3. Contract/interface impact
4. Validation commands executed
5. Test results
6. Behavioral notes (including accepted MVP limitations)
7. Known issues/risks
8. Explicit next-stage start checklist

## Risk Register

1. Contract drift risk:
- Multiple schema/DTO touchpoints can fall out of sync.
- Mitigation: run schema/codegen/transport guardrail suites early.

2. Transition regression risk:
- Room-change tween math may reveal edge cases when dimensions change significantly.
- Mitigation: add tests for transition vectors and staged-surface swaps across aspect changes.

3. Designer migration risk:
- Old projects without room-level fields may get inconsistent defaults.
- Mitigation: strict load-time defaulting policy and migration tests.

4. Bounds-sensitive gameplay checks:
- Out-of-bounds movement logic references session/current-room dimensions.
- Mitigation: ensure runtime room node carries effective bounds cleanly.

## Initial Validation Gates (Draft)

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
3. dotnet test .\Storyboard.TransportCodegen.Tests\Storyboard.TransportCodegen.Tests.csproj
4. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "SchemaCodegenHardcodedDtoGuardrailsTests|SchemaEmittedContractDriftGuardrailsTests|ArchitectureSeparationGuardrailsTests|TransportArtifactGuardrailsTests"
5. WebPortal focused tests for bounds/orientation transitions (new tests to be added during implementation)
6. WebPortal focused tests for click/tap -> normalized room-space mapping under mixed room dimensions (new tests to be added during implementation)
7. Simulator focused tests for room-bounds presentation across mixed aspect ratios (new tests to be added during implementation)

## MVP Acceptance Criteria

1. Rooms authored at 800x600 and 600x800 both render correctly in WebPortal and Simulator using room-effective bounds.
2. Mixed-aspect transitions keep existing directional slide behavior in WebPortal.
3. Visible letterbox/background during mixed-aspect slide transitions is accepted and not treated as a bug for MVP.
4. Simulator may show letterbox/pillarbox under mixed aspect ratios; this is accepted MVP behavior if room layout bounds are correct.
5. Interaction and movement bounds remain correct for active-room dimensions across room changes.
6. WebPortal click/tap translation to normalized room-space coordinates remains correct for active-room dimensions across room changes (including mixed-aspect transitions).

## New Project Validation Rules To Add

1. Width and height must be positive integers.
2. Width and height must be divisible by project cell size.
3. Derived cell counts (width/cellSize, height/cellSize) must be whole numbers.
4. Derived cell counts must be within configured min/max row and column limits.
5. Project validation should emit actionable diagnostics naming the room and offending value(s).

## Design Lock Questions (Record Decisions Here)

Use this section as the formal decision log before implementation starts. Replace each TBD with final approved values.

1. Validation limits (required)
- Question: What are the default min/max column and row limits?
- Decision: Default room grid limits are min 4 and max 200 for both columns and rows.
- Notes: Suggested starter range is 4-200 for both columns and rows.

2. Validation severity and save behavior (required)
- Question: Are invalid room dimensions blocking errors at save/export time, or warnings only?
- Decision: Room dimension/grid violations do not block designer save, but they are blocking validation errors for export and game run/engine execution until corrected.
- Notes: Recommendation is blocking error for divisibility and min/max violations.

3. Legacy migration persistence policy (required)
- Question: After defaulting missing room bounds from project defaults, do we persist explicit room bounds on next save?
- Decision: Use explicit per-room dimensions as durable project data. Legacy rooms missing explicit bounds resolve from project defaults at load and are written back explicitly on next save.
- Notes: Recommendation is yes, persist to eliminate long-term fallback logic.

4. Project-level defaults lifecycle (required)
- Question: Do project-level width/height remain user-editable defaults indefinitely in MVP?
- Decision: Project-level room width/height remain user-editable in MVP as creation defaults only; existing rooms use explicit room dimensions once set.
- Notes: Recommendation is yes for MVP; revisit later only if UX confusion emerges.

5. Contract cleanup scope for room cell-size overrides (required)
- Question: Which specific room-level cell-size deviation fields/paths are removed in this change?
- Decision: Room-level cell-size deviation support is removed from schema/contracts and runtime mappings; project-level cell size is the only supported cell-size source in MVP, protected by contract guardrail coverage.
- Notes: Must be explicitly enumerated in schema/contract PR notes to avoid ambiguity.

6. Mixed-aspect transition acceptance wording (required)
- Question: What exact acceptance statement should QA use for visible letterbox/background during mixed-aspect slide?
- Decision: For MVP, mixed-aspect transitions in WebPortal keep the existing directional slide. Visible letterbox/background exposure during transition is expected behavior and not a defect, provided directional motion, final room alignment, and interaction bounds correctness are preserved.
- Notes: Should explicitly state this is expected MVP behavior, not a regression.

7. Simulator interaction behavior under mixed aspect (required if simulator supports click interaction)
- Question: Which simulator interactions must be validated against active-room bounds (for example click-to-cell, hit tests, overlays)?
- Decision: Simulator mixed-aspect validation covers in-room object placement, movement bounds, overlay anchoring, and any enabled click/hit-test mapping against active-room dimensions across room transitions.
- Notes: Enumerate exact interaction list for focused regression tests.

8. Test matrix minimum set (required)
- Question: What is the minimum mandatory room-size transition test matrix for MVP?
- Decision: MVP transition tests include 800x600 -> 600x800, 600x800 -> 800x600, 800x600 -> 800x600, 600x800 -> 600x800, 1600x1200 -> 1200x1600, 320x240 -> 240x320, and 800x600 -> 320x240.
- Notes: Includes cross-orientation baselines, same-orientation baselines, oversized and undersized non-standard pairs, and explicit large-to-small downscale. For WebPortal Stage 4, this same matrix also applies to click/tap -> normalized room-space mapping assertions.

9. Non-standard size policy communication (required)
- Question: Do we explicitly document that arbitrary sizes are supported only when divisible by project cell size and within limits?
- Decision: Non-standard room sizes are supported in MVP only when width and height are divisible by project cell size and resulting row/column counts are within configured limits; this rule is documented in validation messages and authoring documentation.
- Notes: Recommendation is yes, include in user-facing validation messages and docs.

10. Future phase boundary (required)
- Question: Is per-room cell-size override explicitly deferred out of MVP and tracked as Phase 2 exploration?
- Decision: Per-room cell-size override is out of MVP scope and explicitly deferred to a Phase 2 exploration item; MVP supports only project-global cell size.
- Notes: Recommendation is yes, to prevent scope creep during MVP implementation.

## Suggested First Fresh-Session Agenda

1. Execute Stage 1 only (Contract Schema Adjustments).
2. Produce and review Stage 1 handoff document.
3. Start Stage 2 only after Stage 1 handoff approval.

## Handoff Notes

1. This is intentionally rough and should be refined into phased tasks with explicit file-level checkpoints.
2. The highest-value early de-risk is contract + runtime wiring before deeper UI work.
3. WebPortal appears structurally ready for room-specific bounds, but mixed-aspect transition tests are mandatory.

## Open Questions For Next Session

1. None currently; design-lock decisions and staged handoff workflow are recorded.
