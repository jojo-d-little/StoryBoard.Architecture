# Phase 1-3 Execution Plan

Last updated: 2026-06-22

## Goal
Execute phases 1, 2, and 3 efficiently with small, reviewable increments, then perform a formal review before phase 4.

This plan intentionally focuses on:
1. Native-file debt cleanup and data separation.
2. Standard export v1 implementation.
3. Validation/adoption readiness.

## Canonical Terms
1. Native user project data: authored game/content data.
2. Native application state data: tool/session/workspace state.
3. Clean exported format: external collaboration contract derived from native user project data.

## Working Rules
1. Small batches: each batch should be independently testable and reversible.
2. Write-path first: stop writing debt fields before removing read fallbacks.
3. Explicit review gate: no phase 4 work starts until phase 1-3 review is complete.

## Phase 1: Native Cleanup and Data Separation
Objective:
- Clean authoring file shape and split app/workspace state from authored content.

### Proposed Initial Field Classification Table (Draft)
Use this as the starting point for Phase 1A sign-off.

| Location | Field | Classification | Action | Notes |
|---|---|---|---|---|
| Project root | `name` | Native user project data | Keep | Authored project identity. |
| Project root | `autoSaveSeconds` | Native user project data | Keep | User-authored project behavior setting. |
| Project root | `commandVerbs` | Native user project data | Keep | Author-defined vocabulary. |
| Project root | `directionals` | Native user project data | Keep | Author-defined vocabulary. |
| Project root | `startingPlanetName` | Native user project data | Keep | Runtime content behavior. |
| Project root | `globalVariables` | Native user project data | Keep | Runtime data. |
| Project root | `uiState` | Native application state data | Move | Selection/context should live in project state sidecar. |
| Project root | `planets` and descendants content | Native user project data | Keep | Core authored content tree. |
| Area (project file) | `roomIds` | Native user project data | Keep | Canonical room membership reference. |
| Area (project file) | `links` | Legacy | Remove write | Keep read only if backward compatibility retained. |
| Area (project file) | `rooms` | Legacy | Remove write | Legacy inline room payload; use room sidecars. |
| Room sidecar | `id`, `name`, `description` | Native user project data | Keep | Core room content. |
| Room sidecar | `producerNotes` | Native user project data | Keep in native files, exclude from clean export | User-authored content notes. |
| Room sidecar | `availableActions[].echoMessage` | Native user project data | Keep | Runtime scripted output now lives on echo actions. |
| Room sidecar | `commandPhrases` | Native user project data | Keep | Runtime command mapping. |
| Room sidecar | `availableActions` | Native user project data | Keep | Runtime action behavior. |
| Room sidecar | `variables` | Native user project data | Keep | Runtime state model. |
| Room sidecar | `additionalVerbs`, `additionalDirectionals` | Native user project data | Keep | Author-defined vocabulary extensions. |
| Room sidecar | `objects` | Native user project data | Keep | Core authored object content. |
| Object payload | `commands` | Native user project data (deferred review) | Keep for now | Planned usage is still expected; revisit during clean export contract design. |
| Object payload | `productionName` | Native user project data | Keep | Used in object token matching and author intent. |
| Navigation sidecar | `areas[].links` | Native user project data | Keep | Canonical navigation topology. |
| Navigation sidecar | `areas[].roomPlacements` | Native user project data | Keep in native project data for now | Borderline case; retain with project content for current workflow.

Decision-needed items for Phase 1A sign-off:
- None. Current Phase 1A classification decisions are resolved.

Resolved decisions:
- `autoSaveSeconds`: Keep in project file as native user project data.
- `producerNotes`: Keep in native user project data files; revisit only for clean export contract.
- object `commands`: Keep in native user project data for now; reassess during clean export contract work.
- `roomPlacements`: Keep in native user project data for now.

### 1A. Field Classification (Class A vs Class B)
- [x] Build and approve field-by-field table from current persisted JSON.
- [x] Mark each field as: Keep, Move to state sidecar, Remove, or Legacy-read-only.

Deliverable:
- [x] Classification table committed to docs.

### 1B. State Sidecar Introduction
- [x] Add project-specific state sidecar model and file naming convention.
- [x] Move `ProjectUiState` persistence from core project file into sidecar.
- [x] Keep machine-wide state in app-level files (recent projects, window placement).

Deliverable:
- [x] Project load/save supports sidecar state path.

### 1C. Debt Field Write Cleanup
- [x] Stop writing agreed debt/legacy fields in authoring files.
- [x] Keep temporary read compatibility during transition.

Initial candidates:
- [x] Remove inline legacy area `rooms` write path.
- [x] Remove inline legacy area `links` write path if sidecar navigation is authoritative.
- [ ] Remove object command string list write path if confirmed unused.

Deliverable:
- [x] Cleaner authoring output produced by save.

### 1D. Fixture Migration
- [x] Normalize all sample files to new intended native shape.
- [x] Remove stale examples of deprecated fields from fixtures.

Deliverable:
- [x] Sample corpus aligned to current format.

### 1E. Backward-Read Decision Gate
- [x] Decide whether to remove backward-read compatibility now.
- [x] If removing: remove legacy readers and update docs with one-time break note.
- [ ] If keeping: add a timebox date and explicit revisit checkpoint.

Deliverable:
- [x] Decision logged with rationale.

Exit Criteria (Phase 1):
- Native files are cleaner and split by data classification.
- Sample fixtures pass load/save and reflect current intended shape.

One-time break note (2026-06-22):
- Backward-compatible read support was removed for:
	- inline `uiState` in project root files,
	- legacy `.rooms.json` room catalogs,
	- inline area `links` and `rooms` payloads inside project files.
