# Room Display Name And Transition Presentation Handoff Plan

Last updated: 2026-09-11
Status: Closed and archived (2026-09-11)

Purpose: introduce room-level player-facing naming and per-traversal transition presentation overrides while preserving existing fallback defaults and host/runtime boundaries.

## Problem Statement

Producers need to distinguish authored room identities from player-facing text and need targeted control of traversal transition visuals for specific room-to-room movement. Today rooms only expose a single name field, and transition behavior is largely web-portal-config default driven rather than authored per traversal.

## Non-Goals

1. Replace or remove existing transition defaults in web portal settings.
2. Build a large transition authoring system with full parameter editing (duration, easing, layering, etc.) in this slice.
3. Add hidden parser/runtime fallback vocabulary or implicit command grammar behavior.
4. Couple host projects to designer file formats or project storage internals.

## Current Baseline (Observed)

1. Room authoring supports only one room name, unlike objects that can separate authored identity from in-game display naming.
2. Traversal transition defaults live in web portal configuration and are applied without authored per-traversal override metadata.
3. Designer and runtime contracts do not carry an explicit traversal presentation effect key.
4. Runtime-host communication does not expose traversal-level transition presentation metadata for host-driven rendering choices.

## Proposed Shape

1. Add room-level `nameInGame` support as an optional authored field (with clear runtime mapping and deterministic fallback to existing room name when unset).
2. Extend traversal schema/contracts with optional `presentationEffectKey` metadata for transition selection.
3. Add designer UX to edit traversal presentation effect key outside the traversal wizard (to avoid overloading wizard complexity).
4. Propagate traversal presentation metadata through shared runtime and host-facing transport interfaces.
5. Keep web portal `roomTransitionDefaults` as fallback when traversal-level override is absent or invalid.

## Initial Scope Breakdown

### A) Contract And Runtime Surface

1. Extend schema and generated contract DTOs for room `nameInGame` and traversal `presentationEffectKey`.
2. Update shared runtime models/mapping and export/import paths.
3. Keep additive, non-breaking shape where possible.

Estimated effort: Medium.

### B) Designer Authoring UX

1. Add room `nameInGame` authoring and validation behavior.
2. Add traversal-level transition presentation editing in a non-wizard UX surface.
3. Preserve MVVM boundaries and keep behavior deterministic.

Estimated effort: Medium.

### C) Host Transport And Web Portal Consumption

1. Extend host-facing payload/interface for traversal presentation effect key.
2. Consume traversal override in web portal transition selection with fallback to existing defaults.
3. Preserve backward compatibility for projects without overrides.

Estimated effort: Medium.

## Structured Delivery Order And Session Handoffs (Locked)

Implementation order is fixed for this workstream:

1. Stage 1: Contracts And Shared Runtime Mapping (Backwards-Compatible)
2. Stage 2: Designer Authoring UX
3. Stage 3: GameEngine Runtime Integration
4. Stage 4: Host Interface And Web Portal Runtime Consumption
5. Stage 5: Simulator Host Parity
6. Stage 6: Contract Retirement (Non-Backwards-Compatible)
7. Stage 7: Regression Hardening And Closeout

Each stage must end with a formal handoff markdown document before downstream work begins.

Stage boundary policy for this workstream:
1. Each stage has a hard edit allowlist. Do not edit outside that stage allowlist.
2. Edit implies read. Paths in edit scope do not need to be repeated in read scope.
3. Reads should stay inside the stage read allowlist except brief targeted validation/debug reads.
4. Any required out-of-stage edit must be treated as a stop-and-review boundary decision.

## Stage Inclusion Matrix

| Stage ID | Stage Name | Inclusion (Required/Optional/Skipped) | Reason (if not Required) | Boundary Override (if any) |
| --- | --- | --- | --- | --- |
| 01 | Contracts And Shared Runtime Mapping (Backwards-Compatible) | Required |  | None |
| 02 | Designer Authoring UX | Required |  | None |
| 03 | GameEngine Runtime Integration | Required |  | None |
| 04 | Host Interface And Web Portal Runtime Consumption | Required |  | None |
| 05 | Simulator Host Parity | Required |  | None |
| 06 | Contract Retirement (Non-Backwards-Compatible) | Optional | Run only if retirement targets remain | None |
| 07 | Regression Hardening And Closeout | Required |  | None |

