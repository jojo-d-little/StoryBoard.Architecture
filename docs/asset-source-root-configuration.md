# Asset Source Root Configuration

The Designer resolves authored source assets through environment variables:

- Name: STORYBOARD_ASSET_SOURCE_ROOT
- Purpose: Primary authoring asset root.
- Additional roots: `STORYBOARD_ASSET_SOURCE_ROOT_<NAME>`.

## Canonical Path Format

Persist authored references using percent-delimited environment-variable tokens:

- `%STORYBOARD_ASSET_SOURCE_ROOT%/FormalImages/Victorian/HotelRoom/VicHotel_DoorClosed.png`
- `%STORYBOARD_ASSET_SOURCE_ROOT%/PlaceHolderSounds/CABINET DOOR OPEN.wav`
- `%STORYBOARD_ASSET_SOURCE_ROOT_ASSETLIB%/Sounds/example.wav`

At design time, these resolve to the configured physical roots. Runtime exports
continue to use relative `assets/...` paths and do not depend on authoring roots.

Root names after `STORYBOARD_ASSET_SOURCE_ROOT_` must begin with an uppercase
letter and contain only uppercase letters, digits, and underscores.

## Configure On Windows

PowerShell (persist for future sessions):

```powershell
setx STORYBOARD_ASSET_SOURCE_ROOT "C:\work\HobbyStuff\storyboarding"
```

PowerShell (current session only):

```powershell
$env:STORYBOARD_ASSET_SOURCE_ROOT = "C:\work\HobbyStuff\storyboarding"
```

After changing with `setx`, restart Designer. Designer refreshes missing user
environment variables from the current user's environment settings at startup,
so a Windows reboot or Explorer restart is not required.

## Migration Guidance

- `ASSETROOT:/...` is retired and is not supported.
- Migrate older projects before opening, editing, or publishing them in Designer.
- Existing absolute paths remain readable, but new in-root authored paths are persisted using the canonical `%STORYBOARD_ASSET_SOURCE_ROOT...%/...` form.
- This normalization applies to image paths, sound asset refs, and game preview image paths saved from the authoring dialogs.
- Published runtime/export assets remain generated output.
- Publish writes `assets/assets-manifest.json` with exported path, source paths, SHA-256, and file size metadata.
