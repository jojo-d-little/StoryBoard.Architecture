# Plan Authoring Standard (Staged Handoff Pattern)

Last updated: 2026-09-11
Status: Active standard

Purpose: make the staged handoff planning model the default for future enhancements.

## Standard Summary

Every new non-trivial enhancement plan must be authored as a single canonical plan file with:

1. Locked implementation stage order.
2. A handoff section per stage with explicit downstream consumers.
3. Stage-local codebase context and required validation gates.
4. Explicit stage boundary allowlists for what each stage is allowed to read and edit.
5. A final closeout declaration that references all stage handoff files.
6. A full placeholder handoff document set (Stage 1..N) created at plan start.

This is the repository default pattern for plan-driven work.

## When This Pattern Is Required

Use this pattern when the change touches one or more of the following:

1. More than one project boundary (for example Shared + host).
2. Runtime contracts, schema, DTOs, or transport interfaces.
3. Host/runtime behavior that must be staged to prevent drift.
4. Work expected to span multiple sessions or handoffs.

For tiny single-file fixes, this full pattern is optional.

## Canonical File Locations

1. Main plan: `plans/active/<WORKSTREAM_NAME>_PLAN.md`
2. Stage handoffs: `plans/active/handovers/<PLAN_FILE_STEM>/<WORKSTREAM_NAME>_0N_<STAGE>.md`
3. Template source: `plans/templates/STAGED_HANDOFF_PLAN_TEMPLATE.md`
4. Stage handoff template: `plans/templates/STAGE_HANDOFF_TEMPLATE.md`
5. Standard stage catalog: `plans/STANDARD_STAGE_CATALOG.md`

## Fixed Stage Catalog Policy (Required)

The repository has a hardened default stage set and order in `plans/STANDARD_STAGE_CATALOG.md`.

Authoring rules:

1. Start from the standard stage set and order by default.
2. Per-plan drift should primarily be stage inclusion status (`Required`, `Optional`, `Skipped`) rather than redefining stage meaning.
3. Do not rewrite stage boundary allowlists from scratch unless a justified override is required.
4. If a stage is skipped, include explicit reason and impact.
5. Respect the compatibility split:
- Stage 01 is backward-compatible contract work only.
- Stage 06 is non-backward-compatible contract retirement work.

## Required Plan Structure

Every staged plan must include these sections in order:

1. Problem Statement
2. Non-Goals
3. Current Baseline
4. Proposed Shape
5. Initial Scope Breakdown (by stage)
6. Structured Delivery Order And Session Handoffs (Locked)
7. Stage Inclusion Matrix (Required/Optional/Skipped per standard stage)
8. Stage 01..07 sections with:
- Goal
- Primary output handoff document path
- Required handoff contents
- Downstream usage
- Stage boundary allowlists (allowed read scope and allowed edit scope)
9. Risk Register
10. Validation Gates
11. MVP/Completion Acceptance Criteria
12. Final Closeout Checklist

## Stage Handoff Requirements

Each stage handoff document must include all of:

1. Scope completed
2. Files changed
3. Contract/interface impact
4. Validation commands executed
5. Test results
6. Behavioral notes
7. Known issues/risks
8. Explicit next-stage start checklist
9. Opening prompt for that stage (ready-to-paste stage-start prompt at the top of the handoff)
10. Stage boundary allowlist snapshot copied from the main plan
11. Boundary compliance report (out-of-scope reads/edits and approvals)
12. Compatibility mode declaration for contract stages (`Backward-compatible` for Stage 01, `Non-backward-compatible` for Stage 06)

## Handoff Folder Structure (Required)

Every new staged plan owns a dedicated handoff directory named exactly for the main plan file stem (the filename without `.md`).

1. Main plan: `plans/active/<PLAN_FILE_STEM>.md`
2. Its handoff folder: `plans/active/handovers/<PLAN_FILE_STEM>/`
3. Its stage files: `plans/active/handovers/<PLAN_FILE_STEM>/<WORKSTREAM_NAME>_0N_<STAGE>_HANDOFF.md`

Examples:

1. `plans/active/PORTABLE_PATH_EXPANSION_AND_MULTI_ROOT_ASSETS_PLAN.md`
2. `plans/active/handovers/PORTABLE_PATH_EXPANSION_AND_MULTI_ROOT_ASSETS_PLAN/`
3. `plans/active/handovers/PORTABLE_PATH_EXPANSION_AND_MULTI_ROOT_ASSETS_PLAN/PORTABLE_PATH_EXPANSION_01_SHARED_PORTABILITY_HANDOFF.md`

Do not add new handoff files directly under `plans/active/handovers/`. Existing flat handoff sets remain in place as historical/transition material unless an explicit migration is approved.

## Stage Boundary Policy (Required)

Every stage definition in the main plan must contain explicit allowlists:

1. Allowed read scope.
2. Allowed edit scope.

Edit-implies-read rule:

1. Any path in allowed edit scope is automatically readable.
2. Use allowed read scope only for extra read-only dependencies outside edit scope.

