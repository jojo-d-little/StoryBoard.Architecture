# <Workstream Name> - Stage <N> <Stage Name> Handoff

Status: Complete
Stage: <N> of <Total>
Date: <YYYY-MM-DD>
Owner Session: <Agent/Owner>

## Opening Prompt (Use To Start This Stage)

<Provide a ready-to-paste prompt that starts this stage only, references the main plan file, enforces locked order, and states required validation/reporting expectations.>

## Stage Boundary Allowlist Snapshot (From Main Plan)

Default deny rule:
1. Any path not explicitly listed in allowed read/edit scope is out of scope for this stage.

Edit-implies-read rule:
1. Any path in allowed edit scope is automatically readable.
2. Allowed read scope should list only extra read-only dependencies.

1. Allowed read scope:
- <project/folder list>
2. Allowed edit scope:
- <project/folder list>

## Scope Completed

1. <Completed scope item>
2. <Completed scope item>

## Files Changed

1. <path/file>
- <What changed>
2. <path/file>
- <What changed>

## Contract/Interface Impact

1. <No impact or describe exact impact>
2. <Compatibility notes>

## Validation Commands Executed

1. `<command>`
- <PASS/FAIL and high-level count>
2. `<command>`
- <PASS/FAIL and high-level count>

## Test Results

1. <New tests added/updated>
2. <Regression suites impacted>
3. <Pass/fail summary>

## Behavioral Notes

1. <Behavioral confirmation>
2. <Known accepted MVP behavior>

## Known Issues/Risks

1. <Issue/risk>
2. <Deferred follow-up>

## Boundary Compliance Report

1. Out-of-scope reads performed:
- <None or list>
2. Out-of-scope edits performed:
- <None expected. If any, include approval and reason>
3. Stage-boundary exceptions approved:
- <None or approval reference>
4. Session context scope notes:
- <Any context-limit observations for future sessions>

## Explicit Next-Stage Start Checklist

1. <What next stage should read first>
2. <What invariants must remain unchanged>
3. <Which validation gate must run early>
