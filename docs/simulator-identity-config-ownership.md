# Simulator Identity Config Ownership

This document clarifies which config file owns which responsibility for simulator identity behavior.

## Client Transport Selection (Simulator)

- File: `Storyboard.Simulator/Config/identity-client.settings.json`
- Ownership: simulator caller side.
- Purpose: selects how the simulator calls identity APIs.
- Current modes:
  - `inprocess` (implemented)
  - `remote` (reserved; not implemented yet)

## Runtime Provider Selection (GameEngine)

- File: `Storyboard.GameEngine/Config/runtime-identity.provider.json`
- Ownership: runtime/provider side.
- Purpose: selects runtime identity provider implementation and provider-specific runtime options.

## Runtime Principal Store (GameEngine)

- File: `Storyboard.GameEngine/Config/runtime-identity.users.json`
- Ownership: runtime/provider side.
- Purpose: JSON-backed stand-in principal store consumed by runtime in-process provider.

## Persisted Simulator Defaults (AppData)

- File: `%AppData%/StoryboardSimulator/authentication.settings.json`
- Ownership: local simulator user preferences.
- Purpose: stores default username/password and auto-sign-in preference for streamlined local runs.

## Boundary Rule

- Simulator transport config must not include runtime provider internals like JSON user-store paths.
- Runtime provider config must own runtime provider data source details.
