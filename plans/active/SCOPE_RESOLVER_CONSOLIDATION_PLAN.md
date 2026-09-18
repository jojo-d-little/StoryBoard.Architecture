# Scope Resolver Consolidation Plan

Date: 2026-08-16
Status: Deferred / Planning
Owner: Runtime follow-up

## Goal

Consolidate runtime scope/object/variable resolution under one reusable grammar and resolver strategy to eliminate drift, reduce duplicate parsing logic, and support anchor-rooted references consistently.

## Why This Plan Exists

Recent event/filter anchor syntax work (`anchor::subProperty.variable`) exposed overlapping resolver implementations and split grammar ownership across read and write paths. This plan captures the consolidation approach while context is fresh.

## Locked Direction (Go-Forward)

1. Use `RuntimeReferenceValueResolverPipeline` as the primary composition backbone for reference resolution.
2. Use one shared parser/normalizer as grammar authority for reference tokens.
3. Keep read and mutation interfaces separate, but they must consume the same grammar engine.
4. Keep action-specific resolvers as adapters, not grammar owners.

## Current Relevant Components

Primary keep/extend targets:

1. `Storyboard.GameEngine/GameServices/References/RuntimeReferenceValueResolverPipeline.cs`
2. `Storyboard.GameEngine/GameServices/References/IRuntimeReferenceValueResolver.cs`
3. `Storyboard.GameEngine/GameServices/References/RuntimeActionReferenceValueResolverPipeline.cs`
4. `Storyboard.GameEngine/GameServices/References/ScopeChainReferenceValueResolver.cs`
5. `Storyboard.GameEngine/GameServices/References/ObjectHierarchyReferenceValueResolver.cs`

Mutation-side component to integrate with shared grammar:

1. `Storyboard.GameEngine/GameServices/References/ScopeChainVariableMutationResolver.cs`

## Consolidation Scope

1. Introduce shared runtime token grammar parser/normalizer.
2. Support explicit anchor-rooted form: `anchor::subProperty.variable`.
3. Support compatibility mode for legacy dotted forms where needed.
4. Add deterministic diagnostics for malformed or ambiguous tokens.
5. Remove duplicate parser logic across read and mutation code paths once parity is proven.

## Non-Goals (This Plan)

1. Full runtime event dispatch implementation.
2. Host delivery protocol redesign.
3. Immediate contract removals without migration gates.

## Phased Plan

### Phase 0 - Inventory and Baseline

1. Catalog all token parsing/normalization call sites in runtime read and mutation flows.
2. Capture baseline behavior with golden tests for representative legacy inputs.
3. Record ambiguities and divergence points explicitly.

Done criteria:

1. Complete inventory doc/checklist exists.
2. Golden tests reproduce current behavior for known legacy patterns.

### Phase 1 - Shared Grammar Engine

1. Add a shared parser/normalizer in runtime references area.
2. Represent parse result as explicit token model (kind, alias/root, path segments, diagnostics).
3. Include support for:
- payload-style flat token
- anchor-rooted token
- scoped alias token (`self`, `room`, `area`, `country`, `planet`, `global`, `player`)

Done criteria:

1. Parser unit tests cover valid/invalid/malformed/ambiguous inputs.
2. No behavior change yet in existing runtime consumers.

### Phase 2 - Read Resolver Integration

1. Wire `RuntimeReferenceValueResolverPipeline` resolvers to consume shared parser output.
2. Keep token output behavior equivalent for legacy scenarios.
3. Add optional diagnostics capture for ambiguous legacy forms.

Done criteria:

1. Golden read-resolution tests pass old-vs-new parity.
2. Anchor-rooted references resolve deterministically where supported.

### Phase 3 - Mutation Resolver Integration

1. Refactor `ScopeChainVariableMutationResolver` to delegate grammar parsing to shared parser.
2. Preserve mutation semantics while removing duplicate token normalization logic.
3. Add parity tests for SetGameProperty target resolution.

Done criteria:

1. Mutation parity tests pass.
2. Duplicate parsing logic removed from mutation resolver.

### Phase 4 - Action Resolver Alignment

1. Update action-specific resolver adapters to consume shared parse model where applicable.
2. Remove ad hoc token interpretation duplication in action flows.
3. Ensure action context references remain deterministic and backward compatible.

Done criteria:

1. Action integration tests pass with unchanged behavioral outputs.
2. No remaining duplicate grammar implementations.

### Phase 5 - Retirement and Cleanup

1. Retire legacy duplicate parsing logic after compatibility window and parity sign-off.
2. Keep compatibility mode toggles (if needed) with explicit diagnostics.
3. Update docs and deprecation notes for legacy dotted anchor-like forms.

Done criteria:

1. Deprecated parser paths removed.
2. Diagnostics/deprecation policy documented.

## Retirement Candidates (Future)

Primary candidate:

1. Bespoke normalization/splitting logic currently inside `ScopeChainVariableMutationResolver`.

Likely additional candidates:

1. Ad hoc alias assembly/parsing patterns duplicated in individual action execution helpers.
2. Implicit interpretation paths for legacy dotted anchor-like tokens when explicit anchor-root form is available.

Not retirement targets:

1. Resolver pipelines themselves (`RuntimeReferenceValueResolverPipeline`, `RuntimeActionReferenceValueResolverPipeline`).
2. Resolver interfaces used as extension seams.

## Validation and Migration Gates

1. Golden parity tests for legacy token behavior before and after each migration phase.
2. Parser conformance tests for anchor-root syntax and malformed input diagnostics.
3. Runtime integration tests for read + mutation + action flows.
4. Feature-flag/diagnostic window for legacy dotted anchor-like references.
5. Explicit sign-off before deleting old parser paths.

## Risks and Mitigations

1. Risk: Behavior drift across read and mutation semantics.
- Mitigation: shared parser model + parity test suite before cleanup.

2. Risk: Breaking legacy authored content.
- Mitigation: compatibility mode + warning diagnostics + staged retirement.

3. Risk: Cross-cutting refactor touches many runtime call sites.
- Mitigation: phased migration with stable rollback points and narrow PR slices.

## Suggested Execution Order When This Starts

1. Phase 0 inventory + baseline tests.
2. Phase 1 parser implementation + unit tests.
3. Phase 2 read resolver pipeline migration.
4. Phase 3 mutation resolver migration.
5. Phase 4 action adapter alignment.
6. Phase 5 retire duplicate parsers.

## Exit Criteria

1. One shared grammar parser/normalizer is authoritative for runtime token interpretation.
2. Read/mutation/action resolver paths consume shared grammar output.
3. Duplicate parser implementations are retired.
4. Legacy compatibility policy is explicit and tested.
5. Anchor-rooted syntax support is deterministic and contract-aligned.
