

For Designer-time source assets, see `docs/asset-source-root-configuration.md`.


- Name: STORYBOARD_DEVELOPMENT_ROOT
- Purpose: General root for the development path on disk i.e. C:\work\GIT\

- Name: STORYBOARD_SAMPLE_PROJECTS_ROOT
- Purpose: Root for the sample projects path on disk i.e. C:\work\GIT\StoryBoard.SampleProjects\Samples\

- Name: STORYBOARD_WEBPORTAL_ROOT
- Purpose: Tell the GameHost where to serve the built WebPortal files from, for example `C:\work\GIT\StoryBoard.WebPortal\dist`.
- F01 development launch: The Designer supplies this value to the GameHost child process for each **Run Development WebPortal** launch. It is not normally necessary to define it as a permanent user or system variable; use it only as a machine-level fallback when a local Portal location must be overridden.

- Name: STORYBOARD_GAMEHOST_EXECUTABLE_PATH
- Purpose: Optional machine-level path to `Storyboard.GameHost.exe`, used by the Designer's **Run Development WebPortal** action when it cannot use its packaged or conventional local-development location.
- Example: `C:\work\GIT\StoryBoard.GameEngine\Storyboard.GameHost\bin\Debug\net8.0\Storyboard.GameHost.exe`
- Notes: This is an optional setup override. A configured Designer preference takes precedence.

- Name: STORYBOARD_RUNTIME_GAME_DISCOVERY_JSON_PATH
- Purpose: Tell a GameHost process which `runtime-game-registrations.json` catalog to use for runtime game discovery.
- F01 development launch: The Designer creates a temporary, one-entry catalog for the current project export and sets this value only in the environment of the GameHost process it starts. Developers should not set it permanently as part of workstation setup: doing so could redirect unrelated GameHost processes away from the normal game catalog.

- Name: STORYBOARD_ASSET_SOURCE_ROOT
- Purpose: Tell Designer where to source primary authoring assets, for example `C:\work\GIT\StoryBoard.SampleProjects\SourceAssets\`.

- Name: STORYBOARD_ASSET_SOURCE_ROOT_<NAME>
- Purpose: Additional named authoring asset roots. Persist references using `%STORYBOARD_ASSET_SOURCE_ROOT_<NAME>%/...`.

The retired `ASSETROOT:/...` format is not supported. Restart Designer after
changing these values; a Windows reboot is not required.

