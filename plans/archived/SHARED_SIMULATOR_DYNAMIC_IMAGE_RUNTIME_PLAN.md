# Shared Simulator Dynamic Image Runtime Plan

Status: Completed (archived 2026-07-13; S0.0/S0.1/S1/S2/S3/S4/S6/S7 complete; S5 remains deferred)
Owner: Storyboard.Shared runtime contracts + Storyboard.Simulator host integration
Last updated: 2026-07-13

## 0. Closeout Snapshot (2026-07-13)

This plan is closed and moved to `plans/archived`.

Consolidation outcome:

1. `plans/active/simulator-consolidation.md` moved to `plans/archived/simulator-consolidation.md`.
2. `plans/active/simulator-consolidation-post-implementation-review.md` moved to `plans/archived/simulator-consolidation-post-implementation-review.md`.

Closeout checklist outcome:

1. S3 complete: simulator host integration delivered with runtime payload rendering and regression coverage.
2. S4 complete: runtime regression + architecture boundary hardening completed and validated.
3. S6 complete: one-class-or-interface-per-file normalization across `Storyboard.Shared` treated as complete for this plan scope.
4. S7 complete: portability hardening signoff completed for Shared host/device neutrality expectations.
5. Final closeout complete: validation gates rerun and plan moved to `plans/archived`.

Explicit deferral handling:

1. S5 remains deferred cleanup and is not required to close this plan unless explicitly promoted back into required scope.

## 1. Purpose

Define the follow-on work needed to expose dynamic object image variants and chooser outcomes in shared runtime and simulator flows after designer-side authoring stabilizes.

Portability rule (codified):

1. `Storyboard.Shared` must remain host-agnostic and portable.
2. `Storyboard.Shared` must not take dependencies on UI frameworks, device APIs, platform rendering stacks, or host-specific display technologies.
3. All display/render/device constructs belong in host applications (for this plan, `Storyboard.Simulator`) and are consumed through host-facing contracts only.
4. Shared runtime contracts may describe presentation intent data, but must not encode platform-specific rendering implementation details.

## 2. Why This Exists

The current active plan intentionally limits phase 1 export/runtime expansion.
This future plan captures runtime and contract notes now so follow-on work does not lose context.

## 3. Inputs From Active Designer Plan

Source plan: plans/archived/DYNAMIC_IMAGE_VARIANTS_SHARED_RUNTIME_CHOOSER_PLAN.md

Locked inputs carried forward:

1. Object-only variant scope in initial delivery.
2. Variant shape is VariantName + FullImagePath + default semantics.
3. Chooser script is per-object text with limited if/else-if/else grammar.
4. Chooser can read object + room + area + country + planet + global properties.
5. Fallback order: explicit default, then single variant, then first in list.
6. Validation stays warning-oriented in designer phase.
7. Clean export/runtime integration is intentionally deferred to this follow-on.

## 4. Follow-On Scope Candidates

1. Add additive clean-export fields for object image variants and chooser scripts.
2. Add shared runtime contract DTOs for resolved object image variant selection.
3. Add runtime resolver service in Shared for chooser execution + fallback.
4. Add simulator host display plumbing for resolved image variant visuals.
5. Add deterministic diagnostics for chooser failures and fallback events.
7. Define a stable host-facing command response interface for room and object presentation state.
8. Consolidate host-engine boundary interfaces and payload contracts into a single RuntimeContracts host-facing location, with migration-safe compatibility shims.
9. Enforce one-to-one type/file organization across Storyboard.Shared (one class or interface per source file) before plan completion.
10. Add additive runtime contract fields for authored render resolution so simulator hosts can size scene surfaces from project data instead of host defaults.

## 5. Lock Questions For Follow-On

