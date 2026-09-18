# Scalar Name Count Token Removal Plan

## Goal
Remove scalar-name count action tokens (for example, action.selectedObjectNameCount) and align tests to validate real scalar name tokens and non-empty values when applicable.

## Status
Deferred planning only. No runtime or test behavior changes in this step.

## Scope
In scope:
- Shared action variable descriptor metadata for scalar-name count tokens.
- Shared action variable resolver tests that currently assert count tokens for scalar names.
- Designer registry/token-provider tests that currently expect scalar-name count tokens.

Out of scope:
- List count tokens for true collections (for example missingPartsCount, usedKeysCount).
- Any unrelated action variable contract changes.

## Candidate Tokens To Remove
Initial scalar-name count candidates:
- action.selectedObjectNameCount
- action.OpenedObjectNameCount
- action.ClosedObjectNameCount
- action.UnlockedObjectNameCount
- action.LockedObjectNameCount
- action.movedObjectNameCount
- action.rotatedObjectNameCount
- action.stackedObjectNameCount
- action.stackTargetObjectNameCount
- action.targetContainerNameCount
- action.sourceContainerNameCount
- action.removedObjectNameCount

## Implementation Plan
1. Inventory usage and compatibility check
- Search code, tests, and authored sample data for each candidate token.
- Confirm no production scripts in shipped samples depend on these tokens.
- Decide whether to keep compatibility aliases temporarily.

2. Update shared canonical descriptors
- Remove scalar-name count declarations from RuntimeActionVariableDescriptors for affected action types.
- Keep scalar name tokens and action.all entries.

3. Update resolver emission behavior
- Stop emitting scalar-name count values from affected action variable resolvers.
- Keep scalar-name values unchanged.

4. Update tests to reflect intended contract
- Replace count-token assertions with:
  - Presence of scalar token.
  - Non-empty scalar value when a name is expected.
  - Empty scalar value when no name is resolved.
- Update registry expectation tests to no longer expect removed count tokens.
- Keep guardrail tests ensuring emitted tokens are declared.

5. Validate and stabilize
- Run focused shared/app tests for action variable descriptors and resolvers.
- Run full app test suite.
- Run playback regression smoke gate to ensure no output regressions.

## Risk Assessment
Primary risks:
- Contract break for external projects/scripts that reference removed count tokens.
- Hidden assumptions in tests or tooling that rely on count token presence.

Mitigations:
- Do usage inventory before edits.
- Stage removal in one small change set with complete test updates.
- If compatibility concerns are found, add transitional support and deprecation notes before full removal.

## Validation Commands
- dotnet test .\Storyboard.Shared.Tests\Storyboard.Shared.Tests.csproj
- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
- dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
- dotnet build .\StoryboardDesigner.slnx

## Exit Criteria
- Scalar-name count tokens removed from canonical descriptors and resolver outputs.
- Tests assert scalar-name semantics directly (including empty/non-empty expectations).
- Full regression suite passes.
