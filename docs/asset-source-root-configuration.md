# Asset Source Root Configuration

This repository supports a logical asset source root via environment variable:

- Name: STORYBOARD_ASSET_SOURCE_ROOT
- Purpose: Resolves authored asset paths that use ASSETROOT:/... tokens.

## Path Token Format

Use portable authored references like:

- ASSETROOT:/FormalImages/Victorian/HotelRoom/VicHotel_DoorClosed.png
- ASSETROOT:/PlaceHolderSounds/CABINET DOOR OPEN.wav

At runtime/design time, these resolve to:

- <STORYBOARD_ASSET_SOURCE_ROOT>/FormalImages/Victorian/HotelRoom/VicHotel_DoorClosed.png
- <STORYBOARD_ASSET_SOURCE_ROOT>/PlaceHolderSounds/CABINET DOOR OPEN.wav

## Configure On Windows

PowerShell (persist for future sessions):

```powershell
setx STORYBOARD_ASSET_SOURCE_ROOT "C:\work\HobbyStuff\storyboarding"
```

PowerShell (current session only):

```powershell
$env:STORYBOARD_ASSET_SOURCE_ROOT = "C:\work\HobbyStuff\storyboarding"
```

After changing with `setx`, restart shells and apps that should read the new value.

## Migration Guidance

- Existing absolute paths continue to work for backward compatibility.
- Prefer ASSETROOT:/... for authored data that should stay machine-portable.
- Designer authoring now auto-normalizes persisted paths to ASSETROOT:/... when the selected or entered absolute path resolves under STORYBOARD_ASSET_SOURCE_ROOT.
- This normalization applies to image paths, sound asset refs, and game preview image paths saved from the authoring dialogs.
- Published runtime/export assets remain generated output.
- Publish writes `assets/assets-manifest.json` with exported path, source paths, SHA-256, and file size metadata.