1. SRV-01: Clean export schema versioning and additive field naming.
2. SRV-02: Runtime evaluation cadence (on-demand vs cached + invalidation triggers).
3. SRV-03: Error/diagnostic channel for chooser evaluation failures.
4. SRV-04: Runtime contract for exposing resolved variant name to hosts.
5. SRV-05: Simulator display update triggers and performance guardrails.
6. SRV-06: Backfill strategy for projects that still use pre-variant content snapshots in tests.
7. SRV-07: Final room-change response shape details beyond baseline draft (additional fields and constraints).
8. SRV-08: Object lifecycle/render-delta contract semantics (spawn/update/despawn invariants and required fields).
9. SRV-09: Host payload partitioning between logical command outcome vs presentation outcome.
10. SRV-10: Forward-compatible extension points for non-image cues (audio/effects) without contract churn.
11. SRV-11: Runtime contract for authored project render resolution/viewport dimensions propagation to hosts.

## 5.1 Current Draft Baseline (Implemented)

The following host-facing draft contract shape is now implemented and should be treated as the starting baseline for S0 refinement:

1. `IGameCommandProcessingResult`
	1. Includes `RoomChange` (`IGameCommandRoomChangeData?`).
	2. Includes `RoomObjectChanges` (`IReadOnlyList<IGameCommandRoomObjectChange>`).
	3. Still temporarily includes legacy action-trace members slated for S0.0 removal.
2. `IGameCommandRoomChangeData`
	1. `OldRoom` (`IGameCommandOldRoomSummary?`) with canonical `RoomId` + `Name`.
	2. `NewRoom` (`IGameCommandNewRoomSummary?`) with canonical `RoomId`, `Name`, `Description`, `RoomDisplayMode`, `DirectionalRenderableImages`, and `RenderableRoomObjects`.
3. `IGameCommandRenderableImage`
	1. `ImagePath`, `X`, `Y`, `RotationDegrees`, `Scale`.
4. `IGameCommandRenderableRoomObject`
	1. `Name`.
	2. Composed `RenderableImage` (`IGameCommandRenderableImage`) instead of duplicated image geometry members.
5. `IGameCommandRoomObjectChange`
	1. `ChangeKind` (`Created|Updated|Removed`).
	2. Canonical `ObjectId` + `ObjectName`.
	3. Optional `RenderableRoomObject` payload for create/update render state.

Baseline policy carried into implementation:

1. Canonical room/object IDs are exposed directly to hosts for uniqueness.
2. No synthetic host IDs and no host-ID mapping layer.

## 5.2 Design Questions To Lock Before Implementation

These questions are intended for explicit review signoff. Any item not marked decided remains implementation-blocking for the related slice.

1. DQ-01 (Result envelope shape):
	1. Should `IGameCommandProcessingResult` remain a single combined envelope for logical + presentation outcomes, or should presentation data move into a nested payload object?
	2. Decision needed by: S0 final contract signoff.
2. DQ-02 (Match/success semantics):
	1. Do we keep `MatchedCommand` as-is, rename it for clarity, or replace it with a more explicit outcome enum?
	2. Decision needed by: S0.0 completion.
3. DQ-03 (Diagnostics contract):
	1. Should diagnostics remain free-form text only, or add stable diagnostic codes alongside text for host automation/log filtering?
	2. Decision needed by: before S1/S2 runtime plumbing.
4. DQ-04 (RoomChange nullability semantics):
	1. Is `RoomChange == null` the only valid “no room change” signal, and when non-null must `OldRoom`/`NewRoom` both be present?
	2. Decision needed by: S0 host contract examples.
5. DQ-05 (Room identity guarantees):
	1. Confirm canonical `RoomId` values are stable for the runtime session and sourced directly from scope IDs without transformation.
	2. Decision needed by: S0.1 boundary consolidation.
6. DQ-06 (Renderable image coordinate system):
	1. Define canonical meaning for `X`, `Y`, `RotationDegrees`, and `Scale` (origin, units, handedness, rotation direction, default scale behavior).
	2. Decision needed by: S2 implementation start.
7. DQ-07 (Renderable image layering):
	1. Do we need explicit z-order/render-order in `IGameCommandRenderableImage`, or is list order authoritative?
	2. Decision needed by: S2 implementation start.
