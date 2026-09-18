# Skeleton-Declared Splitter Support Plan (WebPortal)

Status: Future (deferred)
Owner: Storyboard.WebPortal
Last updated: 2026-08-28

## Goal

Provide user-resizable splitter support in config-driven layout preview while keeping layout authors (game producers) in control through skeleton HTML/CSS authoring.

## Why This Plan Exists

Current layout controls support:
1. Explicit grid sizing and growth behavior.
2. Holistic empty-slot collapse behavior.
3. Strong producer control in skeleton CSS.

Current gap:
1. No user drag-resize for selected slot boundaries.
2. Any future resize variable contract can become hidden "magic" if only expressed in TypeScript.

Producer requirement:
1. Keep splitter declarations and variable intent visible in existing skeleton files.
2. Avoid introducing additional configuration files solely for splitter metadata.

## Non-Goals

1. Persist splitter values across sessions.
2. Introduce runtime/game-host coupling for layout control.
3. Replace existing collapse behavior.
4. Add a broad dynamic layout DSL.

## Proposed Direction

Use skeleton-declared splitter markers in template HTML and let preview runtime implement drag behavior generically.

### Authoring Pattern (Skeleton)

Preferred custom element format (must include a hyphen):

```html
<resize-splitter
  data-variable="--left-rail-width"
  data-axis="x"
  data-min="200"
  data-max="700"
  data-target=".layout-main"
  data-track="left"
></resize-splitter>
```

Notes:
1. `resize-splitter` is metadata for preview runtime.
2. `data-variable` names the CSS variable to update during drag.
3. `data-axis` indicates horizontal (`x`) or vertical (`y`) resize.
4. `data-min` and `data-max` define clamping bounds (pixels).
5. `data-target` points to the container whose sizing is controlled.
6. `data-track` indicates which side/track is directly adjusted.

### CSS Variable Consumption (Skeleton CSS)

Variable default + use in track sizing:

```css
.config-preview.desktop-standard {
  --left-rail-width: 320px;
}

.config-preview.desktop-standard .layout-main {
  grid-template-columns: minmax(200px, var(--left-rail-width)) minmax(825px, 1fr);
}
```

## Runtime Responsibilities

Implement in `Storyboard.WebPortal/src/components/ConfigDrivenLayoutPreview.tsx`:
1. Read declared splitter metadata from hydrated template metadata.
2. Render splitter handle(s) in preview with correct axis cursor and hit area.
3. Handle pointer drag lifecycle.
4. Compute clamped values.
5. Apply value via `previewRootElement.style.setProperty(variableName, pxValue)`.

Loader support in `Storyboard.WebPortal/src/orchestration/loader.ts`:
1. Parse splitter tags from template HTML.
2. Store extracted splitter definitions alongside existing template metadata.

## Guardrails

1. Only allow CSS variables with approved prefix (for example `--layout-` or `--slot-`).
2. Ignore malformed splitter declarations and emit diagnostics warnings.
3. Clamp all drag values to parsed min/max with hardcoded safe fallback bounds.
4. Keep behavior session-only by default (no persistence).
5. Preserve existing empty/collapse rules as higher-priority layout behavior.

## Complexity Assessment

1. Compared with a hardcoded single splitter:
- More flexible, slightly higher implementation complexity.

2. Compared with adding a new config file:
- Lower producer cognitive load (all in skeleton), no extra config asset.

3. Overall:
- Medium complexity, supportable if attribute contract remains small and strict.

## Incremental Delivery Plan

### Phase 0: Contract Lock (No Runtime Changes)

1. Lock supported splitter attributes and naming convention.
2. Add concise contributor note in orchestration docs.

Acceptance:
1. Team agrees on stable attribute set.

### Phase 1: Single Splitter Pilot (Desktop Standard)

1. Add one `resize-splitter` marker between left rail and playfield.
2. Wire one variable (`--left-rail-width`) with pixel clamping.
3. Add splitter handle styling in shared CSS.

Acceptance:
1. Dragging resizes left rail in config mode.
2. Existing slot collapse behavior remains correct.

### Phase 2: Generic Splitter Extraction

1. Generalize rendering/drag logic for multiple declared splitters.
2. Support both horizontal and vertical splitters.

Acceptance:
1. Multiple splitters work without additional TypeScript hardcoding per splitter.

### Phase 3: DevTools Discoverability

1. Add read-only list of active splitter declarations in DevTools.
2. Show variable names, axis, min/max, target, and current computed value.

Acceptance:
1. Producers can inspect what splitters are active and what variables they control.

## Risks and Mitigations

1. Risk: Hidden magic returns via weak conventions.
- Mitigation: strict attribute contract + diagnostics + prefix guard.

2. Risk: CSS selector drift between skeleton and runtime DOM.
- Mitigation: continue class-aligned rendering strategy; avoid runtime-only selector dependencies.

3. Risk: Interaction conflicts with drag handles in overlays/devtools.
- Mitigation: scoped z-index and pointer target zones; test with nonmodal panel in docked/undocked modes.

## Validation Plan (When Implemented)

1. `npm run test`
2. `npm run build`
3. Manual checks:
- SignedOut and SignedIn desktop standard layouts.
- Utility panel omitted state collapse + resize behavior.
- Modal hidden/visible interaction with splitter handles.
- Nonmodal devtools docked and undocked behavior.

## Deferred Decision

Persistence is intentionally deferred.
1. If later needed, evaluate session-only persistence first.
2. Do not write to project content by default.