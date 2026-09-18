# Portable Path Expansion — Stage 04 Simulator Verification Handoff

Status: Placeholder — not started  
Stage: 04 of 07  
Date: 2026-09-18  
Owner Session: Unassigned

## Opening Prompt (Use To Start This Stage)

Execute only Stage 04 of `plans/active/PORTABLE_PATH_EXPANSION_AND_MULTI_ROOT_ASSETS_PLAN.md`, starting immediately from Stage 03. Simulator verification is intentionally adjacent to its GameEngine/Host ownership area. Verify Simulator receives host/runtime-export assets and does not consume source-root expressions. Change Simulator only for a proven consumer defect and record the end-to-end two-root test result.

## Stage Boundary Allowlist Snapshot

Allowed read: Stage 03 handoff, `StoryBoard.GameEngine/Storyboard.Simulator*/**`, Host asset contracts.  
Allowed edit: `StoryBoard.GameEngine/Storyboard.Simulator*/**` only for a proven defect; otherwise this handoff only.

## Scope Completed

Not started.

## Files Changed

None.

## Contract/Interface Impact

Expected: none.

## Validation Commands Executed

None.

## Test Results

Pending two-source-root-to-host-to-simulator verification.

## Behavioral Notes

Simulator cache paths and command-line paths are not authoring source roots.

## Known Issues/Risks

Do not widen simulator filesystem access to compensate for an upstream publishing issue.

## Boundary Compliance Report

Out-of-scope reads: None.  
Out-of-scope edits: None.  
Exceptions: None.

## Explicit Next-Stage Start Checklist

1. Provide verification/no-change evidence to WebPortal stage.
2. Preserve runtime-relative asset URL invariants.
3. Record whether Simulator had no required implementation change.
