# Shared Variable Containment and Set Property Resolution Plan

Status: Implemented
Owner: Storyboard.Shared runtime/state + command processing
Last updated: 2026-07-15

## Plan Maintenance (2026-07-15)
1. Implementation and validation records are current.
2. This plan is archive-ready; no remaining implementation slices are open.

## Implementation Outcome (2026-07-15)

1. Delivered canonical shared value-cell authority in runtime session internals while preserving per-scope participant variable presence and existing consumer-facing mutation/read APIs.
2. Delivered deterministic shared initialization precedence with mismatch diagnostics and retained restriction enforcement behavior.
3. Delivered common scope-chain mutation target resolution with normalized token handling for raw, wrapped, and quoted-wrapped forms.
4. Delivered SetGameProperty unification onto the common scope-chain resolver path with diagnostics-level gated unresolved-target details.
5. Delivered additive designer warning coverage for shared initial-value mismatch and shared value-restriction mismatch without changing designer shared model or shared-variable JSON schema shape.
6. Delivered zero-drift regression guard for shared-variable JSON shape and completed full-suite validation signoff.

## Final Validation Result (2026-07-15)

1. dotnet build .\\StoryboardDesigner.slnx: passed.
2. dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests": passed.
3. dotnet test .\\StoryboardDesigner.App.Tests\\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep": passed.
4. dotnet test .\\StoryboardDesigner.slnx: passed.

## Objective

Deliver a contained runtime design where shared-variable participation is internal to variable/session behavior, while consumers keep using existing variable read/write APIs unchanged.

In parallel, unify `SetGameProperty` variable-name resolution with the same scope-chain semantics used by reference value resolution so tokens like `self.isCut` and `{self.isCut}` resolve consistently.

## Why This Plan Exists

Two changes now overlap and should be intentionally sequenced:

1. Shared variable implementation should move from propagation-as-authority to shared-value authority, but without removing per-scope variable presence.
2. `SetGameProperty` property-target resolution should use common scope-chain logic instead of bespoke lookup paths.

Doing (1) first reduces complexity/risk for (2) and avoids broad churn.

## Scope

In scope:

1. `Storyboard.Shared/GameStateData` shared-variable behavior and session internals.
2. Shared runtime resolver surfaces used by `SetGameProperty` action execution.
3. Targeted regression tests in `Storyboard.Shared.Tests` and any required fixture updates in `StoryboardDesigner.App.Tests`.

Out of scope:

1. Host UX changes in Designer or Simulator.
2. Export schema redesign.
3. Broad runtime contract redesign unrelated to variable resolution.
4. Any change to designer shared-variable authoring models.
5. Any change to JSON structure/schema used to define shared variables.

## Hard Constraints (Must Not Change)

1. Designer-side shared-variable models remain unchanged.
2. Shared-variable JSON shape/schema remains unchanged.
3. Runtime behavior changes are implemented in runtime internals; no consumer API churn.
4. One additive exception is allowed: designer-time validation warnings that do not alter model shape or JSON contracts.

## Design Principles (Locked)

1. Per-scope variable instances remain present in their owning scope states.
2. Consumers remain unaware of share participation; no broad API churn.
3. Shared participation is encapsulated in variable/session read-write behavior.
4. Shared value must behave as single-source authority for all participants.
5. Non-shared variables preserve current behavior.

## Plan Phases

## Phase 0 - Baseline and Guardrails

Goal: freeze expected behavior before internals move.

Tasks:

1. Inventory current shared-variable read/write code paths in `GameStateSession`.
2. Add/lock regression tests for current externally observable behavior that must remain stable.
3. Explicitly capture any behavior we intentionally change (for example, drift edge cases).
4. Add a designer validation rule that warns when variables in the same shared group have mismatched initial values.
5. Add a designer validation rule that warns when variables in the same shared group have mismatched value restrictions/types.

Exit criteria:

1. Guardrail tests exist for read/write, linked bindings, and command-facing usage.
2. No ambiguity on preserved vs intentionally changed behavior.
3. Designer validation emits a warning for mismatched initial shared values, with no model/JSON contract changes.
4. Designer validation emits a warning for mismatched shared participant value restrictions/types.

## Phase 1 - Shared Variable Containment Rework

Goal: keep participant variables in place while making shared value authoritative through contained internals.

Tasks:

1. Introduce canonical shared value backing keyed by shared variable id.
2. Update shared participant read path to resolve through canonical backing.
3. Update shared participant write path to write through canonical backing.
4. Retain participant variables in each scope home and preserve existing consumer surfaces.
5. Keep compatibility paths during transition where needed, but ensure canonical source wins.

Exit criteria:

1. Shared participant value reads are consistent regardless of which participant is read.
2. Writing through any participant is immediately reflected by all participants.
3. Existing consumer APIs and call sites remain unchanged.
4. No designer-model or shared-variable JSON contract changes are introduced.

## Phase 2 - Common Scope-Chain Reference Variable Resolver

Goal: establish common resolver logic reusable by both reference evaluation and property-target resolution.

Tasks:

1. Extract/add a scope-chain variable resolver component in Shared runtime services.
2. Ensure canonical token handling covers plain and wrapped references (for example `self.x`, `{self.x}`, quoted wrappers when applicable).
3. Preserve existing action-centric exclusions (`action.*`) where required.
4. Route existing scope-chain resolution call sites through the common component.

Exit criteria:

1. One common scope-chain variable resolution path is used by reference evaluation.
2. Behavior parity maintained for chooser/echo and other reference-resolving flows.

## Phase 3 - SetGameProperty Unification

Goal: make `SetGameProperty` resolve property targets via the common scope-chain resolver.

Tasks:

1. Replace bespoke `SetGameProperty` target lookup with common resolver path.
2. Normalize property-target token inputs before resolution.
3. Keep writes routed through session mutation APIs so shared-variable semantics remain centralized.
4. Add regressions for `self.*`, scope-prefixed aliases, and shared-binding write-through scenarios.

Exit criteria:

1. `SetGameProperty` resolves targets consistently with scope-chain reference rules.
2. Known failure case (`self.isCut`) is covered and passing.
3. Shared linked variables update correctly when set through any participant alias.
4. No designer-model or shared-variable JSON contract changes are introduced.

## Phase 4 - Hardening and Cleanup

Goal: reduce tech debt introduced by migration slices.

Tasks:

1. Remove obsolete propagation-only helper logic no longer needed.
2. Remove duplicated resolver code paths superseded by common resolver.
3. Tighten diagnostics for unresolved targets to aid authoring/debugging.

Exit criteria:

1. No duplicated variable-target resolution logic remains in runtime command paths.
2. Tests/documentation reflect final contained design.

## Validation Gates

