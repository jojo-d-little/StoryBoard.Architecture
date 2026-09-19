# Portable Path Expansion — Stage 06 Legacy Migration Handoff

Status: Complete  
Stage: 06 of 07  
Date: 2026-09-18  
Owner Session: Codex

## Opening Prompt (Use To Start This Stage)

Execute Stage 06 only after explicit approval recorded in the main plan and completion of Stages 01-05. Inventory every authored `ASSETROOT:/` reference and eligible non-token absolute asset path, migrate only approved checked-in project/starter data through tested normalization tooling, regenerate derived outputs from those sources where approved, and decide whether legacy read support remains. Do not use blind replacement and do not remove compatibility without external-project migration evidence.

## Stage Boundary Allowlist Snapshot

Allowed read: repository-wide for inventory and all prior handoffs.  
Allowed edit: explicitly approved project-data folders, `StoryBoard.Designer/**`, Architecture docs, this handoff.

## Scope Completed

Migrated the approved controlled project data to canonical
`%STORYBOARD_ASSET_SOURCE_ROOT...%/...` references:

- `StoryBoard.SampleProjects`: 55 authored JSON files, 130 references.
- `StoryBoard.Game.TheDocks`: 20 authored JSON files, 54 references.
- Designer `StarterProjects/BigHeadStart`: 8 authored JSON files, 29 references.

Generated runtime/export manifests were not edited directly. They remain derived
outputs and must be regenerated from the migrated authored sources when needed.

## Files Changed

Approved authored project-data folders listed above, plus the Designer startup
environment refresh and the asset-root documentation.

## Contract/Interface Impact

`ASSETROOT:/` is retired. It is no longer a supported authored-data format.
No DTO or runtime asset-schema change was made.

## Validation Commands Executed

- Authored JSON parse validation across all three project areas.
- Canonical reference inventory and source-file existence validation.
- Repository search confirming no legacy token remains in authored project data.

## Test Results

212 canonical primary-root references checked; 0 missing source files; 0 authored
JSON parse failures.

## Behavioral Notes

Legacy `ASSETROOT:/` reads are being removed. Older external projects must be
migrated before use.

## Known Issues/Risks

Checked-in generated runtime artifacts and authored sources must stay coherent.

## Boundary Compliance Report

Out-of-scope reads: None.  
Out-of-scope edits: None.  
Exceptions: None.

## Explicit Next-Stage Start Checklist

1. Run the cross-system portability matrix.
2. Regenerate derived runtime outputs from migrated authored sources where needed.
3. Record the final unsupported-legacy decision in Stage 07 closeout.
