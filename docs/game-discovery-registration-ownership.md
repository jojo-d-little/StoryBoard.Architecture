# Game Discovery Registration Ownership

This document clarifies ownership and path semantics for discovery registration data.

## Registration Catalog (Host/Server Owned)

- File: `Storyboard.GameEngine/Config/runtime-game-registrations.json`
- Ownership: host/server registration catalog.
- Purpose: maps host-assigned game identities to runtime project bootstrap locations.

Registration fields are host-owned:

- `gameId`
- `gameKey`
- `runtimeProjectPath`
- `isEnabled`
- `tenantId`
- `orgId`

## Producer Metadata (Project Owned)

Producer-authored metadata should come from project-level authoring/export:

- `displayName`
- `summary`
- `previewImages`

These values are merged into discovery/detail responses with registration metadata.

## Runtime Project Path Resolution Rule

- If `runtimeProjectPath` is absolute, use it as-is.
- If `runtimeProjectPath` is relative, resolve it from host base directory (typically `AppContext.BaseDirectory`).

## Practical Caveat for Local Development

When running from `bin` output, host base directory is the output folder, not repository root. Relative paths that assume repo-root context may fail unless they traverse correctly from output location.