Run after each meaningful phase:

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`
4. Add focused `Storyboard.Shared.Tests` runs for new resolver/shared-variable coverage once test names are finalized.
5. Confirm no edits to designer shared-variable model definitions or shared-variable JSON contract/schema artifacts.
6. Add/validate designer validation tests for mismatched initial shared values.
7. Add/validate designer validation tests for mismatched shared participant value restrictions/types.
8. Phase 3 signoff requires full-suite test execution across all test projects in the solution.
9. Zero-drift certification requires all of: no designer model/JSON contract file shape edits, representative shared-variable serialization round-trip checks, and an explicit regression test asserting shared-variable JSON shape fields are unchanged.

## Risks and Mitigations

1. Risk: subtle behavior shifts in shared-variable edge cases.
- Mitigation: Phase 0 regression lock + incremental internal changes.

2. Risk: resolver extraction introduces token normalization regressions.
- Mitigation: add explicit token-shape tests (`raw`, `{wrapped}`, quoted variants).

3. Risk: hidden coupling to old propagation helpers.
- Mitigation: postpone helper cleanup until after Phase 3 parity is green.

## Design Lock-Off Questions (Committed)

1. SV-01: What is the canonical shared backing representation (full `RuntimeVariableValue` reuse vs dedicated shared value cell model)?
2. SV-02: What is the initialization rule when participants in the same share start with different values (first encountered wins, deterministic precedence order, or fail-fast)?
3. SV-03: What is the lifecycle rule if a participant variable is removed/recreated at runtime while its shared id remains referenced?
4. SV-04: Do we preserve write-through behavior for all variable types currently participating in shares, or explicitly constrain allowed types?
5. SV-05: For property-target token normalization in `SetGameProperty`, do we allow both raw and wrapped forms (`self.x`, `{self.x}`, quoted wrapped forms), and which malformed patterns are rejected?
6. SV-06: Which scope aliases are lock-off required for property-target resolution parity (`self`, `room`, `area`, `country`, `planet`, `global`, `player`)?
7. SV-07: When target resolution fails, what diagnostics contract is required now (current message parity vs richer searched-scope detail)?
8. SV-08: What is the exact parity boundary for action-specific tokens (for example `action.*`) when reusing common scope-chain resolver code?
9. SV-09: What regression set is mandatory before Phase 3 signoff (minimum test list and ownership across `Storyboard.Shared.Tests` and `StoryboardDesigner.App.Tests`)?
10. SV-10: What objective proof is required to certify zero drift in designer shared-variable models and shared-variable JSON schema/shape?

## Lock Decisions (Captured)

1. SV-01: Dedicated shared value cell model (not direct participant object authority).
2. SV-02: Deterministic precedence order winner, with diagnostics emitted in runtime/designer-visible diagnostic output.
3. Companion requirement: add designer validation warning when shared participants start with mismatched values.
4. SV-03: If a participant variable leaves a share at runtime, it becomes fully independent (own value, no shared membership), and the share no longer tracks it.
5. SV-03 planning note: explicit runtime support for unbind/rebind lifecycle should be designed intentionally in this plan, but default semantics remain "leave share => independent variable".
6. SV-04: Preserve compatibility breadth for runtime writes; runtime may accept any incoming string value, but target variable restrictions remain authoritative (`Unrestricted`/`Numeric`/`TrueFalse`) and can reject invalid writes.
7. Companion requirement: add designer validation warning when shared participants in one group have mismatched value restrictions/types.
8. SV-05: SetGameProperty target normalization accepts raw, brace-wrapped, and quoted wrapped tokens (after trim), with malformed patterns rejected.
9. SV-06: Required SetGameProperty target alias set is full scope-chain parity: `self`, `room`, `area`, `country`, `planet`, `global`, `player`.
10. SV-07: Use diagnostics-level gating for unresolved SetGameProperty target detail: default output remains minimal; medium/high diagnostics include normalized token, searched aliases/scopes, and rejection reasons.
11. SV-08: Resolver composition boundary is shared-core plus action extension: action resolver consumes scope-chain resolver output and unions in action-specific tokens (for example `action.*`) without pushing action concerns into the core scope resolver.
12. SV-09: Mandatory regression set for Phase 3 signoff is all tests (full solution test suite), with focused gates treated as supplemental diagnostics.
13. SV-10: Zero-drift proof standard is code-diff checks plus serialization round-trip verification plus an explicit shared-variable JSON shape guard regression test.

Lock-off rule:

1. Phase 1 implementation should not start until SV-01 to SV-04 are answered.
2. Phase 3 implementation should not start until SV-05 to SV-10 are answered.

## Implementation Start Checklist (Locked-Decision Mapping)

1. Shared value-cell introduction (SV-01/SV-02/SV-03/SV-04)
- First touch files:
	- `Storyboard.Shared/GameStateData/GameStateSession.cs`
	- `Storyboard.Shared/GameStateData/SharedVariableBinding.cs`
	- `Storyboard.Shared/GameStateData/GamePropertyRuntimeValue.cs`
	- `Storyboard.Shared/GameStateData/GamePropertyState.cs`
- Checklist:
	- Add canonical shared value-cell backing keyed by shared id.
	- Apply deterministic precedence initialization and emit diagnostics on mismatch.
	- Preserve participant variable presence in each scope state.
	- Enforce "leave share => independent variable" semantics.
	- Preserve current restriction enforcement behavior (`Unrestricted`/`Numeric`/`TrueFalse`).

2. Designer validation additions (companion requirements)
- First touch files:
	- `StoryboardDesigner.App/Validation/Rules/Project/` (new rule files)
	- `StoryboardDesigner.App/Validation/Execution/ValidationRuleRegistry.cs`
	- `StoryboardDesigner.App.Tests/` (validation rule tests)
- Checklist:
	- Add warning rule for mismatched initial values in one shared group.
	- Add warning rule for mismatched value restrictions/types in one shared group.
	- Ensure warnings are additive only (no model/schema edits).

3. Common scope-chain resolver extraction (SV-05/SV-06/SV-08)
- First touch files:
	- `Storyboard.Shared/GameServices/References/ScopeChainReferenceValueResolver.cs`
	- `Storyboard.Shared/GameServices/References/` (new common resolver component)
	- `Storyboard.Shared/GameServices/Commands/RuntimeCommandProcessorService.cs`
- Checklist:
	- Introduce reusable scope-chain token resolver component.
	- Keep action-specific tokens outside core resolver.
	- Compose action resolver as union (scope-core + action additions).
	- Support normalized targets: raw, brace-wrapped, quoted wrapped.

4. SetGameProperty unification (SV-05/SV-06/SV-07)
- First touch files:
	- `Storyboard.Shared/GameServices/Actions/GameActions/RuntimeActionExecutable/RuntimeCommandActionExecutor.SetGamePropertyExecutableAction.cs`
	- `Storyboard.Shared/GameServices/Actions/RuntimeActionPayloadAccessors.cs`
	- `Storyboard.Shared/GameServices/References/` (normalization/resolution helpers)
- Checklist:
	- Replace bespoke target lookup with common scope-chain resolver path.
	- Keep write path through session mutation APIs.
	- Add diagnostics-level gated detail on unresolved targets.

5. Zero-drift proof and signoff (SV-09/SV-10)
- First touch files:
	- `StoryboardDesigner.App.Tests/` and `Storyboard.Shared.Tests/` (new and updated tests)
	- Any snapshot/round-trip fixture files already used by serialization tests
- Checklist:
	- Add explicit shared-variable JSON shape guard regression.
	- Add representative round-trip serialization checks for shared variables.
	- Run full solution test suite before Phase 3 close.

## Phased/Sliced Execution Plan

### Slice A - Phase 0A Baseline Coverage

Goal: lock behavior and add missing guardrails before runtime internals change.

Entry criteria:

1. Lock decisions SV-01 through SV-10 captured.

Deliverables:

1. Shared-runtime tests that pin current read/write and share-propagation external behavior.
2. Repro test for `SetGameProperty` failing `self.*` target resolution.
3. Baseline diagnostics assertions for failure paths.

Exit criteria:

1. New baseline tests pass.
2. No behavior changes merged yet.

### Slice B - Phase 0B Designer Validation Warnings

Goal: add additive warnings for invalid shared-group authoring states.

Entry criteria:

1. Slice A complete.

Deliverables:

1. Warning rule: mismatched initial values in same share.
2. Warning rule: mismatched value restrictions/types in same share.
3. Validation tests covering both rules and ignore/suppression behavior.

Exit criteria:

1. Warnings emitted correctly.
2. No designer model or JSON schema shape changes.

### Slice C - Phase 1 Shared Value-Cell Authority

Goal: move share authority to canonical value cells while retaining participant variable presence.

Entry criteria:

1. Slice A and Slice B complete.

Deliverables:

1. Canonical shared value-cell map in session internals.
2. Deterministic initialization precedence with diagnostics for mismatch.
3. Participant read/write delegation to canonical share value.
4. Unbind behavior: participant becomes independent.

Exit criteria:

1. Shared participant reads/writes are consistent from any participant endpoint.
2. Existing consumer APIs unchanged.

### Slice D - Phase 2 Common Scope Resolver Extraction

Goal: centralize scope-chain token resolution for reuse.

Entry criteria:

1. Slice C complete.

Deliverables:

1. New shared resolver component for scope-chain tokens.
2. Alias parity support: `self`, `room`, `area`, `country`, `planet`, `global`, `player`.
3. Action resolver composition preserving action-only extensions.

Exit criteria:

1. Existing chooser/echo scope resolution parity preserved.
2. No action-only leakage into core resolver.

### Slice E - Phase 3 SetGameProperty Unification

Goal: route SetGameProperty target resolution through the common resolver.

Entry criteria:

1. Slice D complete.

Deliverables:

1. Token normalization coverage: raw, brace-wrapped, quoted wrapped.
2. `SetGameProperty` resolver path replaced with common scope resolver.
3. Diagnostics-level gated unresolved-target details.

Exit criteria:

1. `self.isCut` scenario passes.
2. Shared-binding write-through via set-property passes.

### Slice F - Phase 4 Cleanup and Signoff

Goal: remove transitional duplication and complete proof gates.

Entry criteria:

1. Slice E complete.

Deliverables:

1. Remove obsolete propagation-only helper paths where superseded.
2. Shared-variable JSON shape guard test added and passing.
3. Representative serialization round-trip tests added and passing.
4. Full test suite run completed and green.

Exit criteria:

1. Zero-drift proof complete.
2. Plan closure-ready status confirmed.

## Initial Work Package (proposed first slice)

1. Add Phase 0 tests around current shared binding behavior and `SetGameProperty self.*` failure repro.
2. Introduce canonical shared backing in session internals with compatibility behavior retained.
3. Validate focused regression gates before resolver extraction starts.
