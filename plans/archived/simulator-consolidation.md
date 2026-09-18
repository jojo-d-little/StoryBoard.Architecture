# Simulator Consolidation Plan

## Objective
Retire the embedded lightweight simulator inside the designer and replace it with an external-simulator workflow that is one-click, replay-aware, and optimized for repeated producer review loops, while keeping running-instance ownership inside the simulator host.

## Desired User Workflow
1. User edits content in Designer.
2. User opens Simulator Setup (under Tools) and optionally sets:
   - Replay file path
   - Replay speed
   - Simulator executable path
3. User clicks Run Simulator (new top-level menu).
4. Designer performs:
   - Save project
   - Run validation pass
   - If validation issues exist, show compact summary toast and require explicit continue confirmation
   - Export clean files
   - Launch external simulator executable
   - Pass command-line parameters:
     - Project clean file path
     - Replay file path (if configured)
     - Replay speed (if configured)
5. Simulator decides if this is:
   - A new instance for a different project (allowed), or
   - A duplicate launch for the same project (show user choice)
6. On duplicate launch for same project, simulator shows a user prompt:
   - Switch to running simulator for that project
   - Restart simulator clean for that project
7. Simulator opens directly to current project state and optionally auto-replays commands to a deep gameplay point.
8. User Alt+Tabs between Designer and Simulator repeatedly with minimal overhead.

## Scope
### In Scope
- Remove embedded lightweight game simulator UI/workflow from Designer shell.
- Add one top-level menu action: Run Simulator.
- Add one Tools menu action: Simulator Setup.
- Implement orchestration sequence: Save -> Export Clean -> Launch Simulator.
- Pass current clean project path into simulator process startup automatically.
- Add optional replay startup parameters:
  - Replay file path
  - Replay speed
- Allow simulator executable path override from Simulator Setup.
- Persist replay startup settings per project in the primary project file.
- Persist simulator executable path in Designer app config (machine-local), not per project.
- Designer always attempts launch and does not own duplicate-instance arbitration logic.
- Simulator enforces single-instance per project path (not global single-instance across all projects).
- Simulator duplicate-launch UX for same project includes two options:
  - Switch to running instance
  - Restart clean
- Surface clear success/failure status in Designer output/notifications.

### Out of Scope
- Deep simulator feature changes unrelated to startup handoff.
- Runtime contract redesign (unless needed for startup argument handling).
- Complex multi-instance/session broker features beyond per-project instance arbitration.

## Proposed Phases
### Phase 1: UX and Command Wiring
- Add new menu items in Designer shell:
  - Top-level menu: Run Simulator
  - Under Tools: Simulator Setup
- Bind both commands on main window viewmodel.
- Hide/remove embedded simulator tab and related menu/actions in Designer.

### Phase 2: Mixed-Persistence Simulator Setup
- Add a Simulator Setup dialog with:
  - Replay file path picker/input
  - Replay speed numeric input
  - Simulator executable path picker/input
- Store replay file path and replay speed in the primary project file.
- Store simulator executable path in Designer app config.
- Load settings automatically with project open.

### Phase 3: Launch Orchestration Service
- Add a Designer service responsible for:
  - Verifying project is saved (or forcing save path workflow)
  - Running clean export
  - Resolving simulator executable path
    - Prefer Designer app-configured executable path when provided
    - Fallback to packaged default when empty
  - Building startup arguments:
    - Project path argument (required)
    - Replay path argument (optional)
    - Replay speed argument (optional)
  - Starting simulator process
- Return structured launch result (success, diagnostics, launched path).

### Phase 4: Simulator Startup Argument Support
- Ensure simulator accepts project clean file path argument and initializes session from it.
- Ensure simulator accepts replay file argument and auto-starts replay when provided.
- Ensure simulator accepts replay speed argument.
- Implement simulator-owned per-project single-instance arbitration:
  - Allow concurrent simulator instances for different projects.
  - Detect second launch for same normalized project path.
  - On duplicate for same project, show prompt with:
    - Switch to running instance
    - Restart clean
