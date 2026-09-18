# Enhancement Misses Plan

Last updated: 2026-06-29

Purpose: capture high-value follow-up items discovered from deep review of root planning documents that were not fully closed or not yet promoted to ongoing operating guidance.

## 1) High-Priority Misses

1. Final architecture/dependency validation gate is now closed (2026-06-29).
- Source: `DESIGNER_VS_GAME_SEPARATION_PLAN.md`
- Why it matters: this confirms separation is not only implemented but regression-protected.
- Closure evidence:
  - Added guardrail tests in `StoryboardDesigner.App.Tests/ArchitectureSeparationGuardrailsTests.cs`:
    - `SimulatorProject_DoesNotReferenceDesignerProjectOrNamespace`
    - `SimulatorSource_DoesNotUseDesignerNamespaces`
    - `SimulatorStartup_ComposesSharedRuntimeManager`
  - Verified standalone simulator build: `dotnet build .\\Storyboard.Simulator\\Storyboard.Simulator.csproj`.
  - Verified host fixture runtime path: `SharedManagerHostFixtureTests` passing.
- Ongoing validation checklist:
  - `dotnet build .\\Storyboard.Simulator\\Storyboard.Simulator.csproj`
  - `dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "ArchitectureSeparationGuardrailsTests|SharedManagerHostFixtureTests"`

2. Export contract regression baseline breadth is still narrow.
- Source: `PHASE_1_2_3_REVIEW.md`
- Why it matters: snapshots currently emphasize single-room fixture and may miss broader regressions.
- Suggested action:
  - Add Birmingham snapshot baseline coverage for clean export outputs.
  - Keep update flow explicit (`UPDATE_CLEAN_EXPORT_SNAPSHOTS=1`).

3. Object `commands` write-path decision resolved (2026-06-29).
- Source: `PHASE_1_2_3_EXECUTION_PLAN.md`
- Why it matters: this prevented continued persistence of an ambiguous object-level command-string field that was not part of the active runtime command-phrase routing path.
- Resolution:
  - Stopped writing object `Commands` in project/export/clean-export write mappings.
  - Preserved backward-compatible reads for legacy data (null-safe load behavior retained).
  - Kept scoped object actions/linked-actions behavior unchanged.

## 2) Medium-Priority Misses

1. Command Actions plan is stale relative to current runtime extraction work.
- Source: retired `COMMAND_ACTIONS_PLAN.md` (deleted 2026-06-29).
- Observation: old checklist and assumptions were no longer aligned with current architecture.
- Next action:
  - Create a new scoped command-actions plan when this workstream is reprioritized.

2. MVVM architecture plan is stale and predates current project split.
- Source: `MVVM_ARCHITECTURE_PLAN.md`
- Observation: baseline references no tests and early scaffolding assumptions no longer hold.
- Suggested action:
  - Refresh with current architecture reality and a short prioritized debt backlog.

3. Shared extraction draft should be converted into concise accepted boundary policy.
- Source: retired `SHARED_DLL_EXTRACTION_FILE_LIST.md` (deleted 2026-06-29).
- Observation: historical extraction tracker was too noisy for ongoing policy use.
- Next action:
  - Keep active boundary rules in `ENHANCEMENT_GUIDELINES.md`.

## 3) Low-Priority Misses

1. Root doc organization and status headers are inconsistent.
- Suggested action:
  - Standardize each active planning doc header:
    - `Status`
    - `Owner`
    - `Last updated`
    - `Supersedes` or `Superseded by`

2. Plan lifecycle policy is implicit, not explicit in a dedicated index.
- Suggested action:
  - Add `plans/README.md` index with file status and retention notes.

## 4) Suggested Immediate Next Slice (Small + Safe)

1. Add Birmingham clean-export snapshot fixture.
2. Refresh or retire stale command/MVVM plans.
3. Resume simulator automation design from `SIMULATOR_TEST_AUTOMATION_ENHANCEMENT_PLAN.md` when prioritized.

## 5) Promotion Summary (Already Applied This Session)

Valuable rules already promoted from root plans into living docs:

1. `.github/copilot-instructions.md`
- Added persistence separation and clean export contract intent.
- Added runtime-focused validation command.

2. `ENHANCEMENT_GUIDELINES.md`
- Added persistence/export invariants.
- Added clean export contract/versioning policy.
- Added runtime separation guardrails and focused regression gate.
- Added operational notes for build-lock and snapshot refresh.

3. `README.md`
- Updated to current multi-project architecture.
- Added data/export model summary.
- Added guidance-document map and plan maintenance policy.
