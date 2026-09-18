# Portable Path Expansion — Stage 01 Shared Portability Handoff

Status: Placeholder — not started  
Stage: 01 of 07  
Date: 2026-09-18  
Owner Session: Unassigned

## Opening Prompt (Use To Start This Stage)

Execute only Stage 01 of `plans/active/PORTABLE_PATH_EXPANSION_AND_MULTI_ROOT_ASSETS_PLAN.md`. Create and test the dependency-neutral `Storyboard.Foundation` package, shipping only its `Paths` capability and documented `%NAME%` expression/error contract. Do not implement the deferred Content, Files, or Json fast followers; do not edit Designer, GameEngine, project data, or legacy-token behavior. Record package version, test results, boundary compliance, and the exact Stage 02 adoption steps here.

## Stage Boundary Allowlist Snapshot

Allowed read: `StoryBoard.Architecture/docs/**`, `StoryBoard.Designer/**`, `StoryBoard.GameEngine/**`, package/version documentation.  
Allowed edit: `StoryBoard.Contracts/**`, `StoryBoard.Architecture/docs/**`, main plan, this handoff.

## Scope Completed

Not started.

## Files Changed

None.

## Contract/Interface Impact

Pending: package API only; no DTO/schema change is permitted.

## Validation Commands Executed

None.

## Test Results

Pending.

## Behavioral Notes

Pending.

## Known Issues/Risks

`ASSETROOT:/` remains supported by Designer until Stage 02 adopts the package.

## Boundary Compliance Report

Out-of-scope reads: None.  
Out-of-scope edits: None.  
Exceptions: None.

## Explicit Next-Stage Start Checklist

1. Consume the published/local `Storyboard.Foundation` package version recorded here.
2. Preserve legacy token read compatibility.
3. Start with multi-root resolver tests before dialog changes.
