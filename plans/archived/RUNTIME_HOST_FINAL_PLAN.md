# Runtime Host Final Plan

## Objective
Make the Designer a pure authoring/export tool and move runtime game initialization + execution to Shared, so any host (simulator, game engine, external runner) can consume clean exported JSON and drive runtime through a single manager abstraction.

## Status
Completed:
- F1 Shared runtime load package and seams.
- F2 Manager core decoupled from ProjectModel internals.
- F3 Runtime command request path unified on RuntimeCommandProcessingRequest.
- F4 Concrete manager moved to Shared and wired via app composition.
- F5 Host fixture coverage proving shared manager playback path.
- F6 Boundary hardening with additional guardrails and removal of session model compatibility shim.

## Target End State
1. Designer responsibilities:
- Author content.
- Validate content.
- Export clean runtime JSON only.
- No responsibility for runtime execution architecture.

2. Shared responsibilities:
- Read clean runtime JSON (or runtime bootstrap inputs).
- Build in-memory runtime scope tree.
- Maintain runtime state/session (variables, navigation, containers, object movement).
- Process commands via runtime contracts.
- Expose one canonical manager interface for hosts.

3. Host responsibilities (simulator/game engine/other):
- Provide input command stream and output display/logging.
- Call shared manager APIs only.
- No dependency on Designer model graph (ProjectModel/IScopedAwareNode).

## Current Gap Summary
Already done:
- Runtime session/scope/property state moved to Shared.
- A legacy runtime-manager interface existed and was previously canonical in app usage.

Still blocking full host-agnostic runtime manager move:
- Concrete manager still app-local and uses Designer model types.
- Loader seam still returns ProjectModel.
- Command processor path still goes through app-specific bridge request that includes ProjectModel.

## Architecture Decisions (Final)
1. Canonical manager interface:
- Keep one host-facing runtime manager API.
- Do not reintroduce a parallel manager interface.

2. Runtime data contract:
- Runtime manager operates on runtime-only data package, not ProjectModel.
- Input source is clean export/clean bootstrap derived runtime package.

3. Adapter boundary:
- Any Designer-specific mapping stays in Designer app, not in manager core.
- Shared manager must compile and execute without StoryboardDesigner.App.Models references.

## Execution Plan

### Phase F1 - Introduce Shared Runtime Load Package
Deliverables:
- Add a Shared runtime load package (example: RuntimeLoadedGame) containing:
  - RuntimeGameWorldSnapshot
  - Runtime action catalog / action target graph needed by command execution
  - Optional host metadata used for diagnostics
- Add Shared loader contract that returns RuntimeLoadedGame from clean export artifacts.

Acceptance:
- Shared manager and command pipeline can be constructed from runtime-only package.
- No ProjectModel required by manager constructor/state.

### Phase F2 - Remove ProjectModel from Manager Core
Deliverables:
- Replace app-local manager internals currently storing ProjectModel with RuntimeLoadedGame/runtime contracts.
- Update reference-token/context building to read from runtime session and runtime action graph only.

Acceptance:
- Manager compiles without StoryboardDesigner.App.Models usage.
- Runtime processing works with same behavior for current focused tests.

### Phase F3 - Runtime Command Pipeline Purification
Deliverables:
- Remove AppRuntimeCommandProcessingRequest from manager path.
- Use RuntimeCommandProcessingRequest only.
- Move any remaining model-dependent scope/action resolution logic behind runtime-only lookup contracts.

Acceptance:
- IRuntimeCommandProcessorService.Process accepts runtime request path only in manager flow.
- No IScopedAwareNode/ScopeNodeKind in manager execution path.

### Phase F4 - Move Concrete Manager to Shared
Deliverables:
- Move concrete GameManager implementation into Storyboard.Shared (rename to RuntimeGameManager if desired).
- Keep app thin composition root that instantiates shared manager with shared loader/processor.

Acceptance:
- Designer app references shared concrete manager.
- External host can instantiate same shared concrete manager with no Designer model dependency.

### Phase F5 - Host Surface and Packaging
Deliverables:
- Add small host bootstrap API in Shared:
  - LoadFromProjectPath(string) or LoadFromRuntimeArtifacts(...)
  - ProcessCommand(...)
  - Session accessors/events
- Add minimal sample host in tests or sample folder proving pure Shared runtime usage.

Acceptance:
- Sample host runs command playback without StoryboardDesigner.App assembly references (except DTO contracts if still shared namespace).

### Phase F6 - Designer Boundary Hardening
Deliverables:
- Ensure Designer only calls export + shared manager APIs.
- Remove/restrict any remaining direct model-coupled runtime execution code.
- Add architecture guardrail tests to prevent regressions.

Acceptance:
- Guardrail test fails if Shared manager references ProjectModel or IScopedAwareNode.
- Focused regression suite remains green.

## Proposed Guardrails
1. Shared manager assembly checks:
- Forbidden type references:
  - StoryboardDesigner.App.Models.ProjectModel
  - StoryboardDesigner.App.Models.IScopedAwareNode
  - ScopeNodeKind

2. Runtime pipeline checks:
- Manager command path may only use:
  - RuntimeCommandProcessingRequest
  - IRuntimeCommandWorldLookup
  - IRuntimeScopeMutationGateway
  - Runtime session/scope types from Shared

## Test and Validation Gates
Run at each phase:
- dotnet build .\StoryboardDesigner.slnx
- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"

Add final host-agnostic gate:
- A test fixture that instantiates shared concrete manager directly from clean runtime export and runs command playback.

## Cutover Strategy
1. Parallel path:
- Introduce shared concrete manager while app-local manager forwards to it temporarily.
2. Swap:
- Update app composition root to instantiate shared manager directly.
3. Remove:
- Delete app-local manager implementation and any obsolete bridge-only manager wrappers.

## Definition of Done
1. There is one manager abstraction for host-facing runtime operations.
2. Concrete manager implementation lives in Shared.
3. Manager and runtime execution path have zero Designer model dependencies.
4. Designer is design/export only.
5. External host can run game entirely via clean export + Shared DLL.
6. Regression and guardrail tests pass.
