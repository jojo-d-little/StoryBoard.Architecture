# Room Action Editor Composable Controls Plan

Status: Proposed
Owner: StoryboardDesigner.App authoring UX
Last updated: 2026-07-05

## 1. Purpose

Refactor RoomActionEditorDialog so common fields stay in a shared shell while action-type-specific editing is provided by dedicated controls.

## 2. Why This Plan Exists

The current dialog still centralizes many action-specific concerns in one large view/code-behind surface.

Symptoms:

1. Harder to reason about per-action behavior.
2. Higher regression risk when changing one action editor.
3. UI complexity concentrated in one file.

## 3. Target Structure

Common shell remains in RoomActionEditorDialog:

1. Action Name
2. Action Type
3. Child forwarding controls
4. Inline ResultCode outcome-echo editor

Type-specific section becomes a switched host:

1. Use one host region (ContentControl) for action-specific editors.
2. Provide one dedicated UserControl per action type.
3. EchoMessage action uses no additional type-specific control.

## 4. Proposed Controls

Initial control set:

1. CheckGameProperty editor control
2. SetGameProperty editor control
3. Put/RemoveObjectInContainer editor control
4. Synonym editor control
5. NavigateDirection editor control
6. BuildCompositeByTarget editor control
7. BuildCompositeByParts editor control
8. BreakCompositeItem editor control

## 5. Migration Slices

### Slice R1: Host and Routing

1. Add action-specific host region in RoomActionEditorDialog.
2. Add action-type -> control routing map.
3. Keep existing behavior unchanged for data read/write.

Acceptance:

1. Dialog renders common shell and host region.
2. Action type switch updates hosted editor deterministically.

### Slice R2: Small Editors First

1. Extract CheckGameProperty, SetGameProperty, and Synonym editors.
2. Remove equivalent inline sections from RoomActionEditorDialog.

Acceptance:

1. Save/load parity for extracted types.
2. No behavioral change in script validation and payload mapping.

### Slice R3: Mid-Complexity Editors

1. Extract Put/RemoveObjectInContainer and NavigateDirection editors.
2. Keep existing ResultCode inline outcome editor as shared shell behavior.

Acceptance:

1. Existing mapped outcome behavior unchanged.
2. Navigate defaults and token/script editing parity preserved.

### Slice R4: Composite Editors

1. Extract BuildCompositeByTarget, BuildCompositeByParts, BreakCompositeItem editors.
2. Remove remaining type-specific inline grids from RoomActionEditorDialog.

Acceptance:

1. Composite field validation and save mapping parity preserved.
2. Recipe/target/part displays still sync correctly.

### Slice R5: Cleanup and Guardrails

1. Remove obsolete visibility-switch logic from RoomActionEditorDialog.
2. Add regression tests for host routing and per-control save behavior.
3. Keep MVVM boundaries explicit (UI mechanics in views, business rules in services).

Acceptance:

1. RoomActionEditorDialog code-behind shrinks materially.
2. Focused editor tests and build are green.

## 6. Validation

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "ActionEchoEditorEntryBuilderTests|ActionOutcomeMessageStatusFormatterTests|RuntimeActionResultCodeRegistryTests|GameCommandProcessorFixtureTests"
3. Add/extend targeted tests for new control host routing and action-specific editor save parity.

## 7. Boundary Notes

1. Scope is Designer-only unless a runtime contract gap is discovered.
2. Keep Storyboard.Simulator independent from StoryboardDesigner.App.
3. Shared runtime contracts remain in Storyboard.Shared; no host coupling changes in this refactor.