8. DQ-08 (Asset path semantics):
	1. Should `ImagePath` be absolute path, project-relative path, or host-resolvable asset key; and must this be uniform across rooms and objects?
	2. Decision needed by: S1 export contract work.
9. DQ-09 (Room object change invariants):
	1. For `Created|Updated|Removed`, define required/forbidden combinations of `ObjectId`, `ObjectName`, and `RenderableRoomObject`.
	2. Decision needed by: S2 implementation start.
10. DQ-10 (Object identity stability):
	1. Confirm `ObjectId` stability guarantees across command sequence, room re-entry, and load/reload scenarios.
	2. Decision needed by: S2 runtime regression tests.
11. DQ-11 (Renderable room object shape):
	1. Is `Name + RenderableImage` sufficient for v1, or do we need additional host-facing fields now (visibility, interaction flags, type/category)?
	2. Decision needed by: S2 implementation start.
12. DQ-12 (Delta application model):
	1. Should hosts treat `RoomObjectChanges` as strict ordered deltas to apply sequentially, or an unordered set where final state only matters?
	2. Decision needed by: S3 host integration.
13. DQ-13 (Cross-payload consistency):
	1. When `RoomChange.NewRoom.RenderableRoomObjects` and `RoomObjectChanges` both exist, what precedence/merge rules apply in the same command result?
	2. Decision needed by: S3 host integration.
14. DQ-14 (Backward compatibility window):
	1. For legacy members (`ExecutedActionCount`, `ExecutedActionIds`, `ExecutedActions`), define exact deprecation/removal sequence and host migration expectations.
	2. Decision needed by: S0.0 completion.
15. DQ-15 (Boundary relocation acceptance criteria):
	1. What objective criteria mark S0.1 complete (namespace/folder targets, adapter state, host compile targets, zero behavior delta)?
	2. Decision needed by: S0.1 signoff.

## 5.3 Design Decision Matrix (Review Sheet)

Use this matrix during review to rapidly lock decisions. Set `Status` to `Locked` once agreed.

| ID | Topic | Options | Proposed Default | Status |
| --- | --- | --- | --- | --- |
| DQ-01 | Result envelope shape | A) Single combined result, B) Nested presentation payload | A) Single combined result for v1, revisit after host adoption | Locked (A) |
| DQ-02 | Match/success semantics | A) Keep `MatchedCommand`, B) Rename, C) Outcome enum | C) Outcome enum in future; keep current for S0.0 then migrate | Locked (A, revisit later) |
| DQ-03 | Diagnostics contract | A) Text only, B) Text + stable code | B) Add stable code field while preserving text | Locked (B) |
| DQ-04 | `RoomChange` nullability | A) `null` means no room change, B) Always non-null with flag | A) `null` means no room change; non-null requires both rooms | Locked (A) |
| DQ-05 | Room identity guarantees | A) Canonical scope IDs, B) Host-mapped IDs | A) Canonical scope IDs only | Locked (A) |
| DQ-06 | Coordinates/rotation/scale semantics | A) Documented canonical render-space contract, B) Host-defined interpretation | A) Canonical contract (origin/units/rotation/scale defaults) | Locked (A) |
| DQ-07 | Image layering | A) Explicit z-order field, B) Collection order | A) Explicit z-order for determinism | Locked (A) |
| DQ-08 | `ImagePath` semantics | A) Absolute path, B) Project-relative path, C) Asset key | C) Asset key (host-resolvable) for portability | Locked (B) |
| DQ-09 | Object change invariants | A) Loose nullability, B) Strict required/forbidden matrix per change kind | B) Strict matrix for Created/Updated/Removed | Locked (B) |
| DQ-10 | Object ID stability | A) Session-only stable, B) Stable across load/reload for same project | B) Stable across load/reload when source scope IDs are unchanged | Locked (B) |
| DQ-11 | Renderable room object shape | A) Name + image only, B) Add visibility/type/interaction hints now | A) Minimal v1, add fields additively as needed | Locked (A) |
| DQ-12 | Delta application model | A) Ordered sequential deltas, B) Unordered final-state hints | A) Ordered sequential deltas | Locked (A) |
| DQ-13 | `NewRoom` vs `RoomObjectChanges` merge | A) `NewRoom` authoritative on room change; apply object deltas after, B) Object deltas override always | A) `NewRoom` baseline then apply deltas in result order (rare dual-payload case supported) | Locked (A) |
| DQ-14 | Legacy action-trace member retirement | A) Remove immediately in S0.0, B) Deprecate then remove in next slice | B) Deprecate in S0.0, remove in next planned slice | Locked (B) |
| DQ-15 | S0.1 completion criteria | A) Compile-only, B) Compile + adapter + namespace + behavior parity checklist | B) Explicit checklist with behavior parity required | Locked (B) |

## 6. Suggested Slices

1. S0 (Prerequisite): Command Processor Host Response Contract Design.
	1. S0.0 (Phase Zero, first): Simplify IGameCommandProcessingResult for host boundary safety.
		1. Remove action-specific members from host-facing processing result: ExecutedActionCount, ExecutedActionIds, ExecutedActions.
		2. Preserve OutputLines and Diagnostics as the explainability surface for hosts.
		3. Keep any engine-internal action tracing behind diagnostics and/or non-host internal contracts.
		4. Ensure `RoomChange` and `RoomObjectChanges` remain the primary host-facing presentation update channels.
		5. Update manager mappings, host QA traces, and tests to compile and align with new boundary.
		6. Complete this step before adding any new room/image/object presentation payload properties.
	2. S0.1 (Boundary consolidation): Define a single host-facing service interface location in RuntimeContracts.
		1. Introduce a RuntimeContracts host-facing manager interface for command execution/loading.
		2. Keep existing GameManager interface available as compatibility shim during migration.
		3. Move/alias host-facing result payload contracts into the same RuntimeContracts boundary area.
		4. Plan staged host adoption in Simulator and Designer host layers without behavior change.
		5. Preserve canonical ID exposure semantics during boundary relocation.
	1. Define a versioned host-facing response envelope that keeps request/response flow intact.
	2. Define room presentation payload shape for room-change responses.
	3. Define object presentation payload shape for object lifecycle/render updates.
	4. Define clear separation between command-result text and host presentation data.
	5. Define extension slots for future audio/effects cues.
	6. Produce examples for no-room-change, room-change, and object-state-change responses.
	7. Capture explicit non-goal: no event-stream runtime model in this phase.
2. S1: Export contract extension and schema tests.
3. S2: Shared chooser runtime resolver implementation.
4. S3: Simulator host rendering integration and UX.
	1. Interim verification step (pre-render): at `GameDiagnosticsLevel.Medium`, command-processing diagnostics in host flows must echo room image and room-object image payload details received via `IGameCommandProcessingResult` (`RoomChange` + `RoomObjectChanges`).
	2. During this interim step, simulator rendering behavior remains unchanged; diagnostics are the verification channel.
5. S4: Runtime regression + architecture boundary hardening.
6. S5 (Deferred Cleanup): Remove legacy room command phrase concept.
	1. Remove designer model/persistence plumbing for room command phrases.
	2. Remove runtime command-phrase match branch and rely on verb/scoped-action model only.
	3. Remove now-unused DTO/mapper fields tied exclusively to room command phrases.
	4. Update tests and samples to eliminate commandPhrases coverage.
	5. Explicitly accept no backward-compat support for legacy project files containing room command phrases.
7. S6 (Code Hygiene, completion blocker): Normalize Storyboard.Shared type/file layout.
	1. Ensure one class or interface per source file across Storyboard.Shared.
	2. Split existing multi-type files with migration-safe moves and namespace continuity.
	3. Update references/usings with no behavioral changes.
	4. Complete this normalization before closing this plan.

Prerequisite gate:

1. S0.0 must be completed before S0 interface extension work starts.
2. S0.1 boundary consolidation design must be reviewed before broad host payload expansion.
3. S0 must be reviewed and accepted before S1-S4 implementation starts.
4. S5 is intentionally deferred and should be scheduled after current lighting/runtime deliverables are complete.

Plan completion gate:

1. Storyboard.Shared must satisfy one class/interface per source file before this plan can be marked complete.

## 7. Validation Gates

1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"
3. Focused simulator integration tests for runtime image switching once added.
4. Contract review signoff for S0 response examples before runtime implementation slices.
5. Pre-render verification check: with medium diagnostics enabled, command processing output must include echoed room/image/object payload details from the new interface contracts.
6. Portability guardrail check: Storyboard.Shared must compile without references to host UI/device namespaces or platform-specific rendering assemblies.

## 8. Working Notes

Progress update (2026-07-12):

1. S0.0 complete in code: host-facing processing results removed legacy action-trace members (`ExecutedActionCount`, `ExecutedActionIds`, `ExecutedActions`) across Shared contracts, manager mappings, and host QA trace consumers.
2. Validation gates passed after S0.0 changes:
	1. `dotnet build .\StoryboardDesigner.slnx`
	2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`
	3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`
3. S0.1 complete:
	1. Added RuntimeContracts host-facing manager/result/load interfaces under `Storyboard.Shared/RuntimeContracts/Host`.
	2. `GameManager`, `ProcessCommandResult`, and `LoadGameRuntimeProjectResult` now implement RuntimeContracts host shims while preserving GameManager compatibility interfaces.
	3. Designer and Simulator host composition/viewmodels now depend on RuntimeContracts host manager interface (`IHostRuntimeGameManager`) with no behavior changes.
	4. Added guardrail coverage asserting RuntimeContracts host shim assignability.
4. Relocation completion criteria snapshot:
	1. Namespace/folder target established in RuntimeContracts host location.
	2. Compatibility adapters preserved for staged migration safety.
	3. Host compile/tests pass with RuntimeContracts interface consumption.
	4. No remaining non-Shared legacy runtime-manager consumers outside compatibility guardrail assertions.
5. Next focus shifts to S2/S3 runtime execution and simulator host integration work.
6. S2 progress (shared chooser runtime resolver first slice):
	1. Added runtime object variant metadata carriage from clean bootstrap mapping into `RuntimeScopeNodeDescriptor` and `GameStateScopeNode` (`ImageVariants`, `ImageVariantChooserScript`).
	2. Preserved chooser metadata during runtime materialization cloning so spawned runtime objects retain variant selection inputs.
	3. Added shared chooser evaluation pass in `RuntimeCommandProcessorService` that resolves per-object selected variant path with deterministic fallback: default variant, then first available variant.
	4. Added chooser diagnostics for invalid scripts and unknown variant outputs, while preserving command success flow via fallback behavior.
	5. Updated render payload extraction to prefer resolved selected variant paths before base object image path values.
	6. Added focused regression coverage for chooser success, unknown output fallback, empty chooser fallback, and invalid script fallback diagnostics.
	7. Extended navigation host-payload regression coverage to assert chooser-selected object image paths flow through `RoomChange.NewRoom.RenderableRoomObjects` and medium diagnostics after room transitions.
	8. Added unknown-chooser-output transition regression coverage to assert deterministic fallback image selection in `RoomChange.NewRoom` payloads and diagnostics after navigation.
	9. Added invalid-chooser-script transition regression coverage to assert syntax-failure fallback image selection in `RoomChange.NewRoom` payloads and diagnostics after navigation.
	10. Added transition coverage for chooser conditional-branch evaluation (`IF` / `ELSE`) to assert selected-variant application in `RoomChange.NewRoom` payloads.
	11. S3 host progress (interim): simulator now renders payload-driven room/object images onto a dedicated preview surface constrained to 800x600 without changing overall simulator window layout.
	12. Added SRV-11 to track future contract work for authored project resolution/viewport propagation to hosts so simulator surface sizing can move from host default to authored runtime data.
	13. Added simulator-host regression coverage validating preview baseline construction from `RoomChange.NewRoom` and incremental object delta application from `RoomObjectChanges`.
	14. Added real-manager simulator integration coverage validating end-to-end preview image switching across successive commands.
	15. Added real-manager simulator integration coverage validating navigate-direction room transitions rebuild preview baseline and clear prior-room object overlays.
	16. Implemented SRV-11 additive runtime contract propagation for authored render dimensions (clean export -> runtime snapshot -> load result -> simulator render surface fallback).
	17. Completed room payload directional-image refactor: replaced `NewRoom.RenderableImages` with `NewRoom.DirectionalRenderableImages` and propagated room slot metadata from clean export to runtime host payloads.
	18. Updated simulator room-image rendering semantics to match designer preview ordering: rotate image first, then apply directional orientation/pinning and positional offsets.
	19. Added room display mode plumbing (`Independent` vs `Overlay`) from authored clean room export into runtime room descriptors and host `NewRoom` payloads for forward-compatible simulator mode handling.
	20. Added simulator S3 mode-behavior coverage and implementation: overlay mode renders all directional room images, independent mode currently renders a single preferred directional image (default slot when available).
	11. S2 implementation snapshot: runtime chooser resolution and fallback behavior are implemented in Shared for object variants, with focused regression coverage across direct room-object changes and navigation room-change payloads.

