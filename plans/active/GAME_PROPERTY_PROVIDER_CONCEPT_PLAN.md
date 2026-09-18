# Game Property Provider Concept Plan

Date: 2026-08-16
Status: Deferred

## Goal

Introduce a generalized Game Property Provider concept so scope nodes and non-scope sources (for example traversal legs) can be treated consistently in chooser/search UX.

## Why

1. Current chooser/search flows are mostly scope-node-centric.
2. Non-scope sources become awkward special cases.
3. A unified provider model improves reuse across dialogs.

## Proposed Provider Shape (Draft)

1. Provider identity
- providerId
- providerType (ScopeNode, TraversalLeg, etc.)
- displayName
- path

2. Provider semantics
- scopeHint (optional)
- relations (self, parent, ancestor, child, descendant, sibling, traversal)

3. Provider properties
- propertyName
- valueRestriction
- source flags (authored/runtime)

## Incremental Adoption Path

1. Add neutral provider DTO/viewmodel in Designer (no runtime contract change).
2. Add adapter from existing GamePropertyChoiceItem to provider model.
3. Pilot provider-based filtering in scope/object chooser dialogs.
4. Add traversal-leg providers intentionally after UX rules are agreed.

## Open Questions

1. Should traversal legs be selectable by default or only in advanced search?
2. Should providerType have UI-group defaults per dialog entry point?
3. How should expectedScopeType map to providerType constraints?

## Not In Scope (Now)

1. Runtime contract/schema changes.
2. Event payload runtime resolution changes.
3. Full replacement of existing chooser models in one pass.
