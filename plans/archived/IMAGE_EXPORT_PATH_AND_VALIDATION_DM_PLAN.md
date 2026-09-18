# Image Export Path And Validation DM Plan

Status: Completed (Implemented and validated)
Owner: StoryboardDesigner.App image authoring + clean export pipeline
Last updated: 2026-07-11 (Phase 1-5 complete)

## 1. Purpose

Define and implement a lightweight, maintainable cross-cutting solution for image export and image-path correction guidance that covers:

1. Room images.
2. Object images.
3. Clean export asset consolidation.
4. Source-path health validation with tree-node navigation support.

## 2. Confirmed Direction

1. Keep authoring linked to source image paths.
2. Export clean/runtime payloads to consolidated project-local assets.
3. Keep validation correction lightweight:
- Use validation issue path + Go To Node.
- Do not add per-rule custom correction UIs.
4. Correction happens in existing dialogs for the selected node.

## 3. Current State (As Of 2026-07-11)

Implemented:

1. Object image fields and appearance dialog flow.
2. Object image persistence + clean export DTO fields.
3. Validation report Go To Node navigation callback and UI action.
4. Clean export image materialization to GameExportedJson/assets/images/_shared with sha256-based dedupe.
5. Clean export assets-manifest.json emission and relative image path rewriting.
6. Snapshot baseline refreshed for clean-export contract changes.
7. Shared designer image fallback resolver (source -> project-source-images -> export assets) wired into room/object preview surfaces.
8. Initial image availability validation rule coverage with severity matrix for source/export presence states.
9. Unified Source Image Management Tools workflow with mandatory preview-before-apply, mode selection, optional repoint, and diagnostics file output.
10. Ambiguity-aware image validation messaging for project-source-images filename collisions with explicit Tools workflow guidance.
11. Final fallback-status and validation-hint polish in room/object dialogs via a shared status formatter and clearer user-facing status text.

Not implemented yet:

1. None.

## 4. Target Clean Export Assets Structure

For project file <ProjectName>.sbe.json:

1. GameExportedJson/<ProjectName>.sbe.clean.json
2. GameExportedJson/<ProjectName>.sbe.clean.navigation.json
3. GameExportedJson/<ProjectName>.sbe.clean.rooms/<roomId>.clean.room.json
4. GameExportedJson/assets/images/_shared/<sha256>__<sanitized-original-stem><canonical-ext>
5. GameExportedJson/assets/assets-manifest.json

Notes:

1. Paths written into clean JSON are always relative under assets/.
2. Runtime image paths point directly to assets/images/_shared binaries.
3. Shared image filename uses full sha256 plus original filename stem for human visibility.
4. Canonical extension is detected from actual image format.

## 4.1 Project Source Image Library Structure (Designer Authoring)

Project source image root:

1. project-source-images

Top-level scope buckets:

1. rooms
2. room-templates
3. objects
4. object-templates
5. base-objects

Per-item folders:

1. <technical-stable-name>--<id>

File names:

1. Default to original source filename.
2. On conflict append short hash.

## 5. Phased Execution

### Phase 1: Export Asset Materialization

1. Add a small export-image staging helper in StoryboardDesigner.App.
2. Collect image references from room images and object images.
3. Copy files to clean export assets folders.
4. Rewrite clean DTO image paths to relative exported asset paths.
5. Emit assets-manifest.json with source path, exported path, hash, and references.

Acceptance criteria:

1. Clean export folder contains assets for referenced images.
2. Clean room/object image paths point to relative exported locations.
3. Duplicate source content is deduped by hash.

### Phase 2: Source Missing Fallback

1. Add shared image resolution utility for designer image loading.
2. Resolution order:
- Source path from project JSON.
- project-source-images local match.
- Exported asset path.
3. Keep source path unchanged unless user explicitly relinks/adopts.

Acceptance criteria:

1. Missing source can still render if exported asset exists.
2. UI can indicate when fallback is used.

### Phase 3: Image Validation Rules

1. Add image-path validation rules in Validation/Rules.
2. Rule coverage:
- Source missing + export exists (warning).
- Source missing + export missing (error).
- Export missing while source exists (warning).
3. Rule path values must resolve to the correct tree node.

Acceptance criteria:

1. Validation report lists image issues with actionable path targeting.
2. Go To Node lands user on the correct room/object node.

### Phase 4: Existing Dialog UX Updates (No New Custom Workflow)