Current progress:
- Stage 1 (Contracts And Shared Runtime Mapping Backwards-Compatible): Complete.
- Stage 2 (Designer Authoring UX): Complete.
- Stage 3 (GameEngine Runtime Integration): Complete.
- Stage 4 (Host Interface And Web Portal Runtime Consumption): Complete.
- Stage 5 (Simulator Host Parity): Complete (no-op implementation; validation complete).
- Stage 6 (Contract Retirement Non-Backwards-Compatible): Complete (no-op retirement stage; no approved retirement candidates).
- Stage 7 (Regression Hardening And Closeout): Complete.
- Workstream status: Complete and archive-ready.

### Stage 1: Contracts And Shared Runtime Mapping (Backwards-Compatible)

Goal:
- Add additive schema/contract shape for room `nameInGame` and traversal `presentationEffectKey`.
- Do not perform removals/renames/requiredness tightening/type narrowing.

Codebase context:
1. Primary projects in scope: `Storyboard.Shared.Contracts`, `Storyboard.SchemaCodegen`.
2. Key files expected first: runtime contract schemas for room/traversal DTOs and generated `*_contract.cs` artifacts.
3. Boundary constraints: additive-only contract updates and no manual edits to emitted DTOs outside schema+codegen flow.
4. Required validation/tests before handoff:
   - `dotnet test .\Storyboard.TransportCodegen.Tests\Storyboard.TransportCodegen.Tests.csproj`
5. Allowed read scope: `plans/**`, `Storyboard.TransportCodegen.Tests/**`.
6. Allowed edit scope: `Storyboard.Shared.Contracts/**`, `Storyboard.SchemaCodegen/**`, `plans/active/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_PLAN.md`, `plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_01_CONTRACT_RUNTIME_HANDOFF.md`.

Primary output handoff document:
- plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_01_CONTRACT_RUNTIME_HANDOFF.md

Required handoff contents:
- Scope completed
- Files changed
- Contract/interface impact
- Validation commands executed
- Test results
- Behavioral notes
- Known issues/risks
- Explicit next-stage start checklist
- Stage boundary allowlist snapshot (read/edit)
- Boundary compliance report (out-of-scope reads/edits and approvals)
- Compatibility mode declaration

Downstream usage:
- Stage 2 starts from this handoff.

### Stage 2: Designer Authoring UX

Goal:
- Enable producer authoring workflows for room `nameInGame` and traversal `presentationEffectKey` without adding complexity to traversal wizard MVP.

Codebase context:
1. Primary projects in scope: `StoryboardDesigner.App`.
2. Key files expected first: room edit viewmodels/views, traversal property editing surfaces, designer mapping and save/load plumbing for new fields.
3. Boundary constraints: preserve strict MVVM boundaries; avoid runtime-host coupling; keep wizard flow stable by placing traversal presentation editing in an alternative UX surface.
4. Required validation/tests before handoff:
   - `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "ArchitectureSeparationGuardrailsTests|GameSimulatorPlaybackRegressionTests|GameManagerTests"`
   - any new/updated focused tests for room naming and traversal property editing behavior.
5. Allowed read scope: `plans/**`, `Storyboard.Shared.Contracts/**`.
6. Allowed edit scope: `StoryboardDesigner.App/**`, `StoryboardDesigner.App.Tests/**`, `StoryboardDesigner.App.SmokeTests/**`, `plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_02_DESIGNER_HANDOFF.md`.

Primary output handoff document:
- plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_02_DESIGNER_HANDOFF.md

Required handoff contents:
- Scope completed
- Files changed
- Contract/interface impact
- Validation commands executed
- Test results
- Behavioral notes
- Known issues/risks
- Explicit next-stage start checklist
- Stage boundary allowlist snapshot (read/edit)
- Boundary compliance report (out-of-scope reads/edits and approvals)
- Compatibility mode declaration

Downstream usage:
- Stage 3 starts from this handoff.

### Stage 3: GameEngine Runtime Integration

Goal:
- Integrate room `nameInGame` and traversal `presentationEffectKey` into runtime bootstrap/session behavior and host-facing payload composition in the GameEngine boundary.

Codebase context:
1. Primary projects in scope: `Storyboard.GameEngine`, `Storyboard.Shared` (runtime mapping seams only as required).
2. Key files expected first: clean export/runtime bootstrap readers/builders, runtime snapshot mappers, room-change presentation payload composition paths.
3. Boundary constraints: no host UI logic in engine; maintain runtime/contract compatibility for projects missing new optional fields.
4. Required validation/tests before handoff:
   - `dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj`
   - `dotnet test .\Storyboard.GameClient.Tests\Storyboard.GameClient.Tests.csproj --filter "FullyQualifiedName~GameHostProgram_StaticClient"`
   - `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests"`
