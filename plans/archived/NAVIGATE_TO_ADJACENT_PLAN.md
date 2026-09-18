# Navigate To Adjacent Plan

Status: Implemented
Owner: Runtime command pipeline
Last updated: 2026-07-15

## Plan Maintenance (2026-07-15)
1. Implementation outcome remains accurate and validated by current green solution gate.
2. This plan is archive-ready with no pending phase work.

## Implementation Outcome (2026-07-14)
1. Delivered destination parsing in preprocess result contract and second-pass adjacent room matching.
2. Delivered runtime action type NavigateToAdjacent with destination-first resolution and directional fallback.
3. Delivered explicit runtime result reasons/codes for door-closed, non-passable, ambiguous traversal, unknown destination, and move-failed outcomes.
4. Delivered designer validation warning for duplicate normalized room names within an area.
5. Delivered focused unit/regression coverage for preprocessor extraction, runtime execution outcomes, action token/result-code registries, and validation warning behavior.

## Objective
Add destination-oriented navigation commands (for example: "go to workshop", "enter workshop") without changing existing direction-based navigation behavior.

Clarification (2026-07-14):
- The enhancement focus is robust adjacent-room token extraction from command text regardless of surrounding verbiage.
- Parsed destination data is exposed for actions to consume; action-specific intent handling is resolved at action execution time.

## Scope
1. Add a second preprocessor pass for adjacent-room token matching.
2. Extend preprocess result contract with destination-room fields.
3. Add a new runtime action type: NavigateToAdjacent.
4. Keep existing NavigateDirection behavior unchanged.
5. Add focused regression tests for parsing and execution outcomes.
6. Add a designer validation warning for duplicate room names within the same area.

## Non-Goals
1. No global pathfinding across multiple rooms.
2. No replacement of NavigateDirection.
3. No changes to non-navigation verb behavior beyond conflict resolution rules.

## Proposed Contract Changes
1. In GameCommandPreprocessResult, add:
- DestinationRoomToken (string?)
- HasDestinationRoomQualifier (bool)
- DiagnosticCode (string?)
- DiagnosticDetails (structured diagnostic payload for ambiguity/conflict candidates)
2. Preserve existing fields:
- Direction
- HasDirectionalQualifier
- PrimaryObjectToken and SecondaryObjectTokens

## Preprocessor Design
1. Keep MatchObjectTokensInCommand as first pass (object nouns).
2. Add MatchAdjacentRoomTokensInCommand as second pass:
- Candidate set: only rooms directly reachable from current room.
- Token sources: adjacent room NameInGame first, then Name as fallback when NameInGame is empty.
- Matching mode: longest phrase first, normalized whitespace.
3. Resolve navigation intent by verb class:
- Directional command text -> Direction field.
- Destination command text -> DestinationRoomToken.
4. Conflict handling:
- Do not fail preprocessing solely because both direction and destination tokens are present.
- Preserve both parsed qualifiers so action-specific resolution rules can decide precedence.
- If destination matches multiple adjacent rooms, mark invalid with ambiguity diagnostic.

## Runtime Execution Design
1. Add new action type: NavigateToAdjacent.
2. Executor responsibilities:
- Resolve matching adjacent traversal by DestinationRoomToken.
- Validate traversal passability.
- Evaluate door-linked constraints if present (open/locked).
- Return precise failure messages for blocked, ambiguous, or missing routes.
3. Keep NavigateDirection executor unchanged.
4. Outcome contract for NavigateToAdjacent execution:
- Success outcome when traversal executes.
- Failure outcome with reason DoorClosed when traversal has a door and that door is closed.
- Failure outcome with reason TraversalNotPassable for non-door passability failures.
- Failure outcome with reason TraversalAmbiguous when multiple traversals remain after destination resolution.
5. Qualifier precedence in NavigateToAdjacent:
- If DestinationRoomToken resolves to an adjacent room, honor the room target first.
- If destination room is empty/null/unknown and a directional qualifier is present, fall back to directional resolution.
- If both are present and destination resolves, destination takes precedence.

## Command Routing Rules
1. Directional phrases route to NavigateDirection.
2. Destination phrases route to NavigateToAdjacent.
3. Direct-object verbs (open, unlock, close, use) continue to resolve against object tokens.
4. For commands already routed to NavigateToAdjacent, if both destination and direction qualifiers are present, apply its precedence/fallback rules.

## Test Plan
1. Preprocessor tests:
- "go to workshop" sets DestinationRoomToken.
- "enter supply closet" sets DestinationRoomToken.
- Ambiguous destination yields diagnostic.
- Commands containing both destination and direction preserve both qualifiers for downstream action resolution.
2. Runtime executor tests:
- Success case when adjacent route is passable.
- Blocked case when traversal is not passable.
- Door-gated case when door is closed/locked.
- Unknown destination from current room fails with clear message.
- Duplicate adjacent destination match fails fast with explicit double-match diagnostic.
3. Regression tests:
- Existing "go north" still uses NavigateDirection.
- Existing object commands remain unchanged.
4. Designer validation tests:
- Duplicate normalized room names in the same area produce a warning.
- Distinct normalized room names do not produce this warning.

## Implementation Steps
1. Add preprocess result fields.
2. Implement adjacent-room matcher helper.
3. Wire second pass into preprocessor and diagnostics.
4. Introduce NavigateToAdjacent action type and executor.
5. Add or update routing logic for action binding.
6. Add designer validation rule for duplicate room names within an area.
7. Add unit tests and focused playback regression checks.

## Validation Checklist
0. Between implementation phases, run all existing tests before proceeding to the next phase.
1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameCommandPreprocessorServiceTests|GameCommandProcessorFixtureTests|GameManagerTests"
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
4. dotnet test .\StoryboardDesigner.slnx

Validation result (2026-07-14):
- Step 1 passed.
- Step 2 passed.
- Step 3 passed.
- Step 4 passed (780 passed, 0 failed).

## Risks
1. Ambiguity between room names and object names.
2. Runtime message regressions if failure reasons are not specific.
3. Contract drift if destination fields are added but not consumed consistently.
4. Authors may ignore destination-name ambiguity until runtime if no authoring warning exists.

## Mitigations
1. Apply verb-class routing before final intent binding.
2. Add explicit ambiguity diagnostics and deterministic selection rules.
3. Add contract-focused tests across preprocessor and command processor.
4. Add designer warning for duplicate normalized room names in the same area.

## Open Design Decisions
1. Intent precedence:
- When a command could be both destination and object-oriented, which intent wins first?

Decision (2026-07-14):
- Use verb-family-first routing.
- Under navigation verbs, destination intent is preferred when both destination and object candidates appear.
- If ambiguity remains after verb-family routing, fail with an explicit ambiguity diagnostic.

Clarification (2026-07-14):
- Verb-family-first routing selects the action family.
- Inside NavigateToAdjacent, movement resolution may use two inputs: destination-room first, then directional fallback when destination is empty/null/unknown.
2. Verb classification boundary:
- Should adjacent-room matching depend on a hard-coded destination verb list, or remain verb-agnostic over all remaining tokens?

Decision (2026-07-14):
- Do not use a hard-coded destination verb list in preprocessing.
- Treat token 1 as the command verb and perform adjacent-room matching over remaining tokens regardless of verb identity.
- Keep intent routing decoupled from room-token detection to preserve current architecture.
3. Ambiguity policy:
- Under exact whole-phrase room matching (no partial matching), if multiple adjacent rooms still normalize to the same candidate phrase, should preprocessing fail with ambiguity diagnostics?

Clarification (2026-07-14):
- Do not support partial room matching for destination phrases.
- Example: "east wing" matches only "East Wing"; "east workshop" matches only "East Workshop".
- Example: a short token like "e" should not be treated as a partial room match and should remain available for directional interpretation.

Decision (2026-07-14):
- Yes, fail fast when more than one adjacent room matches the same exact normalized destination phrase.
- Emit a clear diagnostic indicating a double match and include matched room identifiers/names.
- Add an authoring-time designer validation warning when duplicate normalized room names exist in the same area.
4. Direction plus destination in one command:
- For mixed commands (for example, "go north to workshop"), always invalid, or allow only when both qualifiers agree?