1. Room image dialog adds clear path-health indicator and relink action.
2. Object appearance dialog adds same path-health indicator and relink action.
3. Optional action: adopt exported copy as source path.

### Phase 5: Source Image Management (Unified Action)

1. One Tools menu command: Source Image Management.
2. One dialog with two explicit modes:
- Repair Missing Sources
- Consolidate Source Images
3. Mandatory preview pass, then explicit apply confirmation.
4. Consolidation always copies to project-source-images (never modifies originals).
5. Repoint source paths is user-selectable (default ON per locked decisions).
6. Diagnostic outputs:
- append log file with run headers
- per-run preview/apply text files

Acceptance criteria:

1. Users can fix path issues from existing dialogs after Go To Node.
2. No specialized validation-correction dialog is required.

## 6. Test Plan

1. JsonExportServiceProjectStateTests:
- Roundtrip includes object image fields.

2. JsonExportServiceCleanExportTests:
- Export creates expected assets tree and manifest.
- Exported image paths are relative and deterministic.

3. New focused tests:
- Image dedupe by content hash.
- Fallback resolution order and behavior.
- Validation image issue path maps to expected tree node.
- Source Image Management preview/apply output files and append-log content.
- Ambiguity-safe auto-correction behavior (no change when uncertain).

4. Validation defaults:
- dotnet build .\StoryboardDesigner.slnx
- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"

## 7. Risks And Guardrails

1. Risk: accidental absolute paths in clean runtime contract.
- Guardrail: assert relative assets/ path format in tests.

2. Risk: filename collisions in large projects.
- Guardrail: include hash in output filename and use id-scoped folders.

3. Risk: user confusion when fallback is active.
- Guardrail: show explicit fallback indicator in existing dialogs.

## 8. Completion Definition

This DM plan is complete when:

1. Clean export image files are materialized and referenced via relative paths.
2. Missing source handling degrades gracefully via export fallback.
3. Validation reports image-path issues and Go To Node gets user to the fix location.
4. Existing room/object image dialogs can correct bad source paths without introducing custom validation-fix workflows.

## 9. Design Lock-Off Worksheet (With Recommended Decisions)

Lock-off walkthrough completed. Decisions below are superseded by section 10 baseline.

### 9.1 Decision Matrix

1. Question: Should clean export always write image paths as assets-relative values?
- Recommended decision: Yes.
- Rationale: Keeps runtime portable and machine-agnostic.
- Status: Proposed.

2. Question: If an image source is missing and no fallback exists at export time, what should clean JSON contain?
- Recommended decision: Emit empty/null image path for that channel and report a validation/export warning.
- Rationale: Avoids shipping dead absolute paths as if they were valid runtime assets.
- Status: Proposed.

3. Question: Confirm clean export asset root location.
- Recommended decision: Single root at GameExportedJson/assets.
- Rationale: Predictable structure and simpler discovery.
- Status: Proposed.

4. Question: Should area export use the same naming/materialization policy?
- Recommended decision: Yes, same policy and helper logic, even if output roots differ.
- Rationale: Reduces divergence and maintenance risk.
- Status: Proposed.

5. Question: Confirm folder topology for copied images.
- Recommended decision:
	- assets/rooms/<roomName>--<roomId>/<slot>/<channel>__<hash8>.<ext>
	- assets/objects/<objectName>--<objectId>/<channel>__<hash8>.<ext>
- Rationale: Human-readable ownership plus stable uniqueness.
- Status: Proposed.

6. Question: Which hash algorithm should define asset identity?
- Recommended decision: SHA-256; use first 8 chars for filename suffix.
- Rationale: Strong collision resistance with short readable names.
- Status: Proposed.

7. Question: Should dedupe be global across rooms/objects or local per folder?
- Recommended decision: Global dedupe by content hash, while preserving logical owner folders in manifest references.
- Rationale: Saves space and avoids duplicate binaries.
- Status: Proposed.

8. Question: Should source path be auto-rewritten when fallback is used?
- Recommended decision: No automatic rewrite.
- Rationale: Preserves authoring intent and avoids surprising mutation.
- Status: Proposed.

9. Question: Should fallback lookup order be source first, exported second?
- Recommended decision: Yes.
- Rationale: Source remains primary authoring truth; exported copy is resilience path.
- Status: Proposed.

10. Question: Should fallback be enabled in both room image and object image dialogs in this feature slice?
- Recommended decision: Yes.
- Rationale: Consistent behavior and reduced user confusion.
- Status: Proposed.

