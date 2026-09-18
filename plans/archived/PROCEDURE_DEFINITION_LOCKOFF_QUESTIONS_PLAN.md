# Procedure Definition Lock-Off Questions

Last updated: 2026-07-18
Status: Locked (Decision Log Complete)
Question count: 24

## Purpose

Capture the lock-off questions for introducing reusable, producer-friendly procedure definitions (state-transform interactions) distinct from composite recipes (identity-transform interactions).

## Lock-Off Questions

1. Is a Procedure Definition a new top-level project asset (global catalog), or scoped by planet/country/area/room?
2. Should procedures be versioned independently from project schema, or only via project schema version?
3. Can procedures be referenced by multiple actions across different scopes?
4. Should procedures be exportable in clean contract v1 as additive fields, or require a new clean schema version?
5. Do we allow procedure nesting (procedure calling procedure), or forbid initially?
6. What participant role types are required for v1: target, tool, catalyst, context fixture?
7. Do we support participant matching by object ID only in v1, or also by object type/tag?
8. Is participant quantity required in v1, or single-instance only?
9. Which satisfaction modes ship in v1: ExplicitMentionRequired, PossessionRequired, PresenceRequired, hybrids?
10. For ExplicitMentionRequired, does mention have to map to an exact resolved object token, or can synonyms satisfy it?
11. For PresenceRequired, what scope counts as present: current room only, or ancestor scopes too?
12. For PossessionRequired, does possession mean player container only, or any owned/equipped container tree?
13. What consumption policies are mandatory in v1: none, consume instance, decrement quantity, deactivate?
14. Should consumption be modeled as policy metadata or as explicit effect operations (or both)?
15. Do we permit partial success (some effects applied) or require transactional all-or-nothing execution?
16. What effect primitives are in v1: set variable, toggle bool, increment/decrement numeric, move/remove object?
17. Are effects restricted to participants only, or can they target arbitrary resolved scope nodes?
18. How are preconditions authored: simple variable predicates only, or script expression support in v1?
19. How should command parsing bind action text to procedure participants: positional, role qualifiers, or free mention resolution?
20. Must every required explicit participant be named in one command, or can prior context satisfy missing names?
21. What diagnostics contract is required: standardized failure codes plus producer-authored messages?
22. How should simulator/host diagnostics report procedure failures for debugging and replay determinism?
23. What minimal designer UX is acceptable for v1: form-based role/effect editor, no scripting required?
24. What migration behavior is required for existing projects and linked-action content: no-op compatibility, optional conversion tooling, or auto-upgrade?

## Notes

- This plan is intentionally question-first to lock architecture and contract boundaries before implementation.
- Related concept threads:
  - Per-participant consumption behavior.
  - Per-participant satisfaction mode (explicit mention vs possession vs presence).
  - Procedure reuse across multiple actions/verbs.

## Decision Log

1. Global project-level procedure catalog.
2. Procedure versioning follows project schema version.
3. Procedures are reusable across multiple actions/scopes.
4. Clean export support is additive to current contract.
5. No procedure nesting in v1.
6. Participant model uses a target plus generic participant requirements.
7. Participant matching supports more than object ids (type/tag-capable matching direction).
8. Quantity is supported in v1.
9. Satisfaction mode is configured per participant item (not global).
10. Explicit mention accepts synonyms/tokens when they resolve to the participant.
11. PresenceRequired means current room only.
12. PossessionRequired means player inventory/container tree only.
13. Consumption policy set in v1 includes None, ConsumeInstance, DecrementQuantity, DeactivateOnly.
14. Consumption is policy-driven per participant, plus scoped/simple mutation support in procedure: simple participant variable mutations only; procedure effects run only on success.
15. Execution is transactional all-or-nothing.
16. Allowed v1 mutation primitives: SetVariable, IncrementVariable, DecrementVariable on participants.
17. Mutations are restricted to procedure participants.
18. Preconditions are declarative/simple checks only (no script preconditions in v1).
19. Command preprocessing/scoping behavior remains unchanged; procedure binding must adapt to existing command resolution.
20. All required explicit participants must be named in the same command.
21. Diagnostics include standardized result codes plus producer-authored messages, and procedure-provided message helper values exposed through action variables.
22. Simulator/host diagnostics require detailed execution trace.
23. Designer UX in v1 is guided form editor only.
24. Procedure feature is net-new additive: no migration pipeline required for existing content.

## Procedure Message Helper Values

- Provide two author-defined values on each procedure, surfaced as action variables for outcome messages:
  - procedureSummary: short label/value.
  - procedureDescription: short narrative description.

## Phased Implementation Plan

Phase 1: Contracts and Data Model
- Add procedure catalog model at project level.
- Add participant requirement model with per-item satisfaction mode, quantity, and consumption policy.
- Add simple procedure mutation model (set/increment/decrement on participants).
- Add procedure summary/description fields and runtime action-variable exposure contract.

Phase 2: Persistence and Export
- Add procedure persistence to native project save/load.
- Add additive clean export/import support for procedures.
- Maintain backward compatibility for projects that do not define procedures.

Phase 3: Runtime Execution
- Add new invoke-procedure action execution path.
- Bind to existing command preprocessing/scoping outputs (no parser changes).
- Implement participant resolution checks, declarative preconditions, transactional application, and result codes.
- Emit detailed diagnostics for simulator/host.

Phase 4: Designer Authoring UX
- Add guided procedure editor UI.
- Add participant editor (matching selector, satisfaction mode, quantity, consumption).
- Add simple mutation editor and procedure summary/description editor.
- Add action linkage UI so multiple actions can reference one procedure.

Phase 5: Validation and Regression Coverage
- Add validation rules for malformed procedures and invalid action links.
- Add shared/runtime tests for success/failure paths, explicit mention rules, possession/presence checks, and transaction rollback.
- Add simulator-facing diagnostic assertions and representative authoring smoke tests.

## Phase Count

Total phases: 5
