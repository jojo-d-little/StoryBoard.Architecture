# Object Edit Dialog Consistency and Adapter Shim Plan

Status: Draft (planning only, no implementation in this slice)
Owner: StoryboardDesigner.App authoring workflows
Last updated: 2026-07-28

## 1. Purpose

Define one consolidated plan to eliminate object-edit persistence drift across launch entry points by combining:

1. Consistent canonical apply behavior.
2. A single object-to-request adapter/factory shim for dialog request construction.

The goal is one source of truth for object edit request building and one source of truth for applying edits.

## 2. Problem Statement

Object-edit dialogs are value editors that return updated request objects. Today, both request construction and persistence behavior are split across multiple caller-specific orchestration paths.

Recurring failure pattern:

1. A new object field is added.
2. One call site forgets to map it into the request or apply path.
3. The field appears editable but does not persist in some contexts.

Observed examples include entry-point differences between Room Designer, Project Explorer, global objects, and template objects.

## 3. Scope and Non-Goals

In scope:

1. Canonical object-to-request projection path (adapter/factory shim).
2. Canonical request-to-object apply path.
3. Explicit policy modes for contextual differences.
4. Regression tests for parity across openers and policy modes.
5. Dialog-aware editability metadata driven by policy.

Out of scope:

1. Dialogs directly mutating GameObject model instances.
2. Storyboard.Shared runtime contract redesign.
3. Broad UX redesign unrelated to persistence consistency.

## 4. Current Architecture Summary

1. Dialogs return ObjectBasicPropertiesEditRequest values.
2. Multiple ViewModel entry points construct request objects manually.
3. Multiple ViewModel entry points apply updates with duplicated but non-identical logic.
4. Linked-instance and scope policies are enforced in orchestrators, but not uniformly centralized.

## 5. Proposed Direction

## 5.1 Canonical Request Factory (Adapter Shim)

Introduce a single factory/adapter that takes:

1. Source object (GameObject being edited).
2. Optional effective definition source (linked or definition-owned projection paths).
3. Context policy options descriptor.

Factory output:

1. ObjectBasicPropertiesEditRequest.
2. Optional editability metadata (per-field editable/read-only and reason).

## 5.2 Canonical Apply Surface

Use one canonical apply method/service that takes:

1. Target object.
2. Updated request.
3. Same policy context used for request construction.

Responsibilities:

1. Apply shared fields in one place.
2. Enforce policy gates in explicit branches.
3. Return structured apply result (changed fields, optional diagnostics/status hints).

## 5.3 Explicit Policy Modes and Options

Use a small typed options contract (for example ObjectEditRequestBuildOptions/ObjectEditApplyOptions) rather than ad-hoc booleans.

Expected policy dimensions:

1. Mode: FullObjectEdit, RoomDesignerEdit, TemplateCatalogEdit, GlobalCatalogEdit, LinkedInstanceConstrainedEdit.
2. Include chooser variable options.
3. Include lock requirement payload.
4. Effective definition projection behavior.
5. Linked write-through and override rules.

## 5.4 Keep Dialogs as Editors

Dialogs remain value editors only. They do not directly mutate model objects.

Benefits:

1. Preserves MVVM boundaries.
2. Keeps policy and graph mutation centralized.
3. Keeps orchestration testable.

## 5.5 Dialog-Aware Editability Contract

For non-editable fields in a context:

1. Dialog should visibly disable/gray fields.
2. Optional reasons should be surfaced via tooltip/inline note.
3. Canonical apply path remains source-of-truth enforcement even if UI is bypassed.

## 6. Risk Assessment

Overall risk: Low to moderate with staged migration.

Key risks:

1. Linked-instance override behavior drift.
2. Missed side effects (dirty state, selection refresh, status text).
3. Name uniqueness and propagation behavior changes by scope.
4. Over-centralized adapter complexity.
5. UI/apply mismatch if editability metadata and apply enforcement diverge.

Mitigations:

1. Migrate one entry point at a time.
2. Preserve semantics before tightening rules.
3. Add parity and policy tests prior to wide migration.
4. Keep mode-specific internals small and documented.

## 7. Phased Plan

Phase 0 - Inventory and Contract Lock

1. Enumerate all call sites constructing ObjectBasicPropertiesEditRequest.
2. Enumerate all call sites applying object edit results.
3. Create policy matrix by entry point (editable, projected, ignored fields).
4. Lock options contract naming and mode semantics.

Phase 1 - Build Adapter Shim and Apply Surface (No Broad Migration)

1. Add canonical request factory/adapter in StoryboardDesigner.App service layer.
2. Add canonical apply pipeline method/service.
3. Add mode-based editability metadata model.
4. Add unit tests for each mode with fixture objects.

Phase 2 - Incremental Caller Migration

1. Migrate one low-risk entry point first.
2. Migrate Project Explorer object edit paths.
3. Migrate global and template catalog edit paths.
4. Migrate room designer edit path(s).
5. Remove duplicated constructor/apply blocks only after each migration passes tests.

Phase 3 - Hardening and Cleanup

1. Add guardrail tests for request field parity and apply field parity.
2. Add lightweight diagnostics (mode + changed field summary).
3. Final naming and docs cleanup.

## 8. Test Strategy

Automated tests:

1. Factory projection tests by mode.
2. Apply tests by mode for allowed vs blocked fields.
3. Parity tests for equivalent edits across entry points.
4. Linked-instance constrained behavior tests.
5. Dialog editability metadata tests (enabled/disabled states).
6. Guardrail tests for forged input on non-editable fields.
7. Regression coverage for previously missed fields (including SpatialType).
8. Roundtrip tests: object -> request -> updated request -> object.

Manual smoke matrix:

1. Edit NameInGame and NameSynonyms from Project Explorer and Room Designer paths.
2. Edit appearance fields from global/template/room contexts.
3. Verify save/reload roundtrip.
4. Verify linked constraints remain enforced and accurately reflected in UI.

Validation commands:

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
3. Focused test filters for migrated object-edit paths.

## 9. Design Decisions to Lock Before Implementation

1. Keep Room Designer edit as full edit vs scoped subset.
2. Represent policy as enum-only vs enum + flags.
3. Deliver editability metadata from same factory result vs separate provider.
4. Keep linked constrained behavior silent-normalization vs explicit disabled UI with reason.
5. Placement of canonical apply surface (ViewModel partial vs dedicated service).
6. Whether apply returns structured diff for status/diagnostics.
7. Which call site migrates first for least blast radius.

## 10. Exit Criteria

1. No remaining direct call-site construction of ObjectBasicPropertiesEditRequest for object-edit dialogs.
2. All object-edit entry points use canonical request factory and canonical apply pipeline.
3. Policy matrix is documented and enforced by tests.
4. Full build and targeted regressions pass.
5. Manual smoke matrix completes without persistence drift.

## 11. Pipeline Sequencing Notes

1. Approve policy matrix before coding.
2. Land Phase 1 factory/apply/test scaffolding first.
3. Land caller migrations in small PR slices.
4. Keep duplicate-path cleanup as final PR.

## 12. Notes

1. This document is planning-only; no broad refactor implementation is part of this consolidation slice.
2. Existing tactical bug fixes remain interim stability patches until full migration completes.