S2 closeout snapshot (2026-07-12):

1. Completed in S2 implementation scope:
	1. Shared resolver execution and deterministic fallback ordering.
	2. Diagnostics for unknown chooser output and invalid chooser syntax.
	3. Host payload propagation coverage through both `RoomObjectChanges` and `RoomChange.NewRoom.RenderableRoomObjects`.
2. Deferred from S2 to S3 (host integration scope):
	1. Focused simulator runtime image-switching integration tests and UX wiring.
3. Lock decisions resolved for S2 closure:
	1. SRV-02 (evaluation cadence/invalidation model) locked.
	2. SRV-03 (diagnostic channel shape/coding strategy) locked.
	3. SRV-04 (resolved variant-name host exposure scope) locked.

Locked decisions for S2 closeout (approved 2026-07-12):

1. SRV-02 (runtime evaluation cadence):
	1. Lock to post-action evaluation for payload construction: chooser execution must occur after command actions finish and before result payload emission, against the current game state at that exact point.
	2. No cross-command chooser result cache: chooser outputs must not be reused across command runs because properties/variables may have changed.
	3. Add command-cycle incremental reevaluation semantics: only objects/rooms marked changed during the current command run are reevaluated; unchanged entities may reuse already-known selected image state.
	4. Dirty tracking options:
		1. Preferred approach: `lastChangedAt` (or equivalent monotonic change marker) and command-start watermark to compute "changed since command start".
		2. Alternative approach: per-command dirty set built during action execution.
		3. `lastChangedAt` advantage: avoids full object/room cleanup passes to mark entities clean at command end; only update a running change marker when an entity property/variable value changes.
	5. Rationale: keeps correctness deterministic while reducing unnecessary chooser executions within a command cycle.
	6. Follow-up: if performance or complexity warrants, standardize the chosen change-tracking strategy across object and room mutation paths in S4.
2. SRV-03 (chooser diagnostics channel):
	1. Lock to text diagnostics in existing diagnostics list for S2/S3, with stable machine-readable prefix segments for filtering.
	2. Prefix format decision: `SUBSYSTEM.EVENT_CODE:` followed by human-readable details.
	3. Chooser subsystem reserved codes for now: `CHOOSER.UNKNOWN_VARIANT` and `CHOOSER.INVALID_SCRIPT`.
	4. Message-shape guidance (forward compatibility): emit optional `key=value` tokens after the prefix (for example `object=Lantern`, `fallback=day`) so future filters can target fields without contract changes.
	5. Control model compatibility decision: keep existing low/medium/high diagnostic level behavior as-is; when subsystem toggles are introduced later, they must compose with level gates (level gate first, subsystem gate second).
	6. Follow-up: add optional structured diagnostic codes/fields and subsystem-level enablement controls as additive contract evolution in S4 if host automation requires stronger control.
