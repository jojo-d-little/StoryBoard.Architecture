# ProjectGlobalNodeDto and RuntimeProjectDto Alignment Plan

Status: Closed
Owner: Designer persistence/runtime contract alignment
Last updated: 2026-08-06

## Purpose

Create a deliberate, low-risk alignment track between ProjectGlobalNodeDto (designer global sidecar shape) and RuntimeProjectDto (runtime export root shape) without collapsing required lane boundaries.

## Closeout Summary

This plan is complete and closed. The targeted designer/runtime alignment outcomes were delivered, validated, and relocked.

## Completed Outcomes

1. Global node alignment and canonicalization completed.
2. Alias cleanup completed for global-scope inherited fields (verbs, directionals, mappings, actions).
3. Procedure ownership naming unified to ProcedureIds across owning scopes.
4. Runtime inline procedures contract path removed; sidecar-first procedure catalog behavior is enforced.
5. Runtime procedure hydration moved to runtime-authored behavior state (RuntimeProcedures) and no longer depends on contract-surface Procedures.
6. Runtime export/load procedure index behavior uses project-root procedureIds plus GameRuntimeJson/Procedure/*.procedure.json.
7. ProjectGlobalNodeDto contract/authored partial structure remained compile-safe.
8. Shared-core schema work completed for global scope primitives:
   - GlobalScopeNode.core.schema.json introduced.
   - ProjectGlobalNodeDto_contract.schema.json introduced and composed.
   - RuntimeProjectDto_contract.schema.json moved under DtoContracts/Runtime and composed with shared core.
9. ProcedureIds placement finalized at base-schema level (ScopeNodeBase.core.schema.json) so designer/runtime scope bases inherit consistently.
10. Sample and snapshot artifacts were updated where required for deterministic post-lock output.

## Validation Closeout

1. dotnet build .\StoryboardDesigner.slnx passed.
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj passed (741/741).
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep" passed.

## Decisions Locked

1. Designer and runtime lanes remain intentionally separate where semantics diverge.
2. Procedure persistence remains sidecar-first; ownership remains id-reference-only.
3. No runtime-inline procedures payload is reintroduced in runtime disk contract.
4. Global primitives now rely on schema composition rather than ad-hoc duplication.

## Residual Follow-Up (Out of Scope For This Plan)

1. Continue one-class cadence execution under DESIGNER_DTO_ONE_CLASS_CADENCE_PLAN.md for remaining DTO migration slices.
2. Any future behavioral changes to procedure semantics must be handled as new compatibility-reviewed slices.

## Closeout Decision

Plan accepted as complete. Move to archived.