- Ensure switch action brings existing instance to foreground.
- Ensure restart action terminates existing same-project instance and relaunches clean.
- Replay speed semantics:
  - 1.0 = playback timing faithful to recorded command-entry timing
  - 2.0 = double-speed replay
  - higher/lower positive values scale timing proportionally
- If invalid path, show actionable error in simulator and exit gracefully or remain idle.

### Phase 5: Guardrails and Regression Coverage
- Add tests for Designer launch orchestration behavior.
- Add tests for mixed-persistence simulator setup (project replay settings + app-config simulator executable path).
- Add tests for argument parsing/startup/replay-speed semantics in Simulator.
- Add tests for per-project duplicate-launch behavior:
  - same project duplicate prompts switch/restart
  - different project launch allows additional instance
- Add architecture guardrail tests to ensure Designer does not absorb runtime host responsibilities.

### Phase 6: Cleanup
- Remove dead code paths, commands, and views related to embedded lightweight simulator.
- Update docs and operator guidance.

### Phase 7: Mandatory Post-Implementation Review
- Conduct a mandatory architecture and usage audit after implementation stabilizes.
- Verify whether ProjectModelRuntimeSnapshotMapper is still required in the external-simulator-first workflow.
- If no longer used, open a scoped removal/refactor follow-up with tests and contract verification.
- If still required, document why it remains in the final implementation notes.

## Acceptance Criteria
- Embedded lightweight simulator entry points in Designer are removed.
- Run Simulator exists as a top-level menu and is discoverable.
- Simulator Setup exists under Tools and is discoverable.
- Run Simulator performs one-click save/export/launch for current project.
- Simulator receives and uses project path argument.
- If configured, simulator receives replay path and replay speed and auto-runs replay.
- Replay speed 1.0 matches recorded command timing; 2.0 is double speed.
- Simulator executable path is configurable from Simulator Setup and persisted in Designer app config.
- Duplicate launch for same project shows switch/restart choice from simulator UX.
- Different projects can run in parallel simulator instances.
- Errors are clearly reported when save/export/launch fails.
- Existing replay and runtime regression gates remain green.
- Post-implementation review is completed and explicitly records whether ProjectModelRuntimeSnapshotMapper remains required.

## Risks and Mitigations
- Process launch path issues on different dev/user setups:
  - Mitigation: configurable simulator executable path with packaged default fallback.
- Duplicate-launch arbitration complexity for restart flow:
  - Mitigation: define deterministic IPC contract and add integration coverage for switch/restart outcomes.
- Project-path normalization mismatch (relative vs absolute, casing, symlinks):
  - Mitigation: normalize/canonicalize path before instance-key computation.
- Replay file path drift when projects move folders:
  - Mitigation: support project-relative storage and path validation in setup UI.
- User confusion during transition:
  - Mitigation: clear release note and menu labeling.

## Design Questions To Lock Off
1. Menu wording and placement: should Run Simulator be top-level, and Simulator Setup live under Tools?
2. Unsaved project behavior: when Run Simulator is clicked and the project has never been saved, should Designer auto-save directly from known project folder/name context (no Save As)?
3. Duplicate launch prompt copy: what exact title/body text should simulator show when same-project instance is already running?
4. Duplicate launch default action: should the default focused button be Switch, Restart Clean, or Cancel?
5. Remember-choice policy: should duplicate-launch prompt include a remember-my-choice option, and if yes, where should that preference be stored?
6. Restart execution model: on Restart Clean for same project, should existing process self-restart in-place or should existing process exit while new process continues startup?
7. Replay restart semantics: on Restart Clean, should replay always restart from beginning using incoming args, or should current replay/session state ever be preserved?
8. Project identity key: what normalization rules define same project (absolute path, case normalization, long-path normalization, symlink/shortcut resolution)?
9. Simulator executable path validation: in Simulator Setup, what validations are required (exists, file type, launch probe), and when are they enforced (on save vs on run)?
10. Simulator executable path resolution precedence: final order between Designer app-config override, environment/config fallback, and packaged default.
11. Mixed-persistence setup storage format: project-file schema fields for replay values and app-config field for simulator executable path.
12. Run failure UX: for save/export/launch failures, what is the exact notification strategy (toast only, modal+toast, output pane logging), and what minimum diagnostics must be shown?
13. Launch completion mode: should Run Simulator wait synchronously for export and process start confirmation, or use async queue/background flow?
14. Transition strategy: remove embedded simulator in one cutover or keep a temporary fallback period behind a feature flag/menu toggle?

