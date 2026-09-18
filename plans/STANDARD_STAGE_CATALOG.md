# Standard Stage Catalog

Last updated: 2026-09-11
Status: Active standard

Purpose: define the repository-standard stage set, default order, and default read/edit boundaries for staged workstreams.

## Usage Rules

1. New staged plans should start from this catalog.
2. The default stage order is fixed unless a plan explicitly documents why a stage is skipped.
3. Plans should decide stage inclusion per workstream using: Required, Optional, or Skipped (with reason).
4. Stage edit scope is a hard boundary.
5. Reads should stay in stage read scope except narrow validation/debug reads, which must be logged in stage handoff boundary compliance.
6. Contract changes are split by compatibility class:
- Stage 01 handles backward-compatible additive contract changes only.
- Stage 06 handles non-backward-compatible removals/refactors only, after all consumers have adapted.
7. Default deny policy: any path not listed in a stage's allowed read/edit scope is out of scope by default.
8. Edit implies read: any path in allowed edit scope is automatically allowed for read and does not need to be repeated in allowed read scope.

## Standard Stage Set (Default Order)

1. Stage 01: Contracts And Shared Runtime Mapping
2. Stage 02: Designer Authoring UX
3. Stage 03: GameEngine Runtime Integration
4. Stage 04: Host Interface And Web Portal Runtime Consumption
5. Stage 05: Simulator Host Parity
6. Stage 06: Contract Retirement (Non-Backwards-Compatible)
7. Stage 07: Regression Hardening And Closeout

## Stage Profiles

### Stage 01: Contracts And Shared Runtime Mapping (Backwards-Compatible)

Goal:
1. Add schema-first, backward-compatible contract shape and shared runtime mapping seams.
2. No contract member removals, renames, requiredness tightening, or type narrowing.

Allowed read scope (default):
1. plans/**
2. Storyboard.TransportCodegen.Tests/**

Allowed edit scope (default):
1. Storyboard.Shared.Contracts/**
2. Storyboard.SchemaCodegen/**
3. plans/active/*_PLAN.md
4. plans/active/handovers/*_01_*_HANDOFF.md

### Stage 02: Designer Authoring UX

Goal:
1. Implement authoring UX and designer-side model/mapping behavior for approved contract fields.

Allowed read scope (default):
1. plans/**
2. Storyboard.Shared.Contracts/**

Allowed edit scope (default):
1. StoryboardDesigner.App/**
2. StoryboardDesigner.App.Tests/**
3. StoryboardDesigner.App.SmokeTests/**
4. plans/active/handovers/*_02_*_HANDOFF.md

### Stage 03: GameEngine Runtime Integration

Goal:
1. Integrate approved contract fields into engine runtime bootstrap/session/behavior flows.

Allowed read scope (default):
1. plans/**
2. Storyboard.Shared.Contracts/**
3. StoryboardDesigner.App.Tests/**
4. Storyboard.GameClient.Tests/**

Allowed edit scope (default):
1. Storyboard.GameEngine/**
2. Storyboard.GameEngine.Tests/**
3. Storyboard.Shared/**
4. Storyboard.GameHost/**
5. Storyboard.GameClient.Tests/**
6. plans/active/handovers/*_03_*_HANDOFF.md

### Stage 04: Host Interface And Web Portal Runtime Consumption

Goal:
1. Surface runtime metadata to host consumers and apply host/web transition/render behavior with fallback defaults.

Allowed read scope (default):
1. plans/**

Allowed edit scope (default):
1. Storyboard.Shared/**
2. Storyboard.Shared.Contracts/**
3. Storyboard.WebPortal/**
4. Storyboard.WebPortal.Tests/**
5. plans/active/handovers/*_04_*_HANDOFF.md

### Stage 05: Simulator Host Parity

Goal:
1. Implement simulator parity for user-facing behavior based on staged runtime/host data.

Allowed read scope (default):
1. plans/**
2. Storyboard.Shared/**
3. Storyboard.Shared.Contracts/**

Allowed edit scope (default):
1. Storyboard.Simulator/**
2. Storyboard.Simulator.Tests/**
3. Storyboard.Simulator.SmokeTests/**
4. plans/active/handovers/*_05_*_HANDOFF.md

### Stage 06: Contract Retirement (Non-Backwards-Compatible)

Goal:
1. Retire or refactor previously-deprecated contract shape after all consumer stages are complete.
2. Apply non-backward-compatible changes intentionally and in one controlled pass.

Entry criteria:
1. Stages 02 through 05 are complete for impacted consumers.
2. Runtime/host/simulator paths are already adapted to additive replacement fields.
3. Retirement list is explicitly approved in plan and handoff.
4. Stage 06 performs final contract retirement only; consumer-project code removals should already be complete before this stage starts.

Allowed read scope (default):
1. plans/**
2. Storyboard.TransportCodegen.Tests/**

Allowed edit scope (default):
1. Storyboard.Shared.Contracts/**
2. Storyboard.SchemaCodegen/**
3. plans/active/*_PLAN.md
4. plans/active/handovers/*_06_*_HANDOFF.md

Validation expectation (default):
1. Contract/interface guardrail suites are required.
2. Transport codegen verification/tests are required.
3. Broad regression belongs to Stage 07 closeout after retirement is complete.

### Stage 07: Regression Hardening And Closeout

Goal:
1. Run full regressions/guardrails and complete closeout artifacts.

Allowed read scope (default):
1. Repository-wide for validation and closeout evidence.

Allowed edit scope (default):
1. Test files needed for stabilization.
2. Snapshot baselines and expected outputs.
3. Plan and stage handoff documentation.
4. plans/active/handovers/*_07_*_HANDOFF.md

## Stage Inclusion Matrix (Per Plan)

Every plan should include a stage inclusion matrix with one row per standard stage:

1. Stage ID
2. Stage Name
3. Inclusion status: Required, Optional, or Skipped
4. Reason (required when Optional or Skipped)
5. Any boundary override approved for this plan

Compatibility mode rule:
1. Every plan must declare whether Stage 06 retirement is `Required`, `Optional`, or `Skipped`.
2. If Stage 06 is skipped, the plan must list deferred retirement debt explicitly.

## Boundary Override Policy

1. Overrides are allowed only when required by the workstream.
2. Overrides must be additive-narrow, not broadening by default.
3. Each override must be documented in both the main plan and the stage handoff boundary snapshot.
4. Any override should include why the default stage boundary was insufficient.
