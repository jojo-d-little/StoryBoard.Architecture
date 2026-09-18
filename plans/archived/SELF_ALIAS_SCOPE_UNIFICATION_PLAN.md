# SELF_ALIAS_SCOPE_UNIFICATION_PLAN

Last updated: 2026-07-15
Status: Implemented (Archive Ready)
Owner: Copilot + maintainer review

## Plan Maintenance (2026-07-19)
1. Completed slices A-D remain accurate and validated.
2. Slice E remains intentionally deferred pending room multi-image support.
3. Q9 remains intentionally deferred with Slice E and does not block closure of current-scope work.
4. Plan is closed for current scope and ready for archive.

## Purpose
Unify `self.*` reference behavior across script use cases so `self` is owner-scope based (not action-centric), while preserving existing scope aliases and runtime compatibility.

## Problem Statement
Current behavior is split:
1. Action scripts get `self.*` through action execution-scope enrichment.
2. Image chooser scripts inject object aliases through a separate path.
3. This creates drift risk and makes future non-object chooser owners (for example room image choosers) harder to support consistently.

## Goals
1. Define one shared `self` enrichment contract in `Storyboard.Shared`.
2. Make both action scripts and chooser scripts use the same enrichment implementation.
3. Keep existing explicit aliases (`room.*`, `objectName.*`, etc.) functional.
4. Support owner-scope expansion for future room-owned chooser scripts.

## Non-Goals
1. Do not redesign script syntax.
2. Do not remove or rename existing reference tokens.
3. Do not change unrelated command/action execution flow.

## Contract Decisions (Lock)
1. `self` means the scope owner of the current script evaluation context.
2. `self.name`, `self.objectName`, and `self.description` are always present (empty defaults allowed).
3. If owner is `GameObject`, `self.<variable>` aliases map to object variables.
4. If owner is `Room` (future chooser owner), `self.<variable>` aliases map to room variables.
5. Existing canonical scope aliases remain valid and unchanged (`room.*`, `area.*`, `country.*`, `planet.*`, object-name-prefixed aliases).
6. Action-property tokens remain action-pipeline specific and are out of scope for `self` enrichment.

## Lock-Off Questions (Decision Queue)
These are implementation-critical details that should be locked before coding slices B-E.

### Locked So Far
1. Q1 locked (2026-07-14): `self` is always owner-scope based, regardless of script type.
2. Q2 locked (2026-07-14): Canonical explicit aliases are authoritative; `self` enrichment is additive and does not overwrite non-`self` keys.
3. Q3 locked (2026-07-14): `self.objectName` is empty when owner scope is not `GameObject`.
4. Q4 locked (2026-07-14): `self.nearByQuantity` is synthesized only for `GameObject` owner scope when quantifiable semantics apply.
5. Q5 locked (2026-07-14): If owner scope is unresolved, emit default empty scalar aliases (`self.name`, `self.objectName`, `self.description`) and do not synthesize `self.<variable>` aliases.
6. Q6 locked (2026-07-14): Chooser evaluation uses the shared self-enricher path for current object-owned chooser execution.
7. Q7 locked (2026-07-14): Template chooser editing includes object self aliases (for example `self.isOpen`) without requiring room context.
8. Q8 locked (2026-07-14): Acceptance floor tests passed for object-owner action path, object-owner chooser path, template chooser discovery, and default-self fallback coverage.

1. Owner semantics for `self` across all script surfaces.
- Question: Is `self` always owner-scope based, regardless of script type?
- Options:
	- A) Yes, always owner-scope based.
	- B) Keep action-specific behavior and chooser-specific behavior distinct.
- Recommended: A.

2. Alias precedence and overwrite policy.
- Question: Can `self` enrichment overwrite existing non-`self` canonical aliases?
- Options:
	- A) No; canonical aliases (`room.*`, `area.*`, object-name aliases) remain source of truth.
	- B) Yes; `self` enrichment may overwrite collisions.
- Recommended: A.

3. `self.objectName` when owner is not a `GameObject`.
- Question: What value should be used for `self.objectName` on non-object owners (for example room)?
- Options:
	- A) Empty string.
	- B) Owner scope name.
- Recommended: A.

4. `self.nearByQuantity` applicability.
- Question: Should `self.nearByQuantity` be synthesized outside `GameObject` owner scopes?
- Options:
	- A) No; only for `GameObject` owner when quantifiable logic applies.
	- B) Yes; define cross-scope semantics.
- Recommended: A.

5. Missing-owner default behavior.
- Question: If owner scope cannot be resolved, what should be emitted?
- Options:
	- A) Emit default empty `self` scalar aliases (`self.name`, `self.objectName`, `self.description`) only.
	- B) Emit no `self` aliases.
