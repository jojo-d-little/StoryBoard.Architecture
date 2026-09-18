# StoryboardDesigner

StoryboardDesigner is a .NET 8 WPF authoring tool for story/game content, with runtime execution moved to shared contracts so multiple hosts can run the same game logic.

## Current Architecture

1. `StoryboardDesigner.App`
- Designer/authoring host (WPF MVVM).
- Owns editor UX and project authoring flows.

2. `Storyboard.Simulator`
- Standalone runtime simulator host (WPF MVVM).
- Depends on `Storyboard.Shared`, not on `StoryboardDesigner.App`.

3. `Storyboard.Shared`
- Runtime contracts, shared DTOs, runtime services/helpers, and manager/session pipeline seams.
- No WPF UI concerns.

4. `StoryboardDesigner.App.Tests`
- Regression, contract, and architecture-guardrail coverage.

## Data and Export Model

1. Native authored content data is stored in project files/sidecars used for authoring.
2. Native app/session state is stored separately (project state sidecar and app-level state where appropriate).
3. Runtime export is the external integration contract.

Runtime export v1 outputs:
1. `<ProjectName>.sbr.runtime.json`
2. `Area/<areaId>.runtime.json`
3. `Room/<roomId>.runtime.json`
4. `Planet/<planetId>.runtime.json`
5. `Country/<countryId>.runtime.json`
6. `GameObject/<objectId>.runtime.json`
7. `Procedure/<procedureId>.procedure.json` (when procedures are authored)

Runtime export root-scope note:
1. `RuntimeProjectDto` is the logical `Global` scope node and is intentionally stored at `<ProjectName>.sbr.runtime.json` as the bootstrap file.
2. A `Global/` folder layout should only be introduced alongside an explicit root manifest/index contract for loader discovery.

Contract details and change policy:
- `ENHANCEMENT_GUIDELINES.md`

Engineering policy source of truth:
- `ENHANCEMENT_GUIDELINES.md`

## Build, Test, Run

From repository root:

```powershell
dotnet restore .\StoryboardDesigner.slnx
dotnet build .\StoryboardDesigner.slnx
dotnet test .\Storyboard.GameClient.Tests\Storyboard.GameClient.Tests.csproj
dotnet test .\Storyboard.GameClient.Tests\Storyboard.GameClient.Tests.csproj --filter "FullyQualifiedName~GameHostProgram_StaticClient"
dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
dotnet run --project .\Storyboard.GameHost\Storyboard.GameHost.csproj
dotnet run --project .\StoryboardDesigner.App\StoryboardDesigner.App.csproj
dotnet run --project .\Storyboard.Simulator\Storyboard.Simulator.csproj
```

GameClient CI workflow:
1. [.github/workflows/gameclient-tests.yml](.github/workflows/gameclient-tests.yml)

Run simulator in thin mode against a local host:

```powershell
dotnet run --project .\Storyboard.Simulator\Storyboard.Simulator.csproj -- --mode thin --host-url http://127.0.0.1:5086
```

Focused runtime-boundary regression gate:

```powershell
dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"
```

## Host Interface Transport Maintenance

Host transport artifact generation now derives method-to-route bindings from host endpoint mapping sources.

Maintenance flow after changing host interface definitions:
1. Update corresponding endpoint mappings under Storyboard.GameHost/Endpoints when route/verb intent changes.
2. Run transport artifact verify:

```powershell
dotnet run --project .\Storyboard.TransportCodegen\Storyboard.TransportCodegen.csproj -- verify
```

3. If verify reports stale artifacts, regenerate:

```powershell
dotnet run --project .\Storyboard.TransportCodegen\Storyboard.TransportCodegen.csproj -- generate
```

4. Run transport guardrail tests:

```powershell
dotnet test .\Storyboard.TransportCodegen.Tests\Storyboard.TransportCodegen.Tests.csproj
```

Notes:
1. There is no separate manual host transport declaration file to update.
2. Verification fails fast when interface signatures and host endpoint mappings diverge.

## GameHost Packaging

Framework-dependent publish:

```powershell
dotnet publish .\Storyboard.GameHost\Storyboard.GameHost.csproj -c Release --self-contained false -o .\artifacts\gamehost\fdd
```

Self-contained publish examples:

```powershell
dotnet publish .\Storyboard.GameHost\Storyboard.GameHost.csproj -c Release -r win-x64 --self-contained true -p:PublishSingleFile=true -p:PublishTrimmed=false -o .\artifacts\gamehost\win-x64
dotnet publish .\Storyboard.GameHost\Storyboard.GameHost.csproj -c Release -r linux-x64 --self-contained true -p:PublishSingleFile=true -p:PublishTrimmed=false -o .\artifacts\gamehost\linux-x64
```

CI publish matrix workflow:
1. [.github/workflows/gamehost-publish-matrix.yml](.github/workflows/gamehost-publish-matrix.yml)

## Guidance Documents

1. Session entry instructions: `.github/copilot-instructions.md`
2. Engineering policy and guardrails: `ENHANCEMENT_GUIDELINES.md`
3. Canonical project terminology: `TERMINOLOGY.md`
4. Archived export planning/spec history: `plans/README.md`
5. Plan archive and index: `plans/README.md`
6. New action implementation guide: `NEW_ACTION_CREATION_COOKBOOK.md`

## Runtime Contract Schema Workflow

Runtime contract DTOs are schema-generated artifacts.

Source of truth:
1. JSON schema files in Storyboard.Shared.Contracts/RuntimeContracts/Schemas
2. Enum schemas in Storyboard.Shared.Contracts/RuntimeContracts/Schemas/Enums
3. Custom type schemas in Storyboard.Shared.Contracts/RuntimeContracts/Schemas/CustomTypes

Generation flow:
1. Edit schema(s).
2. Generate staging output.
3. Inspect staging diffs.
4. Lock accepted output into live contract folders.

Common commands (from repo root):
1. dotnet run --project .\Storyboard.SchemaCodegen\Storyboard.SchemaCodegen.csproj -- generate-contract-staging
2. dotnet run --project .\Storyboard.SchemaCodegen\Storyboard.SchemaCodegen.csproj -- generate-enum-staging
3. dotnet run --project .\Storyboard.SchemaCodegen\Storyboard.SchemaCodegen.csproj -- generate-custom-type-staging
4. dotnet run --project .\Storyboard.SchemaCodegen\Storyboard.SchemaCodegen.csproj -- lock-contract
5. dotnet run --project .\Storyboard.SchemaCodegen\Storyboard.SchemaCodegen.csproj -- lock-enum
6. dotnet run --project .\Storyboard.SchemaCodegen\Storyboard.SchemaCodegen.csproj -- lock-custom-type

### Schema-to-C# Property Controls

The generator supports schema extension keys for shaping emitted C# properties.

Supported property emission opt-out:
1. x-csharp-emit-property: false
2. x-csharp-emit: false

When either key is set to false on a property schema:
1. The property remains in JSON schema (validation and contract visibility).
2. The property is omitted from generated DTO C# output.

Example:

		"nameInGame": {
			"anyOf": [
				{ "type": "string" },
				{ "type": "null" }
			],
			"x-csharp-emit-property": false
		}
