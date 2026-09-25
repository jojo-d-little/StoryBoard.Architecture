# F02: First-Class Diagnostics And Trace Workflow

Status: Complete; follow-up refinements recorded

## Purpose

This packet is the feature-level execution record. Its authoritative dependency order, pass catalog, and acceptance criteria are maintained in `SIMULATOR_RETIREMENT_READINESS_EXECUTION_ROADMAP.md` under F02.

## Ordered Pass Sequence And Handoffs

F02 keeps the long-range three-log merge goal visible, but the MVP delivers a useful, bounded WebPortal client trace first. It does not implement an automated merger or aggregate backend logs.

| Order | Pass | Area profile | Pass handoff status |
| --- | --- | --- | --- |
| 1 | F02.1 Shared correlation review | `01_CONTRACTS_SHARED_PROFILE` | Complete: `F02.1_SHARED_CORRELATION_REVIEW_HANDOFF.md` |
| 2 | F02.2 Portal trace inventory and event model | `04_WEBPORTAL_PROFILE` | Complete: `F02.2_PORTAL_TRACE_INVENTORY_HANDOFF.md` |
| 3 | F02.3 Portal Diagnostics workspace | `04_WEBPORTAL_PROFILE` | Complete: `F02.3_PORTAL_DIAGNOSTICS_WORKSPACE_HANDOFF.md` |
| 4 | F02.4 Portal trace export | `04_WEBPORTAL_PROFILE` | Complete: `F02.4_PORTAL_TRACE_EXPORT_HANDOFF.md` |
| 5 | F02.5 Simulator diagnostics audit | `05_SIMULATOR_AUDIT_PROFILE` | Not needed for F02 closure; replacement workflow accepted and audit comparison deferred |
| 6 | F02.6 Diagnostics vertical integration | `07_INTEGRATION_CLOSEOUT_PROFILE` | Complete: `F02.6_DIAGNOSTICS_VERTICAL_INTEGRATION_HANDOFF.md` |

Each pass may iterate naturally within its owning area. A separate remediation pass is created only for an unresolved gap that remains after the active owner pass or integration checkpoint.

## MVP Boundary

1. Portal captures and exports its own client trace; it does not collect or aggregate GameHost/GameEngine logs.
2. Existing correlation fields are standardized and propagated where needed; no new shared contract is assumed until a real boundary gap is demonstrated.
3. The future three-log merge utility remains a design target. F02 preserves the metadata it will need but does not build the merger.
4. Portal trace output must be readable to a developer and structured enough for later alignment and tooling.

## Feature Entry Gate

1. Review the roadmap's ordered area-pass matrix before scheduling the first pass.
2. Confirm decisions, contract approvals, and prerequisite pass evidence.
3. Create the selected pass handoff from the local pass-handoff template and name the next pass.

## Feature Closure Gate

1. Every required pass is complete, explicitly not needed, or has an approved follow-up.
2. All parity dispositions, validation evidence, and pass-handoff records agree.
3. The roadmap feature status is updated through the Integration / Closeout profile.

## Working Method

F02 implementation and validation may discover and fix issues inline within F02.1-F02.5. Record those fixes and revalidation in the owning handoff; do not manufacture separate phases for already-resolved issues.

## Feature Closure Record

1. F02.1-F02.4 and F02.6 are complete. F02.5 is explicitly waived for this readiness feature because the Portal workflow is the replacement diagnostic baseline; no laborious Simulator parity comparison is required to establish the Portal foundation.
2. The completed MVP provides mode-driven capture, JSON-backed trace profiles, source scopes, display filters, bounded retention, low-volume delta heartbeat behavior, readable/NDJSON export, and command/asset/session-echo semantic metadata.
3. Validation evidence includes manual Dev Simulator use of start/stop, console visibility, profile/scope controls, trace export, command capture, asset identity, and host echo visibility; the Portal suite passes 27 files/217 tests and the production build succeeds.
4. Deferred follow-ups are trace-volume tuning, additional semantic event coverage as real diagnosis scenarios arise, and any future beta/production ring-buffer policy. These are refinements, not F02 closure blockers.
