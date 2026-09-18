# Globals Sidecar And Import Refresh Plan

Status: In Progress (G1 complete, G3 keep/replace-all flow shipped)
Owner: StoryboardDesigner authoring
Last updated: 2026-07-02

## Implementation Delta (2026-07-02)

Completed in current slice:

1. Globals sidecar persistence shipped in `JsonExportService`:
   - Save writes `<ProjectName>.sbe.globals.json`.
   - Load prefers globals sidecar values when present and falls back to legacy main-file globals when missing.
2. File menu command shipped: `Import Globals From Project`.
3. Import dialog shipped with:
   - Section checkboxes (verbs, directionals, template objects, global game objects).
   - One-shot collision strategy selector: Keep Existing or Replace Existing.
4. Import apply behavior shipped:
   - Verbs/directionals import additively with duplicate skip.
   - Directional mappings import additively; collisions respect strategy.
   - Template/global-object collisions support keep-all or replace-all in one pass.
5. Regression coverage added:
   - Sidecar write + load-prefer-sidecar test.
   - Legacy fallback load test when globals sidecar is missing.
6. Deferred from lock target to next slice:
   - Ask Per Conflict per-item decision flow (keep/replace-all one-shot flow is implemented now).

Validation completed:

1. `dotnet build .\StoryboardDesigner.slnx` passed.
2. Focused suite passed:
   - `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "JsonExportServiceProjectStateTests|TraversalWizardApplyPipelineTests"`
3. Full suite passed:
   - `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj` (245/245).

## 1. Purpose

Define a producer-controlled globals workflow that removes hardcoded startup boilerplate pressure and makes repeated refresh from a master project fast and deterministic.

## 2. Problem Statement

Small project creation for focused testing needs reusable global definitions (verbs, directionals, templates, global objects) without repeatedly authoring them from scratch.

Current behavior mixes convenience defaults with producer ownership concerns.

## 3. Goals

1. Move reusable global definitions into a standard per-project sidecar file.
2. Make import-from-project the primary reuse workflow (no dedicated export required in v1).
3. Support refresh workflows with one-shot collision handling: Keep All, Replace All, Ask Per Conflict.
4. Keep data mutation explicit, deterministic, and undoable.
5. Preserve project/runtime boundaries and existing export determinism.

## 4. Non-Goals (v1)

1. No dedicated Globals Export command.
2. No template/profile marketplace or packaged preset catalog.
3. No automatic background syncing with a source project.
4. No cross-project live link semantics.

## 5. Locked Decisions

Status key:

1. Locked: approved for implementation.

| Decision Point | Locked Decision | Status |
| --- | --- | --- |
| Persistence shape | Globals live in a standard sidecar file next to project file. | Locked |
| Sidecar file name | Use <ProjectName>.sbe.globals.json. | Locked |
| Primary workflow | Import globals from another project sidecar file. | Locked |
| Export in v1 | Not implemented in v1 (deferred). | Locked |
| Sections in scope | Verbs, Directionals, Template Objects, Global Game Objects. | Locked |
| Collision identity for templates/objects | Name-based, case-insensitive matching. | Locked |
| Verbs collision behavior | Additive only; duplicates skipped. | Locked |
| Directionals collision behavior | Additive only; duplicates skipped. | Locked |
| Template/global object collision options | Keep All, Replace All, Ask Per Conflict. | Locked |
| Replace semantics | Deep replace matched item content while preserving destination identity key fields required for local reference stability. | Locked |
| Import transaction boundary | One atomic transaction with one undo step. | Locked |
| New project flow | After project creation, offer Import Globals Now prompt (non-blocking). | Locked |
| Versioning | Sidecar contains explicit formatVersion from v1. | Locked |
| Missing sidecar behavior | Treat as empty globals; create sidecar on first save or first globals mutation. | Locked |

## 6. Data Contract (v1)

Root object fields:

1. formatVersion: string (initial value 1.0).
2. sourceProjectName: string.
3. updatedUtc: string (ISO-8601 UTC).
4. verbs: array (optional when empty).
5. directionals: array (optional when empty).
6. templateObjects: array (optional when empty).
7. globalGameObjects: array (optional when empty).

Determinism requirements:

1. Persist arrays in deterministic order.
2. Use stable property ordering in serialization where current serializer policy allows.
3. Import processing order is deterministic by section then case-insensitive name.

## 7. UX Contract

## 7.1 File Menu Actions