Default deny rule:

1. Any path not explicitly listed in allowed read/edit scope is out of scope.

Enforcement rules:

1. Edit allowlists are hard boundaries. If a required change falls outside the current stage edit scope, stop and record a plan/stage-boundary decision before editing.
2. Read allowlists should be tight to reduce context costs. Read outside the allowlist only when needed for debugging/validation, and record it in the handoff boundary compliance report.
3. Every stage opening prompt must instruct execution to honor both allowlists and prohibit downstream-stage edits.
4. Stage handoff completion is blocked until boundary compliance is documented.

Compatibility enforcement rules:
1. Stage 01 must not include removals/renames/requiredness-tightening/type-narrowing on accepted contracts.
2. Non-backward-compatible contract changes must be deferred to Stage 06 unless an explicit emergency exception is approved.
3. Stage 06 must include retirement list, downstream impact summary, and contract/codegen guardrail evidence.
4. Broad regression evidence is owned by Stage 07 closeout.

## Plan Initialization Gate (Required)

Before implementation begins, complete this gate in one pass:

1. Lock the full Stage 1..N delivery order in the main plan.
2. Create all stage handoff files immediately as placeholders (not only Stage 1).
3. Add stage metadata (`Status`, `Stage`, `Date`, `Owner Session`) to each placeholder.
4. Include an `Opening Prompt (Use To Start This Stage)` section in each placeholder.
5. Confirm all placeholder handoffs are listed in the main plan under each stage's primary output path.
6. Pre-fill each placeholder handoff with stage boundary allowlist snapshot sections.
7. Add a Stage Inclusion Matrix and mark each standard stage `Required`, `Optional`, or `Skipped`.
8. Explicitly mark Stage 06 as `Required`, `Optional`, or `Skipped` and record retirement rationale.

Rationale:
1. Makes the end-to-end negotiation cadence visible at plan creation time.
2. Prevents missing downstream handoff setup during early execution.

## Codebase Context Requirement Per Stage

Each stage section in the main plan must include a short "Codebase Context" block listing:

1. Primary projects in scope.
2. Key files expected to change first.
3. Boundary constraints to preserve.
4. Tests that must be run before handoff acceptance.
5. Allowed read scope.
6. Allowed edit scope.

This keeps handoffs actionable without rediscovery.

## Closeout Rules

A workstream can be closed only when:

1. All locked stages are marked complete.
2. All planned stage handoff docs exist and are status-complete.
3. Required validation gates have been executed and reported.
4. Any deferred items are explicitly listed as non-blocking follow-up.

## Archival Unit Rule (Required)

When archiving a completed staged workstream, archive the full document set as one unit:

1. Main plan file under `plans/active/`.
2. Its dedicated stage-handoff directory under `plans/active/handovers/<PLAN_FILE_STEM>/`.

Do not archive only the main plan or only a subset of handoffs. Partial archival is considered incomplete closeout.

Required archive checklist for each completed workstream:

1. Move main plan to `plans/archived/`.
2. Move the complete handoff directory to `plans/archived/handovers/<PLAN_FILE_STEM>/` (or equivalent archived handoff location used by the repo).
3. Preserve naming parity so stage sequence remains obvious after move.
4. Update `plans/README.md` with one archive entry that references the workstream as a single unit.

## Practical Authoring Flow

1. Copy `plans/templates/STAGED_HANDOFF_PLAN_TEMPLATE.md` to `plans/active/<NEW_PLAN>.md`.
2. Open `plans/STANDARD_STAGE_CATALOG.md` and select stage inclusion (`Required`, `Optional`, `Skipped`) for this workstream.
3. Lock Stage order before coding begins (preserve standard order for included stages, including Stage 06 when used).
4. Create `plans/active/handovers/<PLAN_FILE_STEM>/`, then create handoff files for all included stages in that directory immediately using `plans/templates/STAGE_HANDOFF_TEMPLATE.md` as placeholders.
5. Copy stage boundary allowlists from the main plan into each stage handoff placeholder.
6. Complete one stage at a time and replace placeholder sections with executed outcomes.
7. Start next stage only from prior stage handoff outputs.
8. At finish, update plan progress and closeout checklist.

## Worked Example

Use this closed example as the baseline style reference:

1. Main plan: `plans/active/PORTRAIT_ROOM_ORIENTATION_HANDOFF_PLAN.md`
2. Stage handoffs:
- `plans/active/handovers/PORTRAIT_ROOM_ORIENTATION_01_CONTRACT_HANDOFF.md`
- `plans/active/handovers/PORTRAIT_ROOM_ORIENTATION_02_DESIGNER_HANDOFF.md`
- `plans/active/handovers/PORTRAIT_ROOM_ORIENTATION_03_GAMEENGINE_HANDOFF.md`
- `plans/active/handovers/PORTRAIT_ROOM_ORIENTATION_04_WEBPORTAL_HANDOFF.md`
- `plans/active/handovers/PORTRAIT_ROOM_ORIENTATION_05_SIMULATOR_HANDOFF.md`