5. Allowed read scope: `plans/**`, `Storyboard.Shared.Contracts/**`, `StoryboardDesigner.App.Tests/**`, `Storyboard.GameClient.Tests/**`.
6. Allowed edit scope: `Storyboard.GameEngine/**`, `Storyboard.GameEngine.Tests/**`, `Storyboard.Shared/**`, `Storyboard.GameHost/**`, `Storyboard.GameClient.Tests/**`, `plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_03_GAMEENGINE_HANDOFF.md`.

Primary output handoff document:
- plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_03_GAMEENGINE_HANDOFF.md

Required handoff contents:
- Scope completed
- Files changed
- Contract/interface impact
- Validation commands executed
- Test results
- Behavioral notes
- Known issues/risks
- Explicit next-stage start checklist
- Stage boundary allowlist snapshot (read/edit)
- Boundary compliance report (out-of-scope reads/edits and approvals)
- Compatibility mode declaration

Downstream usage:
- Stage 4 starts from this handoff.

### Stage 4: Host Interface And Web Portal Runtime Consumption

Goal:
- Surface traversal `presentationEffectKey` to host consumers and apply it in transition selection logic with existing defaults as fallback.

Codebase context:
1. Primary projects in scope: `Storyboard.Shared` (host-facing interfaces/adapters only as needed), `Storyboard.WebPortal`.
2. Key files expected first: host-facing route/interface contracts in shared surfaces and web portal transition resolver logic with settings-aware fallback paths.
3. Boundary constraints: host must not read designer/export/project files directly; all data must flow through formal runtime-host APIs; default settings remain authoritative fallback.
4. Required validation/tests before handoff:
   - `dotnet test .\Storyboard.TransportCodegen.Tests\Storyboard.TransportCodegen.Tests.csproj`
   - `dotnet test .\Storyboard.WebPortal.Tests\Storyboard.WebPortal.Tests.csproj`
5. Allowed read scope: `plans/**`.
6. Allowed edit scope: `Storyboard.Shared/**`, `Storyboard.Shared.Contracts/**`, `Storyboard.WebPortal/**`, `Storyboard.WebPortal.Tests/**`, `plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_04_HOST_WEBPORTAL_HANDOFF.md`.

Primary output handoff document:
- plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_04_HOST_WEBPORTAL_HANDOFF.md

Required handoff contents:
- Scope completed
- Files changed
- Contract/interface impact
- Validation commands executed
- Test results
- Behavioral notes
- Known issues/risks
- Explicit next-stage start checklist
- Stage boundary allowlist snapshot (read/edit)
- Boundary compliance report (out-of-scope reads/edits and approvals)
- Compatibility mode declaration

Downstream usage:
- Stage 5 starts from this handoff.

### Stage 5: Simulator Host Parity

Goal:
- Ensure simulator player-facing room labeling and traversal destination display use room `nameInGame` when provided, with deterministic fallback to canonical room `name`.

Codebase context:
1. Primary projects in scope: `Storyboard.Simulator`.
2. Key files expected first: simulator room-preview labeling paths and game-state tree projection display-name mapping.
3. Boundary constraints: keep simulator independent from designer; consume only host/runtime payloads and in-memory runtime session graph.
4. Required validation/tests before handoff:
   - `dotnet test .\Storyboard.Simulator.Tests\Storyboard.Simulator.Tests.csproj`
   - `dotnet test .\Storyboard.Simulator.SmokeTests\Storyboard.Simulator.SmokeTests.csproj`
5. Allowed read scope: `plans/**`, `Storyboard.Shared/**`, `Storyboard.Shared.Contracts/**`.
6. Allowed edit scope: `Storyboard.Simulator/**`, `Storyboard.Simulator.Tests/**`, `Storyboard.Simulator.SmokeTests/**`, `plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_05_SIMULATOR_HANDOFF.md`.

Primary output handoff document:
- plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_05_SIMULATOR_HANDOFF.md

Required handoff contents:
- Scope completed
- Files changed
- Contract/interface impact
- Validation commands executed
- Test results
- Behavioral notes
- Known issues/risks
- Explicit next-stage start checklist
- Stage boundary allowlist snapshot (read/edit)
- Boundary compliance report (out-of-scope reads/edits and approvals)
- Compatibility mode declaration