1. Add Import Globals From Project... under File.
2. No Export Globals item in v1.

## 7.2 Import Dialog Flow

1. Select source project or direct sidecar path.
2. Inspect sidecar and show only sections present.
3. Section checkboxes allow subset import.
4. Collision strategy selector (single choice):
   - Keep All Conflicts
   - Replace All Conflicts
   - Ask Per Conflict
5. For Ask Per Conflict, per-item decision list supports Apply To Remaining.
6. Preview summary before apply:
   - New items
   - Kept items
   - Replaced items
7. Confirm applies import atomically and emits summary diagnostics.

## 7.3 New Project Prompt

After successful project creation:

1. Prompt: Import globals from another project now?
2. Actions:
   - Import Now
   - Skip
3. Skip leaves project with minimal baseline globals only.

## 8. Migration Strategy

## 8.1 Existing Projects (Backward Compatibility)

1. On load, if sidecar exists: load globals from sidecar.
2. If sidecar does not exist but legacy globals are in main project content:
   - Read legacy globals.
   - Continue functioning normally for this session.
   - On next save, write sidecar and remove duplicated legacy globals from main file as part of a controlled migration step.
3. Persist migration marker/version so conversion is idempotent.

## 8.2 Save Behavior

1. Saving project writes main project file and globals sidecar deterministically.
2. Save failure in either file path surfaces clear error and preserves in-memory state.

## 9. Import Semantics

## 9.1 Verbs And Directionals

1. Normalize compare key as case-insensitive name.
2. Add incoming entries absent in target.
3. Skip duplicates and count as kept/skipped.

## 9.2 Template Objects And Global Game Objects

1. Match by case-insensitive name.
2. If no match: add.
3. If match:
   - Keep All: skip incoming.
   - Replace All: deep replace matched content.
   - Ask Per Conflict: user selects keep/replace per item.
4. Replacement records include before/after counts for diagnostics.

## 10. Diagnostics And Undo

1. Import result summary includes per-section Added/Kept/Replaced/Skipped counts.
2. Detailed conflict decisions are written to output diagnostics.
3. Entire import is one undoable action.

## 11. Implementation Phases

## Phase G1: Contracts And Storage

Deliverables:

1. Globals sidecar DTO/schema and serializer.
2. Project save/load integration with sidecar read/write.
3. Legacy project migration path.

Validation gate:

1. Build passes.
2. Save/load round-trip tests pass for sidecar and migrated legacy projects.

## Phase G2: Import Engine

Deliverables:

1. Source sidecar inspection.
2. Import planning and collision detection.
3. Apply engine with Keep All / Replace All / Ask Per Conflict.
4. Atomic undo transaction.

Validation gate:

1. Unit tests for additive and replacement semantics.
2. Deterministic import ordering tests.

## Phase G3: Import UX

Deliverables:

1. File menu command and dialog flow.
2. Section selection UI and strategy selector.
3. Preview summary and post-apply diagnostics summary.
4. New-project import-now prompt.

Validation gate:

1. Workflow tests for keep-all, replace-all, ask-per-conflict paths.
2. Manual UX smoke on repeated refresh from same source project.

## Phase G4: Hardening

Deliverables:

1. Error handling for malformed/missing sidecar.
2. Version mismatch handling.
3. Regression safeguards for clean export determinism and runtime boundaries.

Validation gate:

1. StoryboardDesigner.App.Tests full suite passes.
2. Runtime-focused guardrail suite passes.

## 12. Acceptance Criteria

1. New project can import globals from another project in one guided flow.
2. Reimport refresh can be completed with one global collision strategy choice.
3. Verbs and directionals import additively and skip duplicates.
4. Template/global object conflicts honor Keep All, Replace All, or Ask Per Conflict exactly.
5. Replace operation deep-replaces selected items and is undoable in one step.
6. Existing projects migrate safely to sidecar without data loss.
7. Save/load and clean export remain deterministic after import and migration.

## 13. Open Items (Non-Blocking, Post-v1)

1. Optional standalone globals export/profile packaging.
2. Optional strict sync mode for verbs/directionals (replace set).
3. Optional source fingerprinting to show freshness drift before reimport.

## 14. Immediate Execution Checklist

1. Implement G1 storage contracts and migration tests.
2. Implement G2 import engine with top-level collision strategy.
3. Implement G3 dialog and new-project prompt.
4. Run focused and full validation gates.
5. Update plan status from Locked And Implementation-Ready to In Progress once coding starts.
