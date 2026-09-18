# Plans Archive Index

Last updated: 2026-09-11

This folder contains completed or historical planning artifacts moved out of the repository root to keep active guidance easy to scan.

## Planning Lifecycle Folders

Planning artifacts are expected to move through folders in this order:

1. `plans/future/` -> early thinking and not-yet-activated plans.
2. `plans/active/` -> currently active implementation plans.
3. `plans/archived/` -> completed or retired plans kept for history/audit.

For staged handoff workstreams, archive the plan set as one unit:

1. Main plan file.
2. All stage handoff files tied to that plan.

Do not archive staged plans piecemeal.

## Default Authoring Pattern (New Work)

New non-trivial plans should follow the staged handoff standard:

1. Standard: `plans/PLAN_AUTHORING_STANDARD.md`
2. Main plan template: `plans/templates/STAGED_HANDOFF_PLAN_TEMPLATE.md`
3. Stage handoff template: `plans/templates/STAGE_HANDOFF_TEMPLATE.md`

Mandatory kickoff cadence for staged plans:

1. Lock Stage 1..N order in the main plan.
2. Create `plans/active/handovers/<PLAN_FILE_STEM>/`, then create all Stage 1..N handoff docs immediately as placeholders in that plan-owned folder.
3. Include an opening prompt in each stage handoff placeholder.
4. Begin implementation only after the full placeholder handoff set exists.

Use `plans/archived/PORTRAIT_ROOM_ORIENTATION_HANDOFF_PLAN.md` and its stage handoff set under `plans/archived/handovers/` as the reference example.

## Archived Files

1. `ENHANCEMENT_MISSES.md`
- Status: Archived after triage.
- Notes: Captured one-time backlog reconciliation and closure notes; items were either completed, retired, or promoted.

2. `GAME_STATE_DATA_EXTRACTION_PLAN.md`
- Status: Completed.
- Notes: Runtime game-state extraction into Shared completed with regression validation.

3. `PHASE_1_2_3_EXECUTION_PLAN.md`
- Status: Completed execution history.
- Notes: Phase 1-3 deliverables complete; retained for audit trail.

4. `PHASE_1_2_3_REVIEW.md`
- Status: Review packet snapshot.
- Notes: Captures review-state evidence and open follow-up considerations at time of review.

5. `RUNTIME_HOST_FINAL_PLAN.md`
- Status: Completed.
- Notes: Runtime-host decoupling and shared manager direction finalized and implemented.

6. `MVVM_ARCHITECTURE_PLAN.md`
- Status: Retired (deleted 2026-06-29).
- Notes: Superseded by strict Designer MVVM guidance in `.github/instructions/storyboard-designer-mvvm.instructions.md`.

7. `CLEAN_EXPORT_V1.md`
- Status: Archived (2026-06-29).
- Notes: Export contract essentials promoted into `ENHANCEMENT_GUIDELINES.md`; file retained as historical reference.

8. `EXPORT_STRATEGY_PLAN.md`
- Status: Archived (2026-06-29).
- Notes: Planning content was largely completed/redundant; durable policy moved to living docs.

9. `simulator-consolidation.md`
- Status: Archived (2026-07-13).
- Notes: External-simulator migration plan completed; implementation decisions locked and delivered.

10. `simulator-consolidation-post-implementation-review.md`
- Status: Archived (2026-07-13).
- Notes: Mandatory post-implementation audit completed; ProjectModelRuntimeSnapshotMapper retained for current in-use runtime paths.

11. `SHARED_SIMULATOR_DYNAMIC_IMAGE_RUNTIME_PLAN.md`
- Status: Archived (2026-07-13).
- Notes: Dynamic image runtime follow-on plan completed and moved from active; S5 remains explicitly deferred cleanup outside closure-critical scope.

12. `ECHO_RESULTCODE_SINGLE_PATH_CUTOVER_PLAN.md`
- Status: Archived (2026-07-19).
- Notes: Completed and archive-ready; no remaining implementation tasks.

13. `GAME_OBJECT_SCOPE_COLLECTION_AND_DISCOVERY_UNIFICATION_PLAN.md`
- Status: Archived (2026-07-19).
- Notes: Closed with phase status updates completed across the plan.

14. `NAVIGATE_TO_ADJACENT_PLAN.md`
- Status: Archived (2026-07-19).
- Notes: Implemented and validated; no pending phase work.

15. `SHARED_VARIABLE_CONTAINMENT_AND_SET_PROPERTY_RESOLUTION_PLAN.md`
- Status: Archived (2026-07-19).
- Notes: Implemented and archive-ready after validation signoff.

16. `SHARED_VARIABLE_REFERENCE_ALIASING_PLAN.md`
- Status: Archived (2026-07-19).
- Notes: Implemented with lock decisions resolved and signoff complete.

17. `PROCEDURE_DEFINITION_LOCKOFF_QUESTIONS_PLAN.md`
- Status: Archived (2026-07-19).
- Notes: Question lock-off complete; retained for decision history.

18. `HIDE_EMPTY_CHILDREN_VISIBILITY_PLAN.md`
- Status: Archived (2026-07-19).
- Notes: Implemented and closed; final lock decision on validation visibility behavior recorded.

19. `SELF_ALIAS_SCOPE_UNIFICATION_PLAN.md`
- Status: Archived (2026-07-19).
- Notes: Implemented for current scope; room-owned chooser follow-on is explicitly deferred to future room multi-image support.

20. `TOP_LEFT_ANCHOR_PLACEMENT_SIMPLIFICATION_PLAN.md`
- Status: Completed and archived (2026-07-24).
- Notes: Completed through Phase 6 cleanup with SharedOnly single-model workflow, parity evidence captured, and required gates passing.

21. `TOP_LEFT_ANCHOR_PHASE0_BASELINE_PACKET.md`
- Status: Archived (2026-07-24).
- Notes: Phase 0 baseline tuple/visual packet retained as historical calibration evidence after full plan completion.

22. `OBJECT_EDIT_REQUEST_ADAPTER_SHIM_PLAN.md`
- Status: Archived (2026-07-28).
- Notes: Consolidated into `plans/active/OBJECT_EDIT_DIALOG_CONSISTENCY_PLAN.md` so object-edit consistency/adaptor-shim work has one canonical active source.

23. `OBJECT_MOVEMENT_PLAN.md`
- Status: Archived (2026-08-01).
- Notes: Final closeout completed with lean Phase 5 hardening, export-shape fixes, and full app-test regression pass; transition-hint chooser and waypoint/host-buffered intent remain explicit follow-on planning.

24. `OBJECT_MOVEMENT_RESTRICTIONS_PLAN.md`
- Status: Archived (2026-07-28).
- Notes: Lock-off complete and treated as closed for current transition window; any future restriction extensions should open a new focused plan.

25. `MULTI_PART_COMMAND_LEG_TRAVEL_PLAN.md`
- Status: Archived (2026-07-28).
- Notes: Complete and already archive-ready; multi-leg command travel implementation and regressions are complete.

26. `BASE_OBJECT_OWNERSHIP_LOCKOFF.md`
- Status: Archived (2026-07-28).
- Notes: Tentative closure window passed with no new lock-area issues reported; archived per team decision.

27. `RUNTIME_CONTRACT_MANAGEMENT_PLAN.md`
- Status: Archived (2026-08-01).
- Notes: First-pass runtime contract management completed with contracts assembly boundary stabilization, runtime naming migration completion, schema/codegen coverage expansion, and root-only schema-version policy lock.

28. `RUNTIME_CONTRACT_DTO_MIGRATION_MATRIX.md`
- Status: Archived (2026-08-01).
- Notes: Migration matrix closed after first-pass rename/split rollout completion and policy lock alignment; retained for historical mapping traceability and maintenance reference.

29. `RUNTIME_EXPORT_FILE_PER_SCOPE_PLAN.md`
- Status: Archived (2026-08-02).
- Notes: File-per-scope runtime export rollout completed for current scope; index coverage and snapshot baselines were refreshed, and full test suite verified green at closeout.

30. `PROJECT_PLAYER_SCOPE_NODE_COLLAPSE_PLAN.md`
- Status: Archived (2026-08-02).
- Notes: Collapse complete; `ProjectPlayerScopeNode` removed from product source, player-term canonical path emissions removed, and focused/full validation gates passed at closeout.

31. `GLOBAL_OBJECTS_SCOPE_MODEL_CONSOLIDATION_PLAN.md`
- Status: Archived (2026-08-02).
- Notes: Consolidation phases A-E completed with `ProjectModel` as sole global-object data owner and active `GlobalObjectsScope` ownership removed. Phase F (`ProjectModel` as root scope node parity work) was completed via standalone follow-on plan `plans/archived/PROJECTMODEL_ROOT_SCOPE_PARITY_PLAN.md`.

32. `PROJECTMODEL_ROOT_SCOPE_PARITY_PLAN.md`
- Status: Archived (2026-08-03).
- Notes: Completed through F0-F5 with `ProjectModel` as sole active global root and `ProjectGlobalScopeNode` removed. Focused gates and full solution test suite passed at closeout; no authored project JSON format change occurred, so sample migration was not required.

33. `DESIGNER_PROJECT_GLOBAL_ALIGNMENT.md`
- Status: Closed and archived (2026-08-06).
- Notes: Completed through Step 10 with strict global-node-required load, root DTO cleanup, one-time sample migration executed/validated, and full solution tests green. Schema/contract follow-on was explicitly deferred to a separate future plan.

34. `ROOMLINK_COURSE_CORRECTION_PLAN.md`
- Status: Closed and archived (2026-08-07).
- Notes: Designer/runtime ownership split completed and locked: designer remains canonical on traversal connections, runtime remains canonical on room links with passability via link gameProperties; runtime traversal-connection compatibility artifacts were retired and validation gates passed.

35. `RUNTIME_NAVIGATION_AREA_FIRST_FILE_STRUCTURE_PLAN.md`
- Status: Closed and archived (2026-08-08).
- Notes: Area-first runtime navigation structure is complete and now represented through per-scope runtime payloads; legacy aggregate navigation artifact handling is compatibility cleanup only, and locked traversal-passability linkage requirements were delivered with validated regression coverage.

36. `COMMAND_ACTION_SCAR_TISSUE_RETIREMENT_PLAN.md`
- Status: Closed and archived (2026-08-08).
- Notes: Legacy room-command phrase/link contract surface was retired in favor of action-first execution only; schema/DTO cleanup, lock-manifest reconciliation, runtime export snapshot refresh, and full regression gates completed green.

37. `COMMAND_ACTION_CONTRACT_PAYLOADS_PLAN.md`
- Status: Closed and archived (2026-08-09).
- Notes: Hard-switch action payload contract refactor closed; payload model, discriminator policy acceptance (`$payloadType`), and missing payload diagnostic behavior were finalized and validated.

38. `SIMULATOR_CURRENTSESSION_ISOLATION_PLAN.md`
- Status: Closed and archived (2026-08-25).
- Notes: CurrentSession usage was reduced to debugger-only runtime-state graph projection, thin-safe fallback debt was removed from non-debugger paths, and host-owned capability querying (`SupportsCapability`) now drives runtime-state/debugger feature gating with full regression suite passing at closeout.

39. `DIRECTION_MAPPING_BOUNDARY_PLAN.md`
- Status: Closed and archived (2026-09-07).
- Notes: Direction mapping ownership boundary was finalized with producer-owned traversal mappings exported/bootstrapped end-to-end, command parsing hard-cut to mapping-driven direction resolution, designer Direction10 Up/Down authoring support, and removal of bootstrap legacy alias normalization.

40. `PORTRAIT_ROOM_ORIENTATION_HANDOFF_PLAN.md` (+ handoff unit)
- Status: Closed and archived as a staged handoff unit (2026-09-11).
- Notes: Archived together with Stage 1-5 handoffs under `plans/archived/handovers/` to preserve locked implementation order, stage context, validation evidence, and end-to-end closure traceability.

41. `ROOM_DISPLAY_NAME_AND_TRANSITION_PRESENTATION_PLAN.md` (+ handoff unit)
- Status: Closed and archived as a staged handoff unit (2026-09-11).
- Notes: Archived together with Stage 1-7 handoffs under `plans/archived/handovers/`; Stage 6 was an explicit no-op retirement stage (no approved contract retirement candidates), and Stage 7 closeout gates passed (`dotnet build`, full `StoryboardDesigner.App.Tests`, `Storyboard.TransportCodegen.Tests`, and playback regression filter).

## Root-Level Active Planning Docs

The following remain at root because they still appear active or need explicit retirement/rewrite decisions:

1. `DESIGNER_VS_GAME_SEPARATION_PLAN.md`
2. `SIMULATOR_TEST_AUTOMATION_ENHANCEMENT_PLAN.md`