Downstream usage:
- Stage 6 starts from this handoff.

### Stage 6: Contract Retirement (Non-Backwards-Compatible)

Goal:
- Retire or refactor previously-deprecated contract shape after all consumer stages are complete.
- Apply non-backward-compatible contract removals/refactors in one controlled pass.

Codebase context:
1. Primary projects in scope: `Storyboard.Shared.Contracts`, `Storyboard.SchemaCodegen`.
2. Key files expected first: retirement target schemas and regenerated/accepted contract artifacts.
3. Boundary constraints: consumer-project code removals should already be complete before Stage 6 starts.
4. Required validation/tests before handoff:
   - contract/interface guardrail suites
   - `dotnet test .\Storyboard.TransportCodegen.Tests\Storyboard.TransportCodegen.Tests.csproj`
5. Allowed read scope: `plans/**`, `Storyboard.TransportCodegen.Tests/**`.
6. Allowed edit scope: `Storyboard.Shared.Contracts/**`, `Storyboard.SchemaCodegen/**`, `plans/active/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_PLAN.md`, `plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_06_CONTRACT_RETIREMENT_HANDOFF.md`.

Primary output handoff document:
- plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_06_CONTRACT_RETIREMENT_HANDOFF.md

Required handoff contents:
- Scope completed
- Files changed
- Contract/interface impact
- Validation commands executed
- Test results
- Behavioral notes
- Known issues/risks
- Explicit next-stage start checklist
- Stage boundary allowlist snapshot (read/edit)
- Boundary compliance report (out-of-scope reads/edits and approvals)
- Compatibility mode declaration

Downstream usage:
- Stage 7 starts from this handoff.

### Stage 7: Regression Hardening And Closeout

Goal:
- Execute full regression/guardrail validation, close known gaps, and finalize workstream closeout artifacts.

Codebase context:
1. Primary projects in scope: all touched projects plus `StoryboardDesigner.App.Tests`, `Storyboard.TransportCodegen.Tests`, `Storyboard.WebPortal.Tests`.
2. Key files expected first: updated tests/snapshots/baselines and plan/handoff status sections.
3. Boundary constraints: no opportunistic refactors; focus on behavior confirmation, contract safety, and documentation completeness.
4. Required validation/tests before handoff:
   - `dotnet build .\StoryboardDesigner.slnx`
   - `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`
   - `dotnet test .\Storyboard.TransportCodegen.Tests\Storyboard.TransportCodegen.Tests.csproj`
   - `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`
5. Allowed read scope: repository-wide for validation and closeout evidence.
6. Allowed edit scope: test files, snapshot baselines, and plan/handoff documentation required for closeout evidence only.

Primary output handoff document:
- plans/active/handovers/ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_07_CLOSEOUT_HANDOFF.md

Required handoff contents:
- Scope completed
- Files changed
- Contract/interface impact
- Validation commands executed
- Test results
- Behavioral notes
- Known issues/risks
- Explicit next-stage start checklist
- Stage boundary allowlist snapshot (read/edit)
- Boundary compliance report (out-of-scope reads/edits and approvals)
- Compatibility mode declaration

Downstream usage:
- Final archival/closeout starts from this handoff.

## Risk Register

1. Contract drift risk across schema/codegen/runtime mapping.
   - Mitigation: stage contract work first, run contract/interface guardrail gate before UX/host work.
2. UX discoverability risk for traversal presentation editing if kept out of wizard.
   - Mitigation: choose a clearly discoverable traversal-details surface and add inline helper text.
3. Backward compatibility risk for existing projects without new fields.
   - Mitigation: keep new fields optional and use deterministic fallback (`name` for room display, settings defaults for transitions).
4. Effect key integrity risk (invalid keys or typos).
   - Mitigation: add lightweight validation hints/warnings in designer and safe runtime fallback when keys are unknown.
5. Runtime/host coupling risk.
   - Mitigation: constrain data flow to formal runtime-host interfaces and keep host independent from designer persistence files.

