# Materialization and Composite Quantifiable Improvement Plan

Status: Future (not active)
Owner: TBD
Last updated: 2026-07-09

## Goal

Improve runtime behavior and authoring clarity for quantifiable objects during materialization and composite building, especially when deciding whether to:

1. increase quantity on an existing active object, or
2. materialize an additional instance.

## Why This Plan Exists

Recent work improved quantifiable materialization behavior, but there is still an open design question around composite build flows:

1. what should happen when a composite target is already active,
2. when should build operations increment quantity vs create a new instance, and
3. whether runtime still retains meaningful base-item identity and scope provenance (including area-level base catalogs).

## Current Known Baseline

1. Runtime supports linked base lineage fields (`LinkedBaseObjectId`, `LinkActionsToBaseObject`) on runtime scope nodes.
2. Quantifiable distribution mode (`GroupedStack` vs `IndividualInstances`) can influence materialization behavior.
3. Composite build currently has unresolved policy gaps when target objects are already active and quantifiable.
4. Base objects can originate from multiple catalog tiers (Global, Planet, Country, Area) in designer authoring.

## Problem Statement

The system needs deterministic, testable policy for quantifiable object growth across materialize/build-composite actions so behavior is predictable for players and authors.

A second unresolved concern is observability and contract clarity for base-item lineage in runtime:

1. Is there still a concept of base item after mapping?
2. If yes, what exact identity is retained?
3. Is source catalog tier (for example Area Base Objects) retained or lost?

## Objectives

1. Define a unified decision policy for quantifiable growth:
- quantity increment
- instance-count increment
2. Align MaterializeObjectCopy and BuildCompositeByParts behavior so they do not diverge unexpectedly.
3. Make lineage behavior explicit for linked/base objects from all scope tiers.
4. Decide whether runtime needs explicit base-origin metadata (beyond linked id) for policy and diagnostics.
5. Preserve host boundaries and avoid Designer-Simulator coupling drift.

## Non-Goals (This Planning Slice)

1. No immediate runtime contract/schema breaking change.
2. No UI redesign commitment yet.
3. No migration execution yet.

## Workstream A: Quantifiable Growth Policy

### A1. Define policy matrix

Create a matrix for action type x object capability x distribution mode x active state.

Minimum rows to define:

1. MaterializeObjectCopy + Quantifiable + GroupedStack
2. MaterializeObjectCopy + Quantifiable + IndividualInstances
3. BuildCompositeByParts + Quantifiable target already active
4. BuildCompositeByParts + non-quantifiable target already active

### A2. Choose default growth rule

Decide and document preferred default:

1. GroupedStack: favor quantity increase.
2. IndividualInstances: favor new instance materialization.
3. Mixed/ambiguous cases: explicit fallback and result codes.

### A3. Align composite builder behavior

Evaluate design options:

1. fail when target active (current behavior),
2. materialize a new target when active and quantifiable,
3. allow policy-driven selection by distribution mode.

### A4. Result-code and diagnostics strategy

Define clear result code semantics for:

1. policy branch chosen,
2. materialize fallback attempted,
3. materialize fallback failed,
4. success via quantity increase,
5. success via new instance creation.

## Workstream B: Base-Item Lineage and Scope-Origin Investigation

### B1. End-to-end lineage trace spike

Trace one object from each catalog tier into runtime:

1. Global base object
2. Planet base object
3. Country base object
4. Area base object

For each trace, capture:

1. authoring identity fields,
2. mapped runtime node fields,
3. what survives after runtime materialization,
4. what is available to action execution.

### B2. Determine if runtime keeps base concept meaningfully

Answer explicitly:

1. Is linked base id sufficient to represent base concept?
2. Is base-scope tier provenance required for expected gameplay/editor semantics?
3. If tier is not retained, does any current feature need it now or soon?

### B3. Evaluate metadata options

If needed, evaluate additive metadata options:

1. runtime base-origin tier enum,
2. runtime base-origin scope id/path,
3. lightweight provenance token for diagnostics only.

### B4. Decision checkpoint

Produce a short decision note:

1. keep current lineage model, or
2. add explicit base-origin metadata.

## Workstream C: Composite Build and Break Provenance Robustness

1. Verify provenance recording for quantifiable consumption remains correct when target instance is newly materialized.
2. Verify break-composite restore uses the correct target instance id and does not leak to sibling instances.
3. Decide whether any provenance contract needs additive strengthening.

## Candidate Implementation Slices (Future)

1. Slice 1: Policy matrix and decision doc only (no code changes).
2. Slice 2: Runtime executor behavior alignment for BuildCompositeByParts active-target policy.
3. Slice 3: Optional lineage metadata addition if Workstream B requires it.
4. Slice 4: Diagnostics/result-code and regression hardening.

## Validation Strategy (When Work Starts)

Baseline gates:

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"

Targeted tests to add or expand:

1. BuildCompositeByParts on active quantifiable target follows policy (quantity vs new instance).
2. GroupedStack remains quantity-centric where intended.
3. IndividualInstances creates new runtime instances where intended.
4. BreakComposite restores the correct parts for the correct composite instance.
5. Materialized instances retain intended lineage references.
6. Area-level base objects can be traced through runtime snapshot and action execution context.

## Risks and Mitigations

Risk: hidden policy divergence between materialize and composite build actions.
Mitigation: single policy matrix and shared helper where practical.

Risk: accidental runtime/designer coupling while surfacing base provenance.
Mitigation: keep runtime metadata host-neutral in Shared; keep UI handling in hosts.

Risk: regression in existing quantifiable scenarios.
Mitigation: focused runtime regression suite and targeted fixture-based tests.

## Open Questions

1. Should BuildCompositeByParts ever mutate an already active target, or always require a fresh target in some modes?
2. If a target is quantifiable and active, should policy key off distribution mode alone, or include per-action override settings?
3. Does runtime need to distinguish between source catalogs (Global/Planet/Country/Area), or is linked identity enough?
4. If scope-origin is needed, should it be identity-level metadata or diagnostics-only metadata?

## Notes

1. This is a future planning document only; no implementation is committed in this file.
2. This plan should be coordinated with existing future plans on instance handling and composite recipe relationship decisions.
