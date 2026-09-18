# Schema Property To Runtime GameProperty Control

## Goal

Introduce a schema-driven mechanism (new x-hint) that controls which authored/schema properties are projected into runtime `gameProperties` during clean export, replacing hardcoded exporter rules where practical.

## Problem Statement

Current runtime projection behavior for fields such as `positionX` and `positionY` is embedded directly in exporter code paths (for example, `UpsertCleanNumericGameProperty(...)` in `JsonExportService`).

This works, but it has drawbacks:

- Low discoverability: transform rules are hidden in code.
- Limited governance: schema authors cannot see or adjust projection intent in contracts.
- Drift risk: schema and exporter transform behavior can diverge.

## Desired Outcome

A declarative schema hint that lets export logic determine, per property, whether and how to project a value into runtime `gameProperties`.

## Proposed Hint Family (Draft)

Use one or more extension properties on schema fields (names TBD):

- `x-runtime-gameproperty-export`: boolean/object gate for projection.
- `x-runtime-gameproperty-name`: explicit runtime variable name.
- `x-runtime-gameproperty-type`: numeric/string/bool/enum coercion mode.
- `x-runtime-gameproperty-sanitizer`: finite/clamp/default normalization strategy.
- `x-runtime-gameproperty-lifetime`: runtime lifetime override when needed.

These names are placeholders and should be finalized with consistency checks against existing x-hint naming patterns.

## Scope

Initial scope:

- `ProjectGameObjectDto`-related mappings only.
- Existing hardcoded upserts for `positionX`, `positionY`, `imageRotationDegrees`, and `imageScale`.

Out of scope (first pass):

- Broad rewrite of all runtime export transformations.
- Runtime command processor behavior changes.

## Implementation Approach (Phased)

1. Define hint contract
- Choose final x-hint names and allowed value shapes.
- Document defaults and fallback behavior when hints are absent.

2. Add schema metadata
- Add hints to relevant properties in core/designer schemas.
- Keep behavior unchanged initially by matching current hardcoded semantics.

3. Add exporter rule resolver
- Implement a small projection resolver that reads schema hints and emits runtime `gameProperties`.
- Keep old hardcoded path behind a temporary fallback for safe migration.

4. Flip to schema-led projection
- Route target properties through hint resolver.
- Remove equivalent hardcoded special cases once parity is verified.

5. Guardrails and regression tests
- Add focused tests proving hint-driven projection matches current output.
- Add negative tests for missing/invalid hint data.

## Validation Plan

- `dotnet build .\StoryboardDesigner.slnx`
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`
- `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`

## Risks

- Contract/behavior drift if hints are partially applied.
- Over-generalization of hint format before real usage patterns are proven.
- Backward compatibility concerns for existing sample/runtime baselines.

## Decision Notes To Capture During Execution

- Final hint naming and shape.
- Whether fallback path remains temporary or permanent.
- Which properties remain intentionally code-driven even after hint support.

## Exit Criteria

- Target runtime game property projections are schema-declared and exported through hint logic.
- Existing runtime/export tests remain green.
- Hardcoded duplicates for migrated properties are removed.