## Locked Decisions
1. Menu placement and frequency optimization:
  - Run Simulator is the only new top-level menu action.
  - Simulator Setup is placed under Tools because setup is expected to be infrequent.
2. Unsaved project handling for Run Simulator:
  - If project folder/name are already known (new-project workflow), Designer auto-saves directly with no Save As prompt.
  - Run Simulator then continues with clean export and launch.
3. Same-project duplicate-launch prompt copy:
  - Title: Simulator Already Running
  - Body: This project is already running in another simulator window. Do you want to switch to that window or restart the simulator clean for this project?
  - Buttons: Switch, Restart Clean, Cancel
4. Same-project duplicate-launch default action:
  - Default focused action: Restart Clean
5. Remember-choice policy:
  - No remember-my-choice option.
  - Always ask on same-project duplicate launch.
6. Restart execution model:
  - Existing simulator process exits.
  - Newly launched process continues startup.
  - Chosen as preferred lower-complexity implementation path.
7. Replay restart semantics on Restart Clean:
  - Always restart replay from the beginning using incoming run arguments.
  - Do not preserve current replay/session progression.
8. Project identity key for same-project duplicate detection:
  - Use normalized absolute path matching.
  - Apply case-insensitive full-path normalization.
9. Simulator executable path validation policy:
  - Validate on setup save for obvious issues (for example: missing file or invalid extension).
  - Validate again on run as final guard before launch.
10. Simulator executable resolution precedence:
  - Designer app-config override first.
  - Packaged default second.
  - No environment/config intermediary fallback layer.
  - If path is missing or invalid at run time, show a polite message directing user to Tools -> Simulator Setup.
11. Mixed-persistence simulator setup storage:
  - Store replay file path and replay speed in the primary project file.
  - Store simulator executable path in Designer app config.
  - Do not introduce a dedicated simulator sidecar file for this feature.
12. Run failure and validation UX:
  - Use toast-style UX for run-time workflow messaging (no modal validation report dialog in this path).
  - Run Simulator performs a validation pass before export/launch.
  - If validation issues exist, show a compact validation summary in the toast.
  - User can acknowledge and continue launch despite validation issues.
  - Keep this flow lightweight for rapid iterate-and-preview loops.
13. Launch completion mode:
  - Run Simulator executes asynchronously and does not block Designer UI.
  - Surface progress and outcome via toast updates.
  - Preserve producer ability to alt-tab between Designer and Simulator during run pipeline execution.
14. Transition strategy:
  - Remove embedded simulator in a single cutover release.
  - Do not keep a temporary fallback menu toggle/feature flag.

## Phased Sliced Delivery Plan

### Slice 0: Contract and Wiring Skeleton
- Goal: establish command/service seams with no behavior change.
- Scope:
  - Introduce/confirm Run Simulator command pathway in Designer viewmodel/service layer.
  - Introduce/confirm Simulator Setup command pathway under Tools.
  - Add internal launch request/result DTOs to keep responsibilities explicit.
- Exit criteria:
  - App builds.
  - No user-visible behavior regression.
  - Existing smoke and regression tests remain green.

### Slice 1: Menu Placement and UX Surface
- Goal: deliver agreed menu structure only.
- Scope:
  - Add top-level Run Simulator menu.
  - Add Tools -> Simulator Setup menu.
  - Remove/hide embedded simulator menu entry points.
- Exit criteria:
  - Menu placement matches locked Decision 1.
  - No launch behavior change yet.

### Slice 2: Simulator Setup Persistence (Project + App Config)
- Goal: persist and reload simulator setup values across project content and app config.
- Scope:
  - Add project model fields for replay path and replay speed.
  - Add serialization/deserialization support for replay values in primary project file flow.
  - Add app-config persistence for simulator executable path.
  - Setup dialog reads/writes these values.
- Exit criteria:
  - Replay values round-trip through project save/load.
  - Simulator executable path round-trips via app config load/save.
  - No sidecar introduced for simulator setup.

### Slice 3: Run Pipeline Orchestration (Non-Blocking)
- Goal: implement async Run Simulator pipeline in Designer.
- Scope:
  - Auto-save with known project folder/name (no Save As in this path).
  - Run validation pass.
  - Show compact validation-summary toast with continue option.
  - Export clean files.
  - Build launch arguments and start simulator process.
  - Keep Designer UI responsive throughout.
- Exit criteria:
  - Locked Decisions 2, 12, and 13 are satisfied.
  - Failure guidance for invalid/missing simulator path directs to Tools -> Simulator Setup.

### Slice 4: Simulator Startup Args and Replay Bootstrap
- Goal: simulator consumes startup args and applies replay configuration.
- Scope:
  - Parse project path argument and initialize session.
  - Parse replay path and replay speed arguments.
  - Apply replay speed semantics (1.0 baseline, 2.0 double-speed, positive scaling).
- Exit criteria:
  - Startup from Designer run path opens correct project.
  - Replay starts correctly when configured.

### Slice 5: Per-Project Duplicate Launch Arbitration
- Goal: simulator-owned same-project arbitration with agreed UX.
- Scope:
  - Compute normalized absolute project key for duplicate detection.
  - Allow multi-instance for different projects.
  - On same-project duplicate: show agreed prompt text/buttons.
  - Default action Restart Clean.
  - No remember-choice option.
  - Restart model: existing instance exits, new instance continues.
- Exit criteria:
  - Locked Decisions 3 through 8 are satisfied.
  - Switch and Restart Clean both behave deterministically.

### Slice 6: Test and Guardrail Pass
- Goal: lock behavior with focused regression coverage.
- Scope:
  - Add/update Designer tests for setup persistence and run orchestration.
  - Add/update Simulator tests for arg parsing, replay speed, and duplicate arbitration.
  - Keep architecture separation guardrails green.
- Exit criteria:
  - Validation baseline commands pass.
  - Focused runtime regression gate passes.

### Slice 7: Cleanup and One-Cut Migration
- Goal: complete one-cut migration and remove legacy paths.
- Scope:
  - Remove embedded simulator UI/workflow code.
  - Remove dead adapters/services no longer needed.
  - Update docs and operator notes for new workflow.
- Exit criteria:
  - Locked Decision 14 satisfied.
  - No hidden fallback toggle remains.

### Slice 8: Mandatory Post-Implementation Review
- Goal: enforce final architecture usage audit.
- Scope:
  - Trace usage of ProjectModelRuntimeSnapshotMapper and related mapping paths.
  - Decide retain vs remove based on actual post-move usage.
  - If removal candidate, open scoped follow-up with tests and contract checks.
- Exit criteria:
  - Review artifact produced.
  - Explicit keep/remove rationale documented.

## Validation Baseline For Implementation
- dotnet build .\StoryboardDesigner.slnx
- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"
- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj

## Validation Execution Strategy (Heavier Safety Mode)
- Goal: reduce runtime-boundary regression risk during simulator consolidation.
- Per-slice minimum gate (after each meaningful code slice):
  - dotnet build .\StoryboardDesigner.slnx
  - dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
- Runtime-impact slice gate (required for slices that touch launch orchestration, startup arg parsing, replay bootstrap, duplicate-launch arbitration, or runtime/shared boundaries):
  - dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"
- Pre-merge/full-readiness gate:
  - dotnet build .\StoryboardDesigner.slnx
  - dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
  - dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"
  - dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
- Optional nightly/deep safety run (recommended while this initiative is active):
  - dotnet test .\StoryboardDesigner.App.SmokeTests\StoryboardDesigner.App.SmokeTests.csproj
