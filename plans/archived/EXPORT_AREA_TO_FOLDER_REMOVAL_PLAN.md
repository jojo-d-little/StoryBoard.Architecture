# ExportAreaToFolder Removal

Status: Active
Owner: Copilot + User
Last Updated: 2026-07-28

## Goal
Remove the dead area-export pipeline safely, starting at or above `ExportAreaToFolder`, and continuing down through `RoomExportDto` / `GameObjectDto` once proven unused.

## Current Evidence (Pre-Removal)
1. `JsonExportService.ExportAreaToFolder(...)` exists and writes `map.json` + per-room `room.json` export artifacts.
2. Only one production call site exists in `MainWindowViewModel.ExportSelectedArea`.
3. No XAML binding was found for `ExportAreaCommand`.
4. No tests currently invoke this export command path.
5. No `map.json` artifacts are currently present in workspace samples.

## Safety Rules
1. Remove top-of-stack references first, then descend.
2. Compile and run targeted tests after each slice.
3. Do not mix this extraction with unrelated refactors.
4. If unknown callers appear, pause and reassess before deleting deeper DTOs.

## Phases

### Phase 0 - Baseline Capture
- [x] Capture grep evidence and file references for all symbols in this path.
- [x] Run build + baseline test gates.

Validation:
- `dotnet build .\StoryboardDesigner.slnx`
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`

### Phase 1 - Remove ViewModel Entry Point
- [x] Remove `ExportAreaCommand` command wiring.
- [x] Remove `ExportSelectedArea()` method.
- [x] Remove any status-path messaging specific to area export.

Validation:
- Build + focused app tests.

### Phase 2 - Remove Service Contract Surface
- [x] Remove `ExportAreaToFolder(...)` from `IJsonExportService`.
- [x] Remove implementation from `JsonExportService`.
- [x] Remove now-orphan `SerializeAreaMap(...)` if unused.

Validation:
- Build + focused app tests.

### Phase 3 - Remove Export DTO Layer
- [x] Remove `AreaMapExportDto` and `RoomLinkExportDto` if no references remain.
- [x] Remove `RoomExportDto` + nested DTOs including `GameObjectDto` if no references remain.
- [x] Remove helper mappers used only by this path (for example `ToRoomExportGameObjectDto`) once proven orphaned.

Validation:
- Build + full app test project.
- Runtime-focused filter gate.

### Phase 4 - Guardrail + Closeout
- [x] Add/update a structural test to prevent reintroduction of this dead path.
- [x] Final grep confirms zero references to removed symbols (excluding docs/plans and intentional guardrail assertions).
- [ ] Archive this plan when complete.

Validation:
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`

## Progress Log
- 2026-07-28: Plan created. Confirmed likely dormant path and no current `map.json` artifacts in repo.
- 2026-07-28: Completed phased removal from ViewModel entrypoint through service contract and export DTO layer; all validation gates green (`dotnet build`, full app tests, playback gate, runtime-focused filter). Added architecture guardrail test to prevent `ExportAreaToFolder` API/DTO reintroduction.

## Resume Checklist
1. Re-run symbol search for `ExportAreaToFolder`, `ExportAreaCommand`, `SerializeAreaMap`, `ToRoomExportGameObjectDto`, `RoomExportDto`.
2. Continue at the first incomplete phase checkbox.
3. Execute validation gate for the phase before proceeding.