- Supported path is now sidecar-driven state/navigation/rooms loading.

---

## Phase 2: Standard Export v1 Build
Objective:
- Build a clean, versioned standard export contract from in-memory model.

### 2A. Contract and DTO Setup
- [x] Define standard export root and file set.
- [x] Add `schemaVersion` and required metadata.
- [x] Implement dedicated standard export DTOs (separate from authoring DTOs).

### 2B. Mapping Service
- [x] Implement model -> standard export mapper.
- [x] Enforce deterministic ordering rules.
- [x] Apply omit rules for null/default/empty fields as approved.

### 2C. Output Workflow
- [x] Add explicit command/workflow to generate standard export bundle.
- [x] Keep authoring save/load independent from standard export generation.

Deliverables:
- [x] Working standard export v1 output from sample projects.
- [x] Short developer note documenting ordering and omission rules.

Exit Criteria (Phase 2):
- Standard export v1 can be generated reliably and is clean for external sharing.

---

## Phase 3: Validation and Adoption Readiness
Objective:
- Add safety rails and collaboration readiness before broader usage.

### 3A. Tests and Validation
- [x] Golden/snapshot tests for representative fixtures.
- [x] Schema validation tests for standard export outputs.
- [x] Determinism checks across repeated exports.
- [x] Reference integrity tests (ids, links, action references).

### 3B. Regression Protection
- [x] Tests proving Class B state data is not present in standard export.
- [x] Tests proving deprecated fields remain absent from current write paths.

### 3C. Adoption Docs
- [x] Publish a short collaboration handoff doc.
- [x] Define change policy for export contract and version bumps.

Exit Criteria (Phase 3):
- Export contract behavior is tested, documented, and ready for external integration.

---

## Review Gate Before Phase 4
Phase 4 does not start until this checklist is complete:
- [x] Phase 1 exit criteria met.
- [x] Phase 2 exit criteria met.
- [x] Phase 3 exit criteria met.
- [ ] Team review completed and decisions recorded.
- [x] Open risks/issues list created for future custom transforms.

## Current Status Snapshot
1. Native cleanup and separation are complete and validated in code/tests.
2. Clean export v1 contract and generation pipeline are complete and validated.
3. Post-plan stabilization fixes are complete:
	- global directionals persistence sync fix,
	- scoped action naming behavior restored (blank name by default; auto-name on save if empty).
4. Remaining gate before phase 4: complete the formal team review and record decisions.

## Open Risks / Issues For Future Custom Transforms
1. Team review not yet recorded, so phase-4 custom transforms are still blocked by process gate.
2. Object `commands` write-path cleanup remains intentionally deferred pending explicit confirmation it is unused.
3. Build reproducibility warning: local app process can lock `StoryboardDesigner.App.exe` during rebuilds (MSB3026 retries).

## Restart Next Time (Fast Resume)
1. Run a quick baseline verification:
	- `dotnet build .\\StoryboardDesigner.slnx`
	- `dotnet test .\\StoryboardDesigner.slnx`
2. Perform formal phase 1-3 review and capture decisions directly in this file.
3. If review passes, open phase-4 planning focused on custom transform backlog and acceptance criteria.

## Recommended Execution Order (Fast Path)
1. Phase 1A + 1B (classification and sidecar split).
2. Phase 1C + 1D (write cleanup and fixture migration).
3. Phase 1E decision (remove or retain backward readers).
4. Phase 2A + 2B + 2C (standard export v1 end-to-end).
5. Phase 3A + 3B + 3C (tests and adoption docs).
6. Formal review gate.

## Session Change Log
- 2026-06-22: Created focused phase 1-3 execution plan with review gate before phase 4.
- 2026-06-22: Decision captured: `autoSaveSeconds` remains native user project data.
- 2026-06-22: Decision captured: `producerNotes` remains native user project data.
- 2026-06-22: Decision captured: object `commands` stays in native user project data for now.
- 2026-06-22: Decision captured: `roomPlacements` remains native user project data for now.
- 2026-06-22: Implemented `ProjectUiState` sidecar persistence with legacy inline fallback and test coverage.
- 2026-06-22: Stopped writing legacy inline area `links` and `rooms` fields in project file while preserving backward-compatible reads.
- 2026-06-22: Migrated sample project files to remove inline `uiState`, `links`, and `rooms`, and added Birmingham project state sidecar.
- 2026-06-22: Decision captured: remove backward-compatible read support now (pre-user-base clean break).
- 2026-06-22: Removed legacy read fallbacks for inline `uiState`, `.rooms.json` catalogs, and inline area `links`/`rooms`; migrated MapDemo1 to room sidecars.
- 2026-06-22: Added clean export v1 contract DTOs and `ExportCleanProjectV1` generator with schemaVersion and deterministic output ordering.
- 2026-06-22: Added `CLEAN_EXPORT_V1.md` documenting clean export file set, schemaVersion, and contract boundaries.
- 2026-06-22: Added omission rules for empty optional room/object clean-export fields and added deterministic clean export test coverage.
- 2026-06-22: Added clean export contract change/versioning policy guidance to `CLEAN_EXPORT_V1.md`.
- 2026-06-22: Added clean export schema-shape and reference-integrity validation tests using Birmingham and single-room fixtures.
- 2026-06-22: Added fixture-based clean export snapshot baseline tests and generated single-room approved snapshots.
- 2026-06-22: Added regression test proving native application state fields are excluded from clean export output.
- 2026-06-22: Added explicit current-status snapshot, open-risks list, and fast-resume checklist for next-session restart.
- 2026-06-22: Recorded post-plan stabilization fixes: global directionals save-sync and scoped-action auto-name behavior.