3. SRV-04 (resolved variant name contract exposure):
	1. Lock to no new host-facing contract field in S2/S3; expose resolved image path only.
	2. Decision rationale: simulator rendering requires resolved path, while explicit variant-name exposure is not yet required for host behavior and would expand contract surface prematurely.
	3. Forward-compatible path: if hosts later need variant identity, add an optional additive field (for example `resolvedVariantName`) in a future contract revision without changing existing required members.
	4. Compatibility rule for future additive field: absent/empty means "host should treat variant identity as unspecified" and continue rendering by resolved image path.

Late-phase follow-up (added 2026-07-12):

1. S7 (Portability hardening): enforce Shared runtime host/device neutrality before plan closure.
	1. Add/extend architecture guardrail tests that fail when `Storyboard.Shared` introduces UI/device/platform-specific dependencies.
	2. Add a lightweight dependency audit step to validation gates (namespace/assembly-level) for `Storyboard.Shared`.
	3. Verify simulator-only ownership of display constructs and adapters for room/object rendering payloads.
	4. Document approved boundary patterns for adding new host-facing presentation fields without leaking platform concerns into Shared.
	5. Make S7 completion a required signoff item prior to marking this plan complete.

Deferral decision (2026-07-12):

1. Automated portability guardrail test implementation is approved but intentionally deferred to S7.
2. No additional portability-test automation work is planned in the current S2/S3 execution window.
6. S1 progress (additive clean-export contract extension):
	1. Added optional room image transform fields to clean room contract payloads: `overlayOffsetX`, `overlayOffsetY`, and `overlayRotationDegrees`.
	2. Wired clean export mapping to emit room image transforms from authored room image entries.
	3. Extended clean-export regression coverage to assert transform fields emit when configured, with no schema-version change (`1.0`) and no snapshot churn for default-zero values.
	4. Added Birmingham clean-export snapshot baseline coverage (project + navigation artifacts) to broaden representative contract regression protection beyond single-room fixtures.
	5. Added additive clean-export object chooser-script field (`imageVariantChooserScript`) and regression coverage asserting authored chooser scripts are emitted for object appearance contracts.
	6. Added explicit clean-export image portability marker (`imagePathSemantics: cleanExportRelative`) for object variants and room images, with schema/regression assertions to lock host-readable path semantics.
	7. Added clean-export bootstrap reader tests to verify `imagePathSemantics` deserializes for both room images and object image variants, preserving runtime read compatibility.
	8. Extended assets manifest image entries with the same portability marker (`imagePathSemantics: cleanExportRelative`) and regression assertions to keep path semantics consistent across all clean-export image carriers.
	9. Added a cross-carrier contract guardrail test asserting emitted image paths are relative/non-rooted across clean project payload, clean room payload, and assets manifest entries.
	10. Added unresolved-image fallback guardrail coverage asserting payload image paths export as empty strings while unresolved source paths are retained in assets manifest `unresolvedSources`.
	11. Added schema-shape validation asserting clean assets manifest always exists and exposes `unresolvedSources` as an array for structural contract stability.
	12. Added a consolidated semantics consistency guardrail asserting `imagePathSemantics` is emitted and equal across project object variants, room object variants, room images, and manifest image entries.
	13. S1 completion note: export contract extension and schema/regression coverage are complete for this plan stage; no schema-version bump required and `1.0` remains current.

1. Keep Shared independent of designer-only view models and editor state.
2. Keep room designer transient preview selection out of runtime/export contract.
3. Preserve deterministic behavior when chooser output is unknown.
4. Treat export contract updates as additive and explicitly version-governed.
5. Host-facing room/object identity must use canonical existing scope IDs generated by designer and carried through Shared runtime.
6. Do not synthesize new IDs for host contracts and do not introduce host-ID mapping tables/adapters.
