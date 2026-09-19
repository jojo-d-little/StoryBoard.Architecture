

For Designer-time source assets, see `docs/asset-source-root-configuration.md`.


- Name: STORYBOARD_DEVELOPMENT_ROOT
- Purpose: General root for the development path on disk i.e. C:\work\GIT\

- Name: STORYBOARD_SAMPLE_PROJECTS_ROOT
- Purpose: Root for the sample projects path on disk i.e. C:\work\GIT\StoryBoard.SampleProjects\Samples\

- Name: STORYBOARD_WEBPORTAL_ROOT
- Purpose: Tell the GameHost where to serve the portal code from i.e. C:\work\GIT\Storyboard\Storyboard.WebPortal\dist

- Name: STORYBOARD_ASSET_SOURCE_ROOT
- Purpose: Tell Designer where to source primary authoring assets, for example `C:\work\GIT\StoryBoard.SampleProjects\SourceAssets\`.

- Name: STORYBOARD_ASSET_SOURCE_ROOT_<NAME>
- Purpose: Additional named authoring asset roots. Persist references using `%STORYBOARD_ASSET_SOURCE_ROOT_<NAME>%/...`.

The retired `ASSETROOT:/...` format is not supported. Restart Designer after
changing these values; a Windows reboot is not required.

