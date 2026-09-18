# ScopeChain Pipeline Consolidation Post-Closeout Plan

Last updated: 2026-09-14
Status: Deferred (post-closeout follow-up)
Owner: Runtime follow-up

Purpose: carry forward the Stage 07 deep review item that was intentionally split out of Action Variable Chooser Event Anchor Unification closeout scope.

## Problem Statement

`ScopeChainReferenceValueResolver` full-map population and `RuntimeSessionAnchorDataProvider.TryResolveSingle(...)` single-path resolution now coexist. Stage 07 closeout removed legacy fallback bridges, but long-term ownership of full-map behavior versus single-path canonical behavior still needs an explicit architectural decision.

## Why This Is Deferred

1. It is not required to complete action-output canonicalization and retirement closeout.
2. The immediate workstream closeout can complete safely with current behavior and guardrails.
3. This requires a focused architecture/performance pass that should not block the current staged handoff closure.

## Prior Context Source

1. Originated from Stage 07 deep review item in:
- `plans/active/handovers/ACTION_VARIABLE_CHOOSER_EVENT_ANCHOR_UNIFICATION_07_ACTION_TOKEN_LEGACY_RETIREMENT_HANDOFF.md`

## Confirmed Current Wiring Baseline

1. `RuntimeReferenceValueResolverPipeline.CreateDefault()` includes:
- `ScopeChainReferenceValueResolver`
- `ObjectHierarchyReferenceValueResolver`

2. `RuntimeSessionAnchorDataProvider.Resolve(session)` invokes `_referenceValueResolverPipeline.Resolve(session)` (full-map population path).

3. `RuntimeSessionAnchorDataProvider.TryResolveSingle(...)` resolves token-by-token independently of full-map population.

4. `RuntimeAnchorLookupCache` now uses `TryResolveSingle(...)` directly and no longer has legacy full-map fallback behavior.

## Review Goals

1. Decide if full-map behavior remains pipeline-owned long-term.
2. Decide whether `ScopeChainReferenceValueResolver` stays as-is, becomes a thin facade, or is retired.
3. Confirm whether any remaining runtime consumers still require eager full-map population.
4. Quantify tradeoffs (behavior clarity, performance, maintenance complexity).

## Decision Checklist

1. Keep pipeline as authoritative for full-map values, or generate full-map values from provider canonical logic.
2. Keep or retire `ScopeChainReferenceValueResolver`.
3. Confirm no hidden reliance on removed fallback behavior.
4. Preserve deterministic token/value behavior for existing command/event/action paths.

## Evidence Required Before Decision

1. File-level usage map of every full-map resolver call site.
2. Parity tests comparing representative full-map values against `TryResolveSingle(...)` outputs.
3. Focused replay and command/event/timer regression results.
4. Short performance notes for representative token-heavy script resolution flows.

## Candidate File Targets

1. `Storyboard.GameEngine/GameServices/References/RuntimeReferenceValueResolverPipeline.cs`
2. `Storyboard.GameEngine/GameServices/References/ScopeChainReferenceValueResolver.cs`
3. `Storyboard.GameEngine/GameServices/References/RuntimeSessionAnchorDataProvider.cs`
4. `Storyboard.GameEngine/GameServices/References/RuntimeSessionAnchorValueProvider.cs`
5. `Storyboard.GameEngine/GameServices/Actions/RuntimeCommandActionExecutor.cs`
6. `Storyboard.GameEngine/GameServices/Commands/GameCommandProcessorService.cs`
7. `Storyboard.GameEngine.Tests/*Reference*`
8. `StoryboardDesigner.App.Tests/*PlaybackRegression*`

## Validation Gate

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj --filter "FullyQualifiedName~RuntimeSessionAnchorDataProviderTests|FullyQualifiedName~RuntimeReferenceValueResolverPipelineTests"`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`
4. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests"`

## Exit Criteria

1. A documented keep/retire decision for `ScopeChainReferenceValueResolver`.
2. Updated ownership model for full-map versus single-path resolution.
3. Guardrail/regression evidence proving no behavior drift.
