# <Workstream Name> Handoff Plan

Last updated: <YYYY-MM-DD>
Status: Draft (handoff-ready)

Purpose: capture a staged implementation path with locked order, per-stage context, and formal handoff outputs.

Stage baseline source:
1. Use `plans/STANDARD_STAGE_CATALOG.md` for default stage IDs, order, and read/edit boundaries.
2. This plan should vary mainly by stage inclusion (`Required`, `Optional`, `Skipped`) and workstream-specific file/test targets.
3. Respect contract compatibility split: Stage 01 for backward-compatible additions and Stage 06 for non-backward-compatible retirement.

## Problem Statement

Describe the user/problem outcome and what changes in behavior are expected.

## Non-Goals

1. <Non-goal>
2. <Non-goal>
3. <Non-goal>

## Current Baseline (Observed)

1. <Current behavior>
2. <Current behavior>
3. <Current behavior>

## Proposed Shape

1. <Design choice>
2. <Design choice>
3. <Design choice>

## Initial Scope Breakdown

### A) <Stage Group Name>

1. <Scope item>
2. <Scope item>

Estimated effort: <Low/Medium/High>.

### B) <Stage Group Name>

1. <Scope item>
2. <Scope item>

Estimated effort: <Low/Medium/High>.

## Structured Delivery Order And Session Handoffs (Locked)

Implementation order is fixed for this workstream:

1. Stage 01: Contracts And Shared Runtime Mapping
2. Stage 02: Designer Authoring UX
3. Stage 03: GameEngine Runtime Integration
4. Stage 04: Host Interface And Web Portal Runtime Consumption
5. Stage 05: Simulator Host Parity
6. Stage 06: Contract Retirement (Non-Backwards-Compatible)
7. Stage 07: Regression Hardening And Closeout

Use the Stage Inclusion Matrix below to mark each stage as Required, Optional, or Skipped.

Each stage must end with a formal handoff markdown document before downstream work begins.

Plan initialization requirement:
1. Create all Stage 1..N handoff files as placeholders immediately when this plan is created.
2. Do not begin implementation until every listed stage has a placeholder handoff file in `plans/active/handovers/`.

## Stage Inclusion Matrix

Fill this from `plans/STANDARD_STAGE_CATALOG.md`:

| Stage ID | Stage Name | Inclusion (Required/Optional/Skipped) | Reason (if not Required) | Boundary Override (if any) |
| --- | --- | --- | --- | --- |
| 01 | Contracts And Shared Runtime Mapping (Backwards-Compatible) | Required |  | None |
| 02 | Designer Authoring UX | Required |  | None |
| 03 | GameEngine Runtime Integration | Required |  | None |
| 04 | Host Interface And Web Portal Runtime Consumption | Required |  | None |
| 05 | Simulator Host Parity | Optional | <reason> | None |
| 06 | Contract Retirement (Non-Backwards-Compatible) | Optional | <reason> | None |
| 07 | Regression Hardening And Closeout | Required |  | None |

Rules:
1. Preserve stage order for included stages.
2. If a stage is `Skipped`, document impact in this matrix and Risk Register.
3. Any read/edit boundary override must be copied into stage section and handoff boundary snapshot.
4. Stage 01 may only contain backward-compatible contract changes.
5. Any contract removals/refactors must be scheduled in Stage 06.

Stage boundary requirement:
1. Every stage must define an explicit allowed read scope and allowed edit scope.
2. Default deny: any path not listed in stage read/edit scope is out of scope.
3. Edit scope is a hard boundary for that stage; do not edit outside it.
4. If a change is discovered outside stage edit scope, stop and record a boundary decision before continuing.
5. Edit implies read: do not duplicate edit-scope paths in read scope.
6. Use read scope only for extra read-only dependencies outside edit scope.