## Validation Gates

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`
3. `dotnet test .\Storyboard.TransportCodegen.Tests\Storyboard.TransportCodegen.Tests.csproj`
4. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests|SchemaCodegenHardcodedDtoGuardrailsTests|SchemaEmittedContractDriftGuardrailsTests|TransportArtifactGuardrailsTests"`
5. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "SchemaCodegenHardcodedDtoGuardrailsTests|SchemaEmittedContractDriftGuardrailsTests|ArchitectureSeparationGuardrailsTests|TransportArtifactGuardrailsTests"`
6. `dotnet test .\Storyboard.WebPortal.Tests\Storyboard.WebPortal.Tests.csproj`
7. `dotnet test .\Storyboard.GameClient.Tests\Storyboard.GameClient.Tests.csproj --filter "FullyQualifiedName~GameHostProgram_StaticClient"`
8. `dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj`
9. `dotnet test .\Storyboard.Simulator.Tests\Storyboard.Simulator.Tests.csproj`
10. `dotnet test .\Storyboard.Simulator.SmokeTests\Storyboard.Simulator.SmokeTests.csproj`

## MVP/Completion Acceptance Criteria

1. Rooms can persist and export authored `name` (identity) plus optional `nameInGame` (player-facing), and runtime display uses `nameInGame` when present.
2. Traversals can persist and export optional `presentationEffectKey` without breaking existing schema/contract consumers.
3. Designer exposes practical non-wizard UX to edit traversal `presentationEffectKey`.
4. Host/web portal runtime transition selection uses traversal override when provided and valid; otherwise uses existing `roomTransitionDefaults` behavior.
5. Existing projects with no new fields behave exactly as before.
6. Required guardrail and regression validations pass and are recorded in stage handoff artifacts.
7. Simulator room labels and traversal destination labels use `nameInGame` when present, with canonical `name` fallback.

## Open Design Questions (Resolve During Stage 2)

1. Best UX surface for traversal presentation editing:
   - traversal property panel
   - room-edge inspector
   - dedicated traversal detail flyout
2. Validation strictness for effect key input in MVP:
   - free-form string with warning-only hints
   - constrained pick-list from known web portal effect catalog
3. Whether to surface directional override pairing in MVP or keep single `presentationEffectKey` only.

## Designer UX Placement Options (Stage 2 Shortlist)

### Option 1: Extend Existing Traversal Editor Dialog (Recommended MVP)

Likely touchpoints:
1. `StoryboardDesigner.App/Views/TraversalEditorDialog.xaml`
2. `StoryboardDesigner.App/Views/TraversalEditorDialog.xaml.cs`
3. `StoryboardDesigner.App/Views/RoomTraversalsDialog.xaml`
4. `StoryboardDesigner.App/ViewModels/MainWindowViewModel.ProjectExplorer.cs` (entry workflow that opens traversal edit)

Pros:
1. Lowest adoption risk because producers already use Edit Traversal for advanced settings.
2. Keeps traversal wizard focused on connectivity and door creation.
3. Minimal navigation overhead for room-to-room fine-tuning.

Cons:
1. Dialog is already dense and can become crowded.
2. Effect-key discoverability depends on users opening traversal edit.

### Option 2: Add Traversal Properties Section In Room Traversals Grid

Likely touchpoints:
1. `StoryboardDesigner.App/Views/RoomTraversalsDialog.xaml`
2. `StoryboardDesigner.App/Views/RoomTraversalsDialog.xaml.cs`
3. Traversal edit workflow handlers in `MainWindowViewModel`.

Pros:
1. High visibility because producers can review many traversals at once.
2. Easier bulk auditing of where overrides are or are not set.

Cons:
1. Inline grid editing adds complexity to an otherwise straightforward review dialog.
2. Validation/lookup UX is harder in grid cells than in dedicated forms.

### Option 3: Add Traversal-Leg Property Card In Hierarchy Inspector

Likely touchpoints:
1. `StoryboardDesigner.App/ViewModels/Hierarchy/TraversalLegNodeViewModel.cs`
2. `StoryboardDesigner.App/Views/Controls/ProjectHierarchyPane.xaml`
3. Property panel rendering paths used by selected hierarchy nodes.

Pros:
1. Strong conceptual fit with other scope-specific property editing.
2. Could evolve naturally to per-direction overrides in future.

Cons:
1. Highest implementation complexity in this MVP.
2. Greater risk of hidden coupling with existing hierarchy-driven action/variable workflows.

Initial recommendation:
1. Use Option 1 for MVP delivery speed and low risk.
2. Revisit Option 3 if directional or richer presentation controls are added later.

## Final Closeout Checklist

1. All locked stages marked complete.
2. All stage handoff documents exist and are status-complete.
3. Validation gate outcomes recorded.
4. Deferred items listed as non-blocking follow-up.
5. Main plan updated with final status and date.
6. Archive main plan and all related stage handoffs together as one unit.