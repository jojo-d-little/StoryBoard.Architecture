# Action-Owned Clarification Requests Plan

Status: Deferred for Later Discussion (No implementation in current session)
Owner: Storyboard.Shared runtime command pipeline
Last updated: 2026-07-27

## 1. Purpose

Define a future architecture where action implementations can request requirement clarifications through the command processor, without hard-wired action-specific branching in the preprocessor.

This keeps preprocessing focused on parsing and tokenization, while action-specific execution requirements are evaluated by action and processor layers.

## 2. Why this Plan Exists

Current behavior includes a special-case path for SetActiveRoomObject target clarification in preprocessor logic.

That path works, but it mixes action-specific requirement policy into preprocessing.

Desired direction:

1. Preprocessor parses command text and returns structured parse results.
2. Processor selects candidate actions and orchestrates execution.
3. Action implementation can declare missing required inputs and request a clarification prompt.
4. Processor negotiates and returns unified clarification payload to host.

## 3. Problem Framing

Clarification needs are not all the same. We should separate:

1. Interpretive clarification:
- User said something ambiguous and system needs disambiguation.
- Example: key maps to multiple objects.

2. Requirement clarification:
- User did not provide required input for a selected action.
- Example: select without an object target.
- Example: move without direction when no fallback direction is available.

This plan focuses on giving action implementations a first-class way to request requirement clarifications.

## 4. Design Goals

1. Keep preprocessor clean and parse-focused.
2. Avoid hard-wired magic action lists in preprocess stage.
3. Let action implementations express required-input rules.
4. Keep clarification lifecycle centralized in processor for consistency.
5. Support future qualifying questions beyond object identity.
6. Preserve existing host-facing clarification contract shape where practical.

## 5. Scope and Non-Goals

In scope for future implementation:

1. Add an action preflight contract for requirement checks.
2. Allow action to return ClarificationRequired with structured request metadata.
3. Have processor merge action request into existing clarification result flow.
4. Migrate SetActiveRoomObject requirement clarification from preprocessor to action plus processor path.

Out of scope for this planning item:

1. Implementing full natural-language question generation.
2. Replacing existing synonym ambiguity logic in first cut.
3. Broad runtime command grammar redesign.

## 6. Candidate Contract Shape (Draft)

Action preflight response should be able to report:

1. IsReadyToExecute
2. RequirementKey
3. ClarificationKind (Requirement, Interpretive)
4. PromptText or PromptToken
5. Candidate options (optional)
6. Answer application strategy (how selected answer maps to execution input)

Processor responsibilities should include:

1. Run normal parse.
2. Select matching action candidates.
3. Invoke action preflight for selected candidate.
4. If preflight requests clarification, return standard pending clarification response.
5. On subsequent command correlation, inject answer and continue execution.

## 7. Future Clarification Types to Consider

This plan should explicitly support non-object requirement questions in addition to object target questions:

1. Missing direction:
- Which direction do you mean?

2. Missing distance or quantity:
- How far?
- How many?

3. Missing container or destination target:
- Which container?
- Where should this go?

4. Missing mode or operation type:
- Do you want to rotate by degrees or face a direction?

5. Missing secondary object in two-object actions:
- Which object should be used as the target?

## 8. Migration Strategy (Draft)

Phase 1: Introduce additive processor and action preflight interfaces.

Phase 2: Implement SetActiveRoomObject preflight requirement clarification.

Phase 3: Processor prefers action preflight clarification for SetActiveRoomObject.

Phase 4: Remove SetActiveRoomObject special-case handling from preprocessor.

Phase 5: Expand to other actions where requirement clarifications are useful.

## 9. Risk and Compatibility Notes

1. Clarification correlation and slot identity must remain stable during migration.
2. Regression tests for existing clarification lifecycle must stay green.
3. Playback and diagnostics baselines may need targeted updates where behavior ordering changes.

## 10. Validation Strategy (When Activated)

Suggested focused gates:

1. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"

2. dotnet test .\StoryboardDesigner.slnx

## 11. Deferred Status Note

No implementation is requested at this time.

This document is intentionally captured for future discussion and execution planning, with explicit intent to revisit and activate later.