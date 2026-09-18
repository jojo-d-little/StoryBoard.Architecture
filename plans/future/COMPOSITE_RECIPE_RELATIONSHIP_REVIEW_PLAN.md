# Composite Recipe Relationship Review Plan

Status: Future (not active)
Owner: TBD
Last updated: 2026-07-06

## Goal

Review whether composite modeling should remain one-target-to-one-recipe or evolve to one-target-to-many-recipes.

## Current State (Observed)

1. Actions may carry both composite target and recipe identifiers.
2. Runtime mapping has fallback behavior to reconcile mismatches.
3. Present modeling intent appears close to one target <-> one recipe in practice.

## Key Question

Should composite recipes remain a single internal definition per composite target, or become a first-class one-to-many relationship where a target can be built multiple ways?

## Decision Options

1. Keep one-to-one relationship.
2. Move to one-to-many recipes per composite target.
3. Hybrid: one canonical recipe plus optional alternates.

## Evaluation Criteria

1. Authoring UX clarity for recipe selection and editing.
2. Runtime command matching complexity and determinism.
3. Backward compatibility for existing saved projects.
4. Validation rule complexity and error quality.
5. Export/runtime contract stability.

## Discovery Checklist

1. Inventory all places where composite recipe IDs and target IDs are both stored.
2. Confirm where each ID is used for authoring, validation, mapping, and runtime execution.
3. Identify migration strategy if cardinality changes.
4. Propose canonical identity rules to prevent drift.
5. Define required regression tests before any model change.

## Candidate Deliverables (Future)

1. Architecture decision note documenting chosen cardinality model.
2. Data contract update proposal.
3. Migration plan for existing project files.
4. Focused test plan covering authoring, mapper, and runtime behavior.

## Out of Scope Today

1. No code/model changes.
2. No contract changes.
3. No migrations.
