# Portable Path Expansion — Stage 02 Designer Handoff

Status: Placeholder — not started  
Stage: 02 of 07  
Date: 2026-09-18  
Owner Session: Unassigned

## Opening Prompt (Use To Start This Stage)

Execute only Stage 02 of `plans/active/PORTABLE_PATH_EXPANSION_AND_MULTI_ROOT_ASSETS_PLAN.md`, starting from the completed Stage 01 handoff. Replace Designer's single-root persistence behavior with canonical `%STORYBOARD_ASSET_SOURCE_ROOT...%/` expressions, use deepest-root selection and explicit tie handling, retain `ASSETROOT:/` reads, and cover image, sound, preview, validation, and export. Do not change GameEngine/Host or migrate project data.

## Stage Boundary Allowlist Snapshot

Allowed read: Stage 01 handoff/package, Architecture docs, Designer tests, GameEngine runtime-output contracts.  
Allowed edit: `StoryBoard.Designer/**`, `StoryBoard.Architecture/docs/**`, this handoff.

## Scope Completed

Not started.

## Files Changed

None.

## Contract/Interface Impact

Authored string values gain canonical `%...%` persistence; runtime output remains runtime-relative.

## Validation Commands Executed

None.

## Test Results

Pending primary, named, nested, tied, legacy, image, sound, preview, and export cases.

## Behavioral Notes

New writes must not emit `ASSETROOT:/`.

## Known Issues/Risks

Out-of-root assets remain valid but non-portable until deliberately relocated or configured.

## Boundary Compliance Report

Out-of-scope reads: None.  
Out-of-scope edits: None.  
Exceptions: None.

## Explicit Next-Stage Start Checklist

1. Verify exact package version and resolver semantics used.
2. Verify manifests retain authored source expressions and staged runtime assets stay relative.
3. Supply runtime invariants and test fixtures to Stage 03.
