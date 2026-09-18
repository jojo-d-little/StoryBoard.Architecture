# Clean Export v1

Last updated: 2026-06-22

## Purpose
This document defines the clean exported format produced for external collaboration.

The clean export is derived from native user project data and excludes native application state data.

## File Set
Given a native project file named `<ProjectName>.sbe.json`, clean export produces:

1. `<ProjectName>.sbe.clean.json`
2. `<ProjectName>.sbe.clean.navigation.json`
3. `<ProjectName>.sbe.clean.rooms/` containing one file per room:
   - `<roomIdN>.clean.room.json`

## Versioning
Each clean export file includes:

- `schemaVersion`: currently `"1.0"`

Contract changes require an explicit schema version decision.

## Contract Change Policy
Use these rules for future updates:

1. Non-breaking changes (keep `schemaVersion`):
- Adding optional fields that consumers can safely ignore.
- Tightening deterministic ordering without changing semantic meaning.
- Documentation clarifications only.

2. Breaking changes (bump major schema version):
- Removing existing fields.
- Renaming fields.
- Changing field meaning/type.
- Moving data between files in a way that breaks existing consumers.

3. Process requirements for any contract change:
- Update this document.
- Add or update tests covering the changed contract behavior.
- Include a release note describing consumer impact.

Current target during Phase 1-3:
- Keep contract at `1.0` unless a breaking change is explicitly approved.

## Contract Boundaries
Included:

- Authored gameplay content (hierarchy, rooms, objects, commands, actions, variables, scripts).
- Navigation links and room placements.
- Producer notes (currently retained in clean export v1).

Excluded:

- Native application/workspace state (for example UI selection state).

## Determinism Rules
The exporter applies deterministic ordering where practical:

- Hierarchy nodes sorted by name.
- Rooms sorted by id.
- Action, variable, script, and vocabulary lists sorted by stable keys.
- Link and placement lists sorted by ids and direction.

## Notes for Consumers
- Treat clean export as the external interface contract.
- Do not parse native authoring files as integration input.
- Validate `schemaVersion` before processing.

## Current Limitations
- Omit/default field policy is still being tightened in Phase 2/3.
- Additional profile-based transforms are deferred until after Phase 1-3 review gate.
