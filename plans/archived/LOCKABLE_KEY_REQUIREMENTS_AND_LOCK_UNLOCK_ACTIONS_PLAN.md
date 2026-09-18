# Lockable Key Requirements + Lock/Unlock Actions Plan

Status: Decisions locked; ready for implementation sequencing (2026-07-15).

## Goal

Enhance lockable object support by:

1. Defining key requirements on lockable objects.
2. Adding `UnlockObject` action.
3. Adding `LockObject` action.
4. Supporting explicit key naming (`unlock door with key`) and implicit key possession (inventory).
5. Supporting key consumption policies.
6. Supporting multipart key requirements (`all required`, `x of y required`).

## Confirmed Product Rules (From Current Discussion)

1. Lock requirements are owned/defined by the lockable object.
2. Key objects do not store reverse references to unlock targets.
3. A key object can be reused to unlock multiple objects over time.
4. Unlocking always requires key requirements to be satisfied.
5. Locking may require key requirements, depending on object configuration.
6. Authoring UX for lock requirements should mirror composite-recipe flow pattern with a dedicated dialog.

## Decision Checklist (Resolved)

1. [x] Explicit invalid key fails immediately with `WrongKey` (no implicit fallback).
2. [x] Explicit `with <key>` can resolve from inventory or current room.
3. [x] Implicit unlock (no `with`) searches inventory only.
4. [x] Key ambiguity resolves deterministically using key-builder list order, top to bottom.
5. [x] Unlocking an already unlocked target fails with `ObjectAlreadyUnlocked`.
6. [x] Locking an already locked target fails with `ObjectAlreadyLocked`.
7. [x] `LockObject` key requirement is optional per lockable object.
8. [x] Key matching uses object identity only (object id).
9. [x] Per-key quantity requirements are in scope for v1.
10. [x] A single object can satisfy multiple slots when quantity permits.
11. [x] For `AnyNOfM`, consume exactly selected minimum needed keys.
12. [x] Consumption occurs only after full validation and successful state mutation.
13. [x] Separate requirement sets exist for unlock and lock operations.
14. [x] OR-groups are deferred from v1.
15. [x] Existing lockables without unlock requirements fail strictly (`NoUnlockRequirementsDefined`-style outcome).
16. [x] Include detailed diagnostic action tokens in v1.
17. [x] Key builder UX mirrors composite recipe top section through minimum-count controls.
18. [x] Deterministic key selection must follow displayed key list order (top to bottom).

Question count: 18.

## Proposed Data Model (Draft)

### Lockable Requirement Definition (object-owned)

1. `LockRequirementsDefinition`
2. `RequirementMode`: `AllRequired` | `AnyNOfM`
3. `MinimumRequiredCount` (used for `AnyNOfM`)
4. `OperationPolicy`: unlock required always; lock required optional flag
5. `ConsumptionPolicyUnlock`: `DoNotConsume` | `ConsumeUsed`
6. `ConsumptionPolicyLock`: `DoNotConsume` | `ConsumeUsed`
7. `Candidates`: list of key requirement entries

### Key Requirement Entry

1. `RequiredObjectId`
2. `RequiredQuantity` (default 1)
3. optional descriptive label for authoring UX only

## Runtime Behavior Plan

### Key Resolution Inputs

1. Target object to lock/unlock.
2. Parsed explicit secondary object token(s) from command (`with` phrase).
3. Runtime inventory object set.
4. Runtime current-room object set (explicit `with` only).

### UnlockObject Flow (Draft)

1. Resolve target lockable object.
2. Validate object supports lock requirements.
3. Resolve candidate key set from explicit and/or implicit path.
4. If explicit key is provided and invalid, fail `WrongKey` immediately.
5. For implicit path, search inventory only.
6. For explicit path, search inventory and current room.
7. Evaluate requirement mode (`AllRequired` or `AnyNOfM`) with quantity support.
8. Apply deterministic candidate ordering from key-builder list top-to-bottom.
9. If fail: return deterministic failure token and diagnostics.
10. If success: optionally consume used keys by unlock policy (after state mutation success).
11. Set `isLocked=false`.
12. Emit action variable tokens (`action.UnlockedObjectName`, key diagnostics tokens).

