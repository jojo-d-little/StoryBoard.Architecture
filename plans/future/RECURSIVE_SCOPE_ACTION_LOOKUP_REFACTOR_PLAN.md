# Recursive Scope Action Lookup Refactor Plan

## Why This Plan Exists
Current command action lookup in runtime command processing is mostly hard-coded by scope levels (object, room, global objects, area, country, planet).
This works, but makes scope evolution expensive and encourages special-case branching.
A recursive scope-based lookup model will make command dispatch easier to maintain and safer to extend.

## Problem Statement
- Action lookup order is implemented with explicit sequential branches in command processing.
- Scope traversal logic is duplicated across methods and tied to known hierarchy depth.
- New scope kinds or structural changes can require touching many branches.
- Bubble-up behavior and nearest-match semantics are harder to reason about when lookup is non-generic.

## Goals
- Refactor action lookup after preprocessing to use recursive/iterative scope traversal, not hard-coded scope ladders.
- Preserve existing lookup precedence and behavior unless explicitly changed.
- Keep synonym/linked-flow execution behavior unchanged.
- Improve testability of scope traversal and match precedence.

## Non-Goals
- No action payload or scripting behavior changes.
- No clean export/runtime contract schema changes.
- No large redesign of preprocess tokenization in this effort.

## Current System Snapshot
Primary pipeline entry points:
- `Storyboard.Shared/GameServices/Commands/RuntimeCommandProcessorService.cs`
  - `Process(...)`
  - `FindScopedActionMatches(...)`
  - `GetBestMatchingActionForScope(...)`
- `Storyboard.Shared/GameServices/Commands/GameCommandActionTriggerMatcher.cs`
  - `IsMatch(...)`

Current behavior (simplified):
1. Preprocess command.
2. If explicit room command phrase matched, execute linked action ids.
3. Else scoped lookup in this order:
   - object scope (if parsed object token)
   - room scope
   - global object scopes
   - area
   - country
   - planet
4. Execute first/ordered matches with bubble-up control.

## Proposed Design

### 1) Introduce a Scope Traversal Builder
Create a single method that yields candidate scopes in precedence order for a parsed command.

Candidate signature:
- `EnumerateActionCandidateScopes(RuntimeCommandProcessingRequest request, GameStateScopeNode roomNode, GameCommandPreprocessResult parsed)`

Responsibilities:
- Resolve scoped object candidate(s) first when requested.
- Then yield room, global object roots/objects, then ancestor chain recursively.
- Avoid duplicate scope nodes via visited set keyed by `ScopeNodeId` or object reference.

### 2) Centralize Matching Per Scope
Retain existing trigger logic but route through one matcher method:
- `TryGetMatchingActionForScope(scopeNode, parsed, out action)`

Keep existing semantics:
- NoVerbLinkage stays excluded for command-trigger matching.
- Directional triggers remain honored through `GameCommandActionTriggerMatcher.IsMatch`.

### 3) Recursive Ancestor Walk
Instead of explicit area/country/planet blocks, use parent recursion from room:
- room -> parent -> parent ... until global boundary or null.

This removes hard-coded depth assumptions.

### 4) Global Objects Handling
Keep global objects traversal explicit but generic:
- Enumerate player/global-object root children through world lookup/session helper.
- Optionally include nested objects as needed by current behavior.
- Preserve precedence relative to room and ancestor scopes.

### 5) Diagnostics Consistency
Standardize diagnostics messages from candidate traversal and match outcomes:
- object token unresolved
- candidate scope visited
- scope matched action
- no match found

## Migration Phases

### Phase 1: Characterization and Guardrails
- Add/expand tests to lock current precedence and selection behavior.
- Capture expected order for ambiguous matches across scopes.

Deliverable:
- Failing tests if precedence changes unintentionally.

### Phase 2: Extract Traversal Utilities
- Extract candidate-scope enumeration and dedupe helper methods.
- Keep old method behavior using new helpers to minimize risk.

Deliverable:
- Behavior unchanged, code less duplicated.

### Phase 3: Switch to Recursive Lookup Core
- Replace explicit scope blocks in `FindScopedActionMatches` with traversal enumerator.
- Keep action execution and bubble-up unchanged.

Deliverable:
- Recursive scope traversal in production path.

### Phase 4: Diagnostics and Cleanup
- Simplify residual special-case logic.
- Improve internal comments for traversal precedence.

Deliverable:
- Cleaner code and stable diagnostics.

## Proposed Acceptance Criteria
- Command lookup behavior matches baseline for existing fixtures.
- No regressions in linked action flow, synonyms, and directional qualifiers.
- New traversal code has focused unit tests for precedence and deduping.
- Full solution build and tests pass.

## Validation Strategy
Default:
1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`

Runtime-focused:
1. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameManagerTests|GameSimulatorPlaybackRegressionTests"`

## Initial Test Backlog
- Object-token command resolves object action before room/ancestor scopes.
- Room action selected when object token absent.
- Global object action selected before area/country/planet when applicable.
- Ancestor recursion order preserved room->area->country->planet.
- Duplicate scopes not evaluated twice in one lookup pass.
- Directional qualifiers still prioritize directional-trigger actions.

## Risks
- Behavior drift in subtle precedence cases.
- Over-broad dedupe suppressing legitimate repeated candidates.
- Diagnostics churn affecting test baselines.

## Open Questions
- Should global object scopes be considered part of ancestor recursion or a separate parallel branch long-term?
- Should multiple object token candidates continue to short-circuit on first matching object scope?
- Do we want a configurable precedence policy for future custom hosts?

## Implementation Checklist

### Prep
- [ ] Confirm baseline behavior by running runtime-focused command processor tests.
- [ ] Add explicit characterization tests for current scope precedence and tie-break behavior.

### Refactor Step 1: Extract Candidate Traversal
- [ ] Add `EnumerateActionCandidateScopes(...)` helper in `RuntimeCommandProcessorService`.
- [ ] Ensure helper yields scopes in existing precedence order.
- [ ] Add dedupe guard (by `ScopeNodeId` when present, fallback to reference identity).

### Refactor Step 2: Centralize Scope Match
- [ ] Add `TryGetMatchingActionForScope(...)` wrapper using `GameCommandActionTriggerMatcher.IsMatch`.
- [ ] Preserve existing ordering semantics for directional-trigger preference.

### Refactor Step 3: Replace Hard-Coded Ladder
- [ ] Update `FindScopedActionMatches(...)` to use traversal helper instead of explicit area/country/planet blocks.
- [ ] Preserve object-token-first semantics and unresolved-object diagnostics.

### Refactor Step 4: Recursive Ancestor Walk
- [ ] Add room-ancestor enumerator (`room -> parent -> ...`) with clear stop conditions.
- [ ] Keep global objects branch precedence unchanged relative to ancestors.

### Refactor Step 5: Diagnostics Stabilization
- [ ] Normalize diagnostic message wording/order for matched scopes and misses.
- [ ] Update tests that assert diagnostics text where necessary.

### Validation Gates
- [ ] `dotnet build .\\StoryboardDesigner.slnx`
- [ ] `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj`
- [ ] `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameManagerTests|GameSimulatorPlaybackRegressionTests"`

### Done Criteria
- [ ] No behavioral regressions in command-action lookup precedence.
- [ ] No duplicate-scope matching in one lookup pass.
- [ ] Code path no longer depends on hard-coded depth-specific scope blocks.
