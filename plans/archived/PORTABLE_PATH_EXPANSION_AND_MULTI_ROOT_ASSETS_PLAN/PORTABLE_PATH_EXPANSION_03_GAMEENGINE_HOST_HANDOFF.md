# Portable Path Expansion — Stage 03 GameEngine and Host Handoff

Status: In progress — code slice implemented; documentation and clean consumer rebuild pending  
Stage: 03 of 07  
Date: 2026-09-18  
Owner Session: Codex / StoryBoard.GameEngine

## Execution Checkout Metadata

- Coordination source: `StoryBoard.Architecture/plans/active/PORTABLE_PATH_EXPANSION_AND_MULTI_ROOT_ASSETS_PLAN.md`
- Execution repository: `StoryBoard.GameEngine`
- Execution document: `plans/PORTABLE_PATH_EXPANSION_03_GAMEENGINE_HOST_HANDOFF.md`
- Baseline plan revision: `7c4aa9b` (`better env support`), 2026-09-18.
- Required Foundation package: `Storyboard.Foundation 0.1.1`
- Package source/feed: configured `github` source `https://nuget.pkg.github.com/jojo-d-little/index.json`; resolved from the local NuGet cache during this session.
- Foundation release commit: `6e37aabf51feef3fbada1baf635a78763f3fa515` (recorded in the 0.1.1 nuspec).
- Return commit: Pending.
- Lifecycle status: Checked out in this workspace; not yet returned to Architecture.

## Scope Completed

Implemented the GameEngine/GameHost code slice for the approved configured-path boundaries:

1. Added direct `Storyboard.Foundation 0.1.1` references to GameEngine and GameHost.
2. Replaced direct environment expansion in runtime registration catalog, runtime-project, discovery-provider JSON, identity-provider JSON, and static-client mount resolution with `PathExpressionResolver.Resolve`.
3. Preserved relative resolution against each owner’s existing base directory and preserved runtime asset containment under the resolved runtime project directory.
4. Unresolved or malformed expressions no longer fall through as relative paths. Runtime/provider routes become unavailable; static mounts remain disabled and log the Foundation diagnostic.
5. Kept runtime asset locators relative; no authoring source-root dependency was introduced.

## Files Changed

1. `Storyboard.GameEngine/Storyboard.GameEngine.csproj`
2. `Storyboard.GameHost/Storyboard.GameHost.csproj`
3. `Storyboard.GameEngine/GameManager/GameManager.cs`
4. `Storyboard.GameEngine/GameServices/Assets/RuntimeHostAssetManagementClient.cs`
5. `Storyboard.GameEngine/GameServices/Discovery/JsonRuntimeGameDiscoveryProvider.cs`
6. `Storyboard.GameEngine/GameServices/Discovery/RuntimeGameDiscoveryProviderFactory.cs`
7. `Storyboard.GameEngine/GameServices/Identity/RuntimeIdentityProviderFactory.cs`
8. `Storyboard.GameHost/Infrastructure/StaticClientMountingExtensions.cs`
9. This handoff.

## Contract/Interface Impact

Package dependency only; no DTO/schema or runtime asset locator contract changes. Stage 02 equal-root authoring-choice UX remains deferred.

## Validation Commands Executed

1. `dotnet restore Storyboard.GameEngine/Storyboard.GameEngine.csproj --configfile NuGet.Config` — passed.
2. `dotnet build Storyboard.GameEngine/Storyboard.GameEngine.csproj --configuration Release --no-restore --property:UseSharedCompilation=false` — passed, 0 warnings/errors.
3. `dotnet test Storyboard.GameEngine.Tests/Storyboard.GameEngine.Tests.csproj --configuration Release --no-restore --no-build` — passed baseline binary, 657/657.
4. `git diff --check` — passed.
5. Direct-expansion inventory over `Storyboard.GameEngine` and `Storyboard.GameHost` — no remaining `ExpandEnvironmentVariables` calls in the Stage 03 boundary.

## Test Results

The GameEngine project builds cleanly. A clean rebuild of the GameEngine test project and GameHost currently fails during MSBuild project-reference evaluation with `Build FAILED`, 0 warnings, and 0 compiler errors; the existing no-build test binary passed and is not treated as new-code validation. Focused new-case execution remains pending that rebuild repair.

## Behavioral Notes

Source-root expressions must never be required to consume a published runtime asset. Missing or malformed configured expressions now produce safe unavailable/disabled behavior instead of relative fallback.

## Known Issues/Risks

1. The test/GameHost clean rebuild has an environment/MSBuild project-reference failure that needs resolution before final Stage 03 closure.
2. Architecture must update `environment-variables-setup.md` and `asset-source-root-configuration.md`, or explicitly assign those edits to a named approved follow-up stage.
3. Designer equal-depth root choice is a deferred UX item; Host/docs must not represent it as delivered behavior.

## Boundary Compliance Report

Out-of-scope reads: None.  
Out-of-scope edits: None.  
Exceptions: Architecture docs were not edited because they are outside this repository’s writable workspace boundary; they remain an explicit completion dependency.

## Explicit Next-Stage Start Checklist

1. Resolve the clean test/GameHost rebuild issue and run focused registration, provider, identity, runtime-project, static-mount, two-root, and missing-variable cases.
2. Provide the resolved registration/runtime invariants, Foundation provenance, and two-root Host fixture to Simulator.
3. Confirm runtime assets remain `assets/...` locators.
4. Record Architecture documentation delivery status and any named follow-up for Designer equal-root choice UX.
5. Defer WebPortal-specific static-mount and URL verification to Stage 05.