Decision (2026-07-14):
- Preserve both qualifiers through preprocessing when mixed direction-plus-destination text appears.
- In NavigateToAdjacent, honor destination first when destination resolves.
- If destination is empty/null/unknown and direction is present, fall back to directional handling.
5. Room token source of truth:
- Should matching use NameInGame only, Name plus NameInGame, and are aliases in-scope now?

Decision (2026-07-14):
- Use NameInGame as the primary destination token source.
- Fall back to Name only when NameInGame is empty.
- Do not include additional aliases in this enhancement.
6. Normalization rules:
- Should normalization include case and whitespace only, or also punctuation stripping and typo tolerance?

Decision (2026-07-14):
- Phase one normalization is limited to case-folding and whitespace normalization.
- Do not strip punctuation in this enhancement.
- Do not add typo tolerance or fuzzy matching in this enhancement.
7. Object versus room token collision:
- If a phrase matches both an object and an adjacent room, what deterministic resolution rule applies?

Decision (2026-07-14):
- Keep preprocessing verb-agnostic for token detection.
- Let routing determine intent by action family.
- If object and destination qualifiers both resolve but imply different actions, fail with an explicit cross-intent ambiguity diagnostic.
- Do not silently prioritize one qualifier in collision cases.
8. Destination contract shape:
- Are DestinationRoomToken and HasDestinationRoomQualifier sufficient, or do we also need structured diagnostics?

Decision (2026-07-14):
- Keep DestinationRoomToken and HasDestinationRoomQualifier.
- Add structured diagnostic fields in preprocessing for machine-checkable failure assertions:
- DiagnosticCode
- DiagnosticDetails (including candidate room ids/names and conflict type where applicable)
- Keep user-facing message composition in runtime/action execution layers.
9. Runtime traversal selection:
- If multiple traversals can target the same destination token, how should selection or failure be handled?

Decision (2026-07-14):
- Runtime here refers specifically to the NavigateToAdjacent action implementation/executor.
- Destination resolution must produce a single adjacent room candidate or fail.
- Executor then evaluates traversals from current room to that destination:
- If exactly one eligible traversal exists and is passable, return success.
- If traversal has a door and door state is closed, return failure reason DoorClosed.
- If traversal is not passable for another reason, return failure reason TraversalNotPassable.
- If multiple traversals remain, return failure reason TraversalAmbiguous.
10. Failure messaging standard:
- Should blocked, locked, ambiguous, and unknown outcomes use canonical message strings locked by tests?

Decision (2026-07-14):
- Keep canonical diagnostic/outcome codes stable and use them as the primary test assertion surface.
- Keep user-facing message templates generally stable, but allow minor copy refinement without changing outcome semantics.
- In playback/output-line regressions, lock only high-value failure-category lines instead of all prose.
11. Phase-one command form scope:
- Should shorthand destination commands (for example, "go workshop") be introduced in phase one, or should phase one require explicit destination phrasing?

Decision (2026-07-14):
- Do not add shorthand destination forms in this enhancement.
- Continue supporting explicit destination phrasing and robust room-token extraction from surrounding command verbiage.
12. Test matrix minimum:
- What exact command set must pass before merge (positive, ambiguous, mixed-qualifier, and collision cases)?

Decision (2026-07-14):
- Minimum must-pass matrix includes:
- Preprocessor destination extraction: explicit destination phrases, mixed surrounding verbiage extraction, no partial room matching, duplicate-adjacent ambiguity diagnostics with candidate details.
- Mixed qualifier preservation: commands containing destination and direction preserve both qualifiers; preprocessing does not fail solely due to both qualifiers being present.
- NavigateToAdjacent execution outcomes: success, DoorClosed, TraversalNotPassable, TraversalAmbiguous, and directional fallback when destination is empty/null/unknown.
- Guardrail regressions: existing NavigateDirection behavior remains intact; existing object command behavior remains intact; designer duplicate-room-name warning coverage remains intact.
- Phase-gate requirement: all existing tests must be run between implementation phases.
