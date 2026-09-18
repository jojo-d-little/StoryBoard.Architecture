# Portable Path Expansion — Stage 06 Legacy Migration Handoff

Status: Placeholder — approval required  
Stage: 06 of 07  
Date: 2026-09-18  
Owner Session: Unassigned

## Opening Prompt (Use To Start This Stage)

Execute Stage 06 only after explicit approval recorded in the main plan and completion of Stages 01-05. Inventory every authored `ASSETROOT:/` reference and eligible non-token absolute asset path, migrate only approved checked-in project/starter data through tested normalization tooling, regenerate derived outputs from those sources where approved, and decide whether legacy read support remains. Do not use blind replacement and do not remove compatibility without external-project migration evidence.

## Stage Boundary Allowlist Snapshot

Allowed read: repository-wide for inventory and all prior handoffs.  
Allowed edit: explicitly approved project-data folders, `StoryBoard.Designer/**`, Architecture docs, this handoff.

## Scope Completed

Not started; approval required.

## Files Changed

None.

## Contract/Interface Impact

Potential authored-data compatibility retirement only; no unapproved DTO/schema change.

## Validation Commands Executed

None.

## Test Results

Pending migration inventory and open/save/publish verification.

## Behavioral Notes

Legacy reads remain until an approved removal decision is made.

## Known Issues/Risks

Checked-in generated runtime artifacts and authored sources must stay coherent.

## Boundary Compliance Report

Out-of-scope reads: None.  
Out-of-scope edits: None.  
Exceptions: None.

## Explicit Next-Stage Start Checklist

1. Record migration counts and retained legacy exceptions.
2. Supply temporary-root validation fixtures to Stage 07.
3. State whether legacy read support is retained or removed.
