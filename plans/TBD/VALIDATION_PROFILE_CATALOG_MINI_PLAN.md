# Validation Profile Catalog Mini Plan

Status: Planned (Deferred)
Owner: StoryboardDesigner.App validation execution
Last updated: 2026-07-03

## 1. Purpose

Define a small, low-risk path to introduce profile-based validation routing without broad validation-engine redesign.

## 2. Why This Is Separate

The full validation management plan includes profile catalog and result metadata expansion, but this mini plan isolates only the profile-routing slice so it can be implemented later in a single focused pass.

## 3. Scope (In)

1. Add profile selection to validation execution request.
2. Add a profile catalog that maps profile to deterministic include/exclude policy.
3. Route Save and ExportGate workflows through profile-based rule selection.
4. Add tests proving deterministic rule selection per profile.

## 4. Scope (Out)

1. No severity remap policy changes in this slice.
2. No skipped-rule metadata expansion in this slice.
3. No UI redesign for profile selection.
4. No new validation rule families.

## 5. Minimum Target Design

## 5.1 Request Contract

Add profile to execution request:

1. ValidationExecutionRequest.Profile (default Save for current save/validate flow).

## 5.2 Profile Catalog

Create a catalog (execution-layer owned) that defines:

1. Profile id.
2. IncludeRuleIds and/or ExcludeRuleIds.
3. Default completion mode per profile.

Initial profiles to wire:

1. Save
2. ExportGate
3. ScopedAuthoring

## 5.3 Engine Orchestration

1. Engine resolves effective profile config before executing rules.
2. Engine executes only profile-eligible rules, preserving registration order.
3. Existing behavior remains equivalent for Save unless intentionally changed.

## 6. Implementation Steps

1. Add Profile field to ValidationExecutionRequest.
2. Add ValidationProfileCatalog type in Validation/Execution.
3. Add profile filter pass in ValidationEngine before rule execution.
4. Update Save and Export callsites to pass explicit profile.
5. Add/adjust tests:
   - profile selection tests,
   - save/export profile integration tests,
   - deterministic execution order assertions.

## 7. Acceptance Criteria

1. Save path uses explicit Save profile.
2. Export gate uses explicit ExportGate profile.
3. Profile filtering is deterministic and test-covered.
4. No regressions in existing validation behavior unless intentionally documented.
5. Full test suite remains green.

## 8. Validation Commands

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "ValidationEngineRegistrationTests|JsonExportServiceTraversalValidationTests"

## 9. Defer Notes

When this mini plan starts, align it back to:

1. plans/future/VALIDATION_RULES_MANAGEMENT_PLAN.md (Infrastructure Completion Checklist items on profile catalog and request routing).
2. Potential future enhancement: add explicit ignore-scope semantics that can be configured as either NodeOnly (current behavior) or InheritedDownTree for selected workflows/rules.
3. If inherited ignore support is added later, require explicit opt-in to avoid changing current authoring expectations; cover with regression tests for project-level global ignores, node-only ignores, and inherited-node ignores.