### LockObject Flow (Draft)

1. Resolve target lockable object.
2. Fail with `ObjectAlreadyLocked` when already locked.
3. Determine whether lock operation requires keys per object policy.
4. If required: run key resolution/evaluation pipeline using lock-specific requirement set.
5. Optionally consume used keys by lock policy (after state mutation success).
6. Set `isLocked=true`.
7. Emit action variable tokens (`action.LockedObjectName`, key diagnostics tokens).

## Command Parsing Plan

1. Extend parser support for `unlock <target> with <key>` and `lock <target> with <key>`.
2. Preserve existing object token resolution for primary and secondary objects.
3. Enforce precedence: explicit `with` path overrides implicit path and does not fallback.

## Authoring UX Plan

1. In object properties, when `isLockable` is enabled, surface a `Configure Lock Requirements...` action.
2. Open dedicated dialog similar to composite recipe authoring pattern.
3. Dialog supports:
4. candidate key object selection
5. all-required vs any-n-of-m mode
6. minimum required count for n-of-m
7. per-operation consumption toggles
8. lock operation key requirement toggle
9. Validation rules:
10. no duplicate key candidates
11. minimum count valid for candidate count
12. no impossible constraints

## Persistence/Mapping Plan

1. Add lock requirement model fields in designer project serialization.
2. Map to/from clean export contract (additive compatible fields).
3. Map to runtime snapshot descriptors used by command processor.
4. Ensure normalization/rewrite paths preserve requirement definitions.

## Shared Runtime Integration Touchpoints

1. `Storyboard.Shared/RuntimeContracts/Enums/CommandActionType.cs`
2. `Storyboard.Shared/GameServices/Actions/RuntimeActionResultCodeRegistry.cs`
3. `Storyboard.Shared/GameServices/Actions/RuntimeCommandActionExecutor.cs`
4. New executable actions:
5. `RuntimeCommandActionExecutor.UnlockObjectExecutableAction.cs`
6. `RuntimeCommandActionExecutor.LockObjectExecutableAction.cs`
7. New result code enums + token maps for both actions.
8. New action variable resolvers/descriptors/token catalog entries.

## Designer Integration Touchpoints

1. `StoryboardDesigner.App/ViewModels/CommandActionTypeValues.cs`
2. `StoryboardDesigner.App/Models/Actions/ActionPayloads/ActionPayloadSchemaMap.cs`
3. `StoryboardDesigner.App/Models/Actions/CommandActionPayloadCoordinator.cs`
4. `StoryboardDesigner.App/Views/RoomActionEditorDialog.xaml(.cs)` for action-specific visibility/validation.
5. Object properties dialog and new lock requirement editor dialog.
6. Echo token providers for UnlockObject and LockObject.

## Test Plan

### Shared Tests

1. UnlockObject success/failure matrix.
2. LockObject success/failure matrix.
3. Explicit vs implicit key resolution behavior.
4. Inventory vs room source behavior per final policy.
5. Consumption behavior correctness.
6. All-required and any-n-of-m requirement modes.
7. Action variable emission guardrails.

### Designer/App Tests

1. Authoring dialog validation and serialization roundtrip.
2. Runtime snapshot mapper coverage for lock requirements.
3. Command processing integration regressions.
4. Architecture separation guardrails unchanged.

## Validation Gates

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\Storyboard.Shared.Tests\Storyboard.Shared.Tests.csproj`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`
4. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`
5. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`

## Delivery Phases

1. Phase 1: Decision closure for 18 questions.
2. Phase 2: Contract/model scaffolding (no behavior changes).
3. Phase 3: UnlockObject runtime behavior + tests.
4. Phase 4: LockObject runtime behavior + tests.
5. Phase 5: Authoring dialog + persistence/export + mapper.
6. Phase 6: Regression hardening and documentation updates.