11. Question: Should assets-manifest.json be emitted, and what is its role?
- Recommended decision: Emit assets-manifest.json as a diagnostics/audit artifact, not a runtime-required contract.
- Rationale: Supports traceability without tightening external contract coupling.
- Status: Proposed.

12. Question: Validation severity for image health states?
- Recommended decision:
	- Source missing + export fallback exists -> Warning.
	- Source missing + export fallback missing -> Error.
	- Source exists + export asset missing -> Warning.
- Rationale: Prioritize broken runtime outcomes as blocking while allowing recoverable states.
- Status: Proposed.

13. Question: What level of path granularity should validation issues use?
- Recommended decision: Path resolves to owning tree node; issue description/hint carries slot/channel detail.
- Rationale: Works with current Go To Node model without path parser complexity.
- Status: Proposed.

14. Question: Should custom validation-fix dialogs be added?
- Recommended decision: No.
- Rationale: Existing dialogs are sufficient when combined with Go To Node and path-health indicators.
- Status: Proposed.

15. Question: Export failure policy if one image copy fails.
- Recommended decision: Continue export, record warning(s), and omit unresolved channel path from clean JSON.
- Rationale: Prevents all-or-nothing failures while surfacing actionable issues.
- Status: Proposed.

16. Question: Should export-relative path memory be persisted in core project model now?
- Recommended decision: No new persistent authoring fields in this slice; use manifest/runtime export outputs first.
- Rationale: Keeps phase-1 additive and low risk; revisit after behavior proves stable.
- Status: Proposed.

17. Question: Should this roll out behind a feature toggle?
- Recommended decision: No toggle, ship always-on with tests and clear release notes.
- Rationale: Simpler support surface and fewer split-code paths.
- Status: Proposed.

### 9.2 Sign-Off Checklist

Lock this plan for implementation when all items below are explicitly agreed:

1. Path policy locked (relative assets-only in clean export).
2. Folder structure and naming/hash policy locked.
3. Dedupe scope locked.
4. Fallback + source mutation policy locked.
5. Validation severity matrix locked.
6. Export partial-failure behavior locked.
7. Manifest role locked.

## 10. Locked Decisions (Implementation Baseline)

This section is the authoritative implementation baseline from the lock-off walkthrough.

1. Clean export writes image paths as assets-relative values only.
2. If unresolved at export, write empty string for that channel and report issue.
3. Clean export assets root is GameExportedJson/assets with images under assets/images.
4. Export dedupe is global by content hash.
5. Shared export image naming:
- <sha256>__<sanitized-original-stem><canonical-ext>
6. Canonical extension is detected by image format.
7. Runtime clean JSON points directly to assets/images/_shared files.
8. Source path is never auto-rewritten to export path.
9. Fallback chain order:
- JSON source path
- project-source-images local match
- export assets path
10. Fallback applies to both room and object image surfaces.
11. Ambiguous local match behavior:
- do not auto-correct
- raise validation for manual correction
12. Unambiguous local match criterion:
- exactly one candidate after scope-bucket + filename filtering
13. Project source image root folder:
- project-source-images
14. Project source organization:
- scope buckets: rooms, room-templates, objects, object-templates, base-objects
- per-item folder: <technical-name>--<id>
15. Consolidation behavior:
- always copy, never modify original external file
- repoint source path user-selectable and default ON
16. Unified management UX:
- single Tools menu action
- single dialog with explicit Repair/Consolidate modes
- mandatory preview before apply
17. Scope filtering in management run:
- none (always all scopes)
18. Already-local references:
- skip by default
19. Local conflict replacement:
- optional toggle, default OFF
20. Validation report behavior:
- Go To Node only, no auto-open editor
- issue text remains diagnostic (no extra guided hint text)
21. Validation severities:
- resolvable via fallback chain = warning
- ambiguous withheld = warning
- unresolvable = error
22. Diagnostics outputs:
- simple append log with timestamped run headers
- per-run text files:
	- source-image-run-<ts>-preview.txt
	- source-image-run-<ts>-apply.txt
- preview file is written even if apply is canceled
- no auto-prune of historical run files
23. Pipeline design goal:
- channel-agnostic image reference pipeline for future image types
24. Implementation sequence is locked (low-risk order):
- generic reference abstraction
- clean export copy + relative path rewrite
- unified management dialog + preview/apply
- project-source copy/repoint logic
- fallback resolution chain
- validation rules and path coverage
- diagnostics polish
