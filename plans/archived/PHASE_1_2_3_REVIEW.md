# Phase 1-3 Review Packet

Last updated: 2026-06-22

## Scope Reviewed
- Phase 1: Native cleanup and data separation.
- Phase 2: Clean export v1 build.
- Phase 3: Validation and adoption readiness.

## Completion Summary
## Phase 1
- Completed:
  - Native/application-state separation for UI state via project state sidecar.
  - Legacy write cleanup for inline area `links`/`rooms`.
  - Controlled fixture migration.
  - Backward-read clean break (legacy fallbacks removed) per decision.

- Key outcome:
  - Native user project data and native application state data are now separated in persistence path.

## Phase 2
- Completed:
  - Dedicated clean export v1 DTO contract.
  - Export generator producing:
    - `<name>.sbe.clean.json`
    - `<name>.sbe.clean.navigation.json`
    - `<name>.sbe.clean.rooms/*.clean.room.json`
  - `schemaVersion` included in all clean export artifacts.
  - Deterministic ordering and omission policy implemented.

- Key outcome:
  - Stable clean export contract path exists and is independent of native save/load internals.

## Phase 3
- Completed:
  - Schema-shape validation tests.
  - Determinism tests.
  - Reference-integrity tests.
  - Snapshot baseline tests (single-room fixture).
  - Regression tests for exclusion of native application state in clean export.
  - Consumer-facing clean export documentation and versioning/change policy.

- Key outcome:
  - Contract behavior is tested and documented for external collaboration.

## Test Status
- Latest run:
  - `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`
- Result:
  - Total: 17
  - Passed: 17
  - Failed: 0

## Snapshot Baselines
Approved snapshot files:
- `StoryboardDesigner.App.Tests/Snapshots/CleanExportV1/single-room/single-room.sbe.clean.json`
- `StoryboardDesigner.App.Tests/Snapshots/CleanExportV1/single-room/single-room.sbe.clean.navigation.json`
- `StoryboardDesigner.App.Tests/Snapshots/CleanExportV1/single-room/single-room.sbe.clean.rooms/11111111111111111111111111111111.clean.room.json`

Snapshot update mode:
- Set `UPDATE_CLEAN_EXPORT_SNAPSHOTS=1` when intentionally refreshing baselines.

## Open Risks / Follow-ups (Pre-Phase 4)
1. Object `commands` remains retained in native user project data by current decision; revisit only during clean export contract evolution.
2. `roomPlacements` remains in native user project data by current decision; revisit if future collaboration needs a stricter runtime-only split.
3. Consider adding one more snapshot fixture beyond single-room (for example Birmingham) if you want broader baseline sensitivity.

## Review Gate Status
- Phase 1 exit criteria: Met.
- Phase 2 exit criteria: Met.
- Phase 3 exit criteria: Met.
- Team review completed and decisions recorded: Pending explicit sign-off.
- Open risks/issues list created for future custom transforms: This document includes initial list; expand during Phase 4 kickoff if approved.