- Recommended: A.

6. Chooser path migration constraint.
- Question: Must chooser evaluation use the exact shared self-enricher path (no parallel alias shaping)?
- Options:
	- A) Yes, single shared enricher only.
	- B) Allow chooser-specific alias shaping.
- Recommended: A.

7. Template chooser discovery floor.
- Question: For template object chooser editing, what minimum token availability is required?
- Options:
	- A) Must include object self aliases (for example `self.isOpen`) even with no room context.
	- B) Only global aliases unless room context exists.
- Recommended: A.

8. Test acceptance floor for closure.
- Question: What minimum test set must pass before closing this plan?
- Options:
	- A) Object-owner action + object-owner chooser + template chooser + default-self fallback tests.
	- B) Action-only tests.
- Recommended: A.

9. Missing runtime value diagnostics behavior.
- Question: Should runtime diagnostics explicitly report when a referenced token is known but not present in the runtime reference-values map?
- Options:
	- A) Yes; emit diagnostics in High diagnostics mode only, while preserving current empty-string substitution behavior.
	- B) No; keep silent empty-string substitution only.
- Recommended: A.

## Design Approach
1. Add a shared self-alias enricher in `Storyboard.Shared/GameServices/References`.
2. Enricher input is a context record (owner scope kind + owner metadata + optional runtime scope node + base reference map).
3. Enricher delegates core alias shaping to existing alias helper logic to avoid duplication.
4. Enricher supports optional extras currently tied to object ownership (for example `self.nearByQuantity` where applicable).

## Implementation Slices

### Slice A - Shared Contract and Enricher
1. Add context model for self-alias enrichment.
2. Add shared enricher service/function with deterministic precedence rules.
3. Keep behavior equivalent to current object self alias output for object owners.

Status: Completed (2026-07-14).

### Slice B - Action Path Migration
1. Update action self-alias resolver to call shared enricher.
2. Preserve action-property token resolver ordering and behavior.
3. Confirm no regression in action echo/script references.

Status: Completed (2026-07-14). Runtime-focused guardrail suite passed after migration.

### Slice C - Chooser Path Migration
1. Update image chooser script evaluation path to call shared enricher.
2. Remove chooser-specific alias shaping drift points where possible.
3. Keep fallback/default variant behavior unchanged.

Status: Completed for object-owned chooser path (2026-07-14). Room-owned chooser integration remains future-facing (Slice E).

### Slice D - Designer Variable Discovery Follow-Through
1. Ensure chooser variable-choice building for template and room objects uses scope-node-based discovery consistently.
2. Ensure template object editing path receives chooser variable choices/tokens.
3. Preserve behavior for room object editing.

Status: Completed (2026-07-14). Verified with `QuantifiableRenamePropagationTests`, including `EditObjectBasicProperties_TemplateObject_IncludesChooserVariableChoicesAndSelfAliasTokens`.

### Slice E - Future-Proof Hook for Room-Owned Chooser Scripts
1. Add tests and plumbing so room-owner context can project `self.*` without special-case rewrites later.
2. No UI for room chooser authoring required in this slice unless already available.

Status: Deferred by product decision (2026-07-14). Hold until room multi-image support is implemented. Current runtime chooser entry point is object-only (`TryResolveVariantImagePath(GameStateScopeNode objectNode, ...)`) and does not yet evaluate room-owned chooser scripts.

## Test Plan

### Runtime/Shared Focus
1. Add/extend tests validating `self.*` for object-owner action scope.
2. Add/extend tests validating chooser evaluation receives identical `self.*` semantics.
3. Add test coverage for owner-scope fallback defaults.

### Designer Focus
1. Add test ensuring template object chooser variable list includes `self.isOpen` when available.
2. Add test ensuring room object chooser list still includes expected ancestor-scope options.

## Validation Commands
1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`
4. Runtime-focused gate during shared changes:
`dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`

## Exit Criteria
1. One shared self-alias enrichment implementation is used by both action and chooser script evaluation paths.
2. No regression in existing action script token behavior.
3. Template chooser and existing object-owned chooser experiences are functional and consistent; room-owned chooser support is explicitly deferred.
4. Build and targeted regressions pass.

## Risks and Mitigations
1. Risk: Token precedence drift.
- Mitigation: Lock ordering and add direct token-map assertions in tests.
2. Risk: Hidden regressions in chooser fallback behavior.
- Mitigation: Keep chooser fallback logic untouched and test around script failure/unknown variant cases.
3. Risk: Over-coupling designer/runtime concerns.
- Mitigation: Keep alias enrichment in Shared; keep UI/discovery orchestration in Designer.