Current progress:
- Stage 01 (<Stage 1 Name>): Not started.
- Stage 02 (<Stage 2 Name>): Not started.
- Stage 03 (<Stage 3 Name>): Not started.
- Stage 04 (<Stage 4 Name>): Not started.
- Stage 05 (<Stage 5 Name>): Not started / Optional.
- Stage 06 (<Stage 6 Name>): Not started / Optional.
- Stage 07 (<Stage 7 Name>): Not started.
- Workstream status: Not started.

Initial placeholder handoff file set (create up front for included stages):
1. plans/active/handovers/<WORKSTREAM>_01_<STAGE>_HANDOFF.md
2. plans/active/handovers/<WORKSTREAM>_02_<STAGE>_HANDOFF.md
3. plans/active/handovers/<WORKSTREAM>_03_<STAGE>_HANDOFF.md
4. plans/active/handovers/<WORKSTREAM>_04_<STAGE>_HANDOFF.md
5. plans/active/handovers/<WORKSTREAM>_05_<STAGE>_HANDOFF.md
6. plans/active/handovers/<WORKSTREAM>_06_<STAGE>_HANDOFF.md
7. plans/active/handovers/<WORKSTREAM>_07_<STAGE>_HANDOFF.md

### Stage 1: <Stage 1 Name>

Goal:
- <Goal statement>

Codebase context:
1. Primary projects in scope: <project list>
2. Key files expected first: <file list>
3. Boundary constraints: <constraints>
4. Required validation/tests before handoff: <tests>
5. Allowed read scope: <project/folder allowlist>
6. Allowed edit scope: <project/folder allowlist>

Primary output handoff document:
- plans/active/handovers/<WORKSTREAM>_01_<STAGE>_HANDOFF.md

Required handoff contents:
- Scope completed
- Files changed
- Contract/interface impact
- Validation commands executed
- Test results
- Behavioral notes
- Known issues/risks
- Explicit next-stage start checklist
- Stage boundary allowlist snapshot (read/edit)
- Boundary compliance report (out-of-scope reads/edits and approvals)

Downstream usage:
- Stage 2 starts from this handoff.

### Stage 2: <Stage 2 Name>

Goal:
- <Goal statement>

Codebase context:
1. Primary projects in scope: <project list>
2. Key files expected first: <file list>
3. Boundary constraints: <constraints>
4. Required validation/tests before handoff: <tests>
5. Allowed read scope: <project/folder allowlist>
6. Allowed edit scope: <project/folder allowlist>

Primary output handoff document:
- plans/active/handovers/<WORKSTREAM>_02_<STAGE>_HANDOFF.md

Required handoff contents:
- Scope completed
- Files changed
- Contract/interface impact
- Validation commands executed
- Test results
- Behavioral notes
- Known issues/risks
- Explicit next-stage start checklist
- Stage boundary allowlist snapshot (read/edit)
- Boundary compliance report (out-of-scope reads/edits and approvals)

Downstream usage:
- Stage 3 starts from this handoff.

### Stage 3: <Stage 3 Name>

Goal:
- <Goal statement>

Codebase context:
1. Primary projects in scope: <project list>
2. Key files expected first: <file list>
3. Boundary constraints: <constraints>
4. Required validation/tests before handoff: <tests>
5. Allowed read scope: <project/folder allowlist>
6. Allowed edit scope: <project/folder allowlist>

Primary output handoff document:
- plans/active/handovers/<WORKSTREAM>_03_<STAGE>_HANDOFF.md

Required handoff contents:
- Scope completed
- Files changed
- Contract/interface impact
- Validation commands executed
- Test results
- Behavioral notes
- Known issues/risks
- Explicit next-stage start checklist
- Stage boundary allowlist snapshot (read/edit)
- Boundary compliance report (out-of-scope reads/edits and approvals)

Downstream usage:
- Stage 4 starts from this handoff.

### Stage 4: <Stage 4 Name>

Goal:
- <Goal statement>

Codebase context:
1. Primary projects in scope: <project list>
2. Key files expected first: <file list>
3. Boundary constraints: <constraints>
4. Required validation/tests before handoff: <tests>
5. Allowed read scope: <project/folder allowlist>
6. Allowed edit scope: <project/folder allowlist>

Primary output handoff document:
- plans/active/handovers/<WORKSTREAM>_04_<STAGE>_HANDOFF.md

Required handoff contents:
- Scope completed
- Files changed
- Contract/interface impact
- Validation commands executed
- Test results
- Behavioral notes
- Known issues/risks
- Explicit next-stage start checklist
- Stage boundary allowlist snapshot (read/edit)
- Boundary compliance report (out-of-scope reads/edits and approvals)

Downstream usage:
- Stage 5 starts from this handoff.

### Stage 5: <Stage 5 Name>

Goal:
- <Goal statement>

Codebase context:
1. Primary projects in scope: <project list>
2. Key files expected first: <file list>
3. Boundary constraints: <constraints>
4. Required validation/tests before handoff: <tests>
5. Allowed read scope: <project/folder allowlist>
6. Allowed edit scope: <project/folder allowlist>

Primary output handoff document:
- plans/active/handovers/<WORKSTREAM>_05_<STAGE>_HANDOFF.md

Required handoff contents:
- Scope completed
- Files changed
- Contract/interface impact
- Validation commands executed
- Test results
- Behavioral notes
- Known issues/risks
- Explicit next-stage start checklist
- Stage boundary allowlist snapshot (read/edit)
- Boundary compliance report (out-of-scope reads/edits and approvals)

Downstream usage:
- Stage 6 starts from this handoff.

### Stage 6: <Stage 6 Name>

Goal:
- <Goal statement>

Codebase context:
1. Primary projects in scope: <project list>
2. Key files expected first: <file list>
3. Boundary constraints: <constraints>
4. Required validation/tests before handoff: <tests>
5. Allowed read scope: <project/folder allowlist>
6. Allowed edit scope: <project/folder allowlist>

Primary output handoff document:
- plans/active/handovers/<WORKSTREAM>_06_<STAGE>_HANDOFF.md

Required handoff contents:
- Scope completed
- Files changed
- Contract/interface impact
- Validation commands executed
- Test results
- Behavioral notes
- Known issues/risks
- Explicit next-stage start checklist
- Stage boundary allowlist snapshot (read/edit)
- Boundary compliance report (out-of-scope reads/edits and approvals)

Downstream usage:
- Stage 7 starts from this handoff.

### Stage 7: <Stage 7 Name>

Goal:
- <Goal statement>

Codebase context:
1. Primary projects in scope: <project list>
2. Key files expected first: <file list>
3. Boundary constraints: <constraints>
4. Required validation/tests before handoff: <tests>
5. Allowed read scope: <project/folder allowlist>
6. Allowed edit scope: <project/folder allowlist>

Primary output handoff document:
- plans/active/handovers/<WORKSTREAM>_07_<STAGE>_HANDOFF.md

Required handoff contents:
- Scope completed
- Files changed
- Contract/interface impact
- Validation commands executed
- Test results
- Behavioral notes
- Known issues/risks
- Explicit next-stage start checklist
- Stage boundary allowlist snapshot (read/edit)
- Boundary compliance report (out-of-scope reads/edits and approvals)

Downstream usage:
- Final closeout starts from this handoff.

## Risk Register

1. <Risk and mitigation>
2. <Risk and mitigation>
3. <Risk and mitigation>

## Validation Gates

1. <Build command>
2. <Core tests>
3. <Focused regression tests>
4. <Contract guardrails if relevant>

## MVP/Completion Acceptance Criteria

1. <Criterion>
2. <Criterion>
3. <Criterion>

## Final Closeout Checklist

1. All locked stages marked complete.
2. All stage handoff documents exist and are status-complete.
3. Validation gate outcomes recorded.
4. Deferred items listed as non-blocking follow-up.
5. Main plan updated with final status and date.
6. Archive main plan and all related stage handoffs together as one unit.
