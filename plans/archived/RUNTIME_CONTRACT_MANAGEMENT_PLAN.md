# Runtime Contract Management Plan

Status: Closed 2026-08-01 - First-pass contract management complete; follow-on work moved to maintenance slices
Owner: Storyboard.Shared contract boundary + host mapping seams
Last updated: 2026-08-01 (implementation alignment refresh)

## Closeout Snapshot (2026-08-01)

Closed with the following first-pass outcomes:

1. Contracts assembly and dependency direction are in place (`Shared` depends on `Contracts`; hosts consume both as needed).
2. Runtime DTO naming migration to `Runtime*` is complete for the targeted family.
3. Broad schema/codegen coverage and generated-partial seam pattern are established in `Storyboard.Shared.Contracts`.
4. Root-only `SchemaVersion` authority is locked as policy.
5. Enum convergence achieved substantial progress and is now maintenance-mode only (new slices only for clear duplicate-overlap cases).

Follow-on maintenance guidance:

1. If desired, remove nested schemaVersion writes while preserving read tolerance for legacy artifacts.
2. Open enum convergence work only when a concrete duplicate enum concept is identified in non-boundary model/request paths.

## 1. Purpose

Establish a formal, repeatable way to manage shared runtime contracts so schema intent is clear, change risk is controlled, and host boundaries remain clean.

Enum-first objective:
1. Where a runtime contract field is an enum, host/runtime domain models should also use that enum type as the canonical in-memory representation.
2. String conversion is allowed only at explicit boundaries (UI input widgets, legacy file read tolerance, and JSON wire serialization concerns).
3. Avoid helper-driven normalize-to-string patterns in core model logic once enum contract types are available.

This plan starts with one pilot object and one full lifecycle before broad rollout.

This plan now also formalizes project separation by introducing a contracts-only assembly.

## 2. Key Decisions Captured

1. Designer authoring model and designer persistence DTO are not the same responsibility.
2. In Designer, Planet and PlanetDto may overlap in fields but should remain separate types.
3. The shared runtime-facing contract for this pilot is currently CleanPlanetDto.
4. The "Clean" prefix is unclear and should be retired in favor of explicit Runtime naming.
5. Contracts and implementations should be split into separate assemblies.
6. New target assembly: Storyboard.Shared.Contracts for DTO contracts and shared interfaces only.
7. Runtime/service implementations remain in Storyboard.Shared.
8. Contract enum types are the source-of-truth types for model state; string forms are boundary-only.

## 3. Target Project Boundaries

Contracts project:
1. Storyboard.Shared.Contracts contains DTO contracts and cross-host interfaces only.
2. No host UI dependencies.
3. No runtime service implementations.

Runtime implementation project:
1. Storyboard.Shared keeps runtime engines, managers, mappers, and other implementations.
2. Storyboard.Shared references Storyboard.Shared.Contracts.

Host projects:
1. StoryboardDesigner.App and Storyboard.Simulator can reference contracts directly when needed.
2. Hosts continue to reference Storyboard.Shared for runtime behavior.

## 4. Pilot Contract (Historical)

Pilot type:
1. Storyboard.Shared RuntimeContracts Dtos CleanPlanetDto (pre-cutover)

Target name:
1. RuntimePlanetDto (implemented)

Naming rule for pilot and follow-ons:
1. Use RuntimeXxxDto naming.
2. Avoid repeating "Contract" in each type name because contract intent is already encoded by namespace and folder.

## 5. Scope

In scope for pilot:
1. Create Storyboard.Shared.Contracts project and add it to the solution.
2. Move the pilot runtime contract DTO into Storyboard.Shared.Contracts.
3. Rename shared runtime contract type from CleanPlanetDto to RuntimePlanetDto.
4. Update compile-time usages in Shared, Designer export mapping, simulator paths, and tests.
5. Preserve serialized payload compatibility where practical.
6. Add or update focused tests that prove no behavior regression for export to bootstrap lifecycle.
7. Document the reusable migration checklist.
8. Add enum-convergence acceptance checks for migrated fields so model and request surfaces align with contract enum types.

Out of scope for pilot:
1. Full rename of every Clean* contract in one change.
2. Refactoring Designer Planet to wrap PlanetDto.
3. Broad schema redesign beyond pilot naming and guardrails.
4. Moving runtime implementations out of Storyboard.Shared.

## 6. Full-Cycle Pilot Workflow

Phase 0: Project Split Foundation
1. Create Storyboard.Shared.Contracts project and wire into StoryboardDesigner.slnx.
2. Add initial project references from Storyboard.Shared and consuming hosts/tests that need contract types.
3. Keep namespaces stable where practical for low-risk migration.

Phase 1: Contract Identification and Boundary Lock
1. Confirm source of truth contract lives in Storyboard.Shared.Contracts.
2. Confirm host-specific authoring models remain host-local.
3. Record lock decision: Planet model remains separate from runtime contract DTO.

Phase 2: Pilot Move and Naming Cutover
1. Move pilot DTO file into Storyboard.Shared.Contracts.
1. Rename type and file CleanPlanetDto -> RuntimePlanetDto.
2. Keep JSON property payload shape unchanged.
3. Update references in:
   - shared runtime mapping
   - designer export mapping
   - simulator/runtime consumers
   - shared and host tests

Phase 3: Compatibility and Serialization Validation
1. Confirm property names and JSON shape remain unchanged unless explicitly intended.
2. Verify existing fixtures/snapshots still deserialize and map correctly.
3. If any shape changes are unavoidable, document migration note in same change.

Phase 4: Mapping and Lifecycle Verification
1. Validate designer authored project -> clean/runtime export DTO -> shared runtime bootstrap mapping path.
2. Validate simulator/runtime behavior through existing regression gates.
3. Ensure no dependency direction violations are introduced.

Phase 5: Evidence and Pattern Capture
1. Record what changed, what stayed stable, and why.
2. Capture repeatable rename checklist for next contract type.
3. Promote checklist to follow-on phases for Country/Area/Room/etc.

## 7. Validation Gates

Minimum gates for pilot:
1. dotnet build .\StoryboardDesigner.slnx
2. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj
3. dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"

Recommended additional pilot checks:
1. Shared contract mapping tests touching runtime bootstrap and scope attachment.
2. Any clean-export snapshot tests impacted by contract type rename.

## 8. Risks and Mitigations

1. Risk: Hidden references to old type names in tests or mapping helpers.
Mitigation: workspace-wide symbol rename plus focused compile/test gates.

2. Risk: Semantic drift if naming changes imply schema shape changes.
Mitigation: keep payload property names stable in pilot; treat naming as internal API cleanup only.

3. Risk: Over-expanding pilot scope into full contract-family migration.
Mitigation: lock pilot to one type and one lifecycle proof before scaling.

4. Risk: Circular or unnecessary project reference growth during split.
Mitigation: keep contracts project dependency-light and ensure Shared depends on Contracts, not the reverse.

## 9. Exit Criteria for Pilot Completion

1. Storyboard.Shared.Contracts exists in solution and compiles.
2. RuntimePlanetDto compiles from contracts assembly and fully replaces CleanPlanetDto references for planet runtime contract usage.
3. Export -> bootstrap runtime behavior remains green in focused regression gates.
4. A reusable migration checklist exists and is validated by pilot execution evidence.
5. Team signoff that the pattern is safe to apply incrementally to next contracts.
6. For migrated enum-backed contract members, internal model/request/edit flows no longer normalize or compare enum values as freeform strings.
7. Any remaining string handling for those enum-backed members is explicitly isolated to boundary adapters (UI selection parsing or legacy JSON compatibility input paths).

## 10. Follow-On Rollout Template

After pilot success, apply the same 4-phase cycle per type family in small batches:
1. RuntimeCountryDto
2. RuntimeAreaDto
3. RuntimeRoomDto
4. RuntimeGameObjectDto

Batching rule:
1. One contract family per change where practical.
2. Keep naming/mechanical changes separate from behavioral changes.

## 11. Current Next Actions

1. Apply root-only schema version authority in implementation: keep `SchemaVersion` authoritative at runtime project root payload; treat nested schema versions as compatibility-only and remove nested writes for clean export.
2. Keep enum convergence on hold-by-default after first-pass progress; only open new convergence slices for clear duplicate-enum concepts that overlap contract enums in non-boundary model/request paths.
3. Capture concise evidence when a convergence slice is opened (duplicate symbol inventory and focused regression results).
4. Convert this plan to archived once root-only schema version behavior is implemented and validated.

## 12. Pilot Execution Snapshot (Completed This Session)

Completed:
1. Created Storyboard.Shared.Contracts and added it to StoryboardDesigner.slnx.
2. Added host references to Storyboard.Shared.Contracts from StoryboardDesigner.App and Storyboard.Simulator.
3. Added Storyboard.Shared -> Storyboard.Shared.Contracts reference and preserved boundary direction (`Shared` depends on `Contracts`).
4. Removed CleanPlanetDto and completed symbol cutover to RuntimePlanetDto.
5. Updated Designer export projection, shared runtime mapping, and tests to use RuntimePlanetDto for planet contract objects.
6. Expanded runtime contract schema/codegen coverage across the Runtime DTO family in Storyboard.Shared.Contracts.
7. Established generated `*_contract.cs` + partial `*.cs` split across runtime DTOs for consistent contract/runtime extension seams.
8. Ran validation gates:
   - dotnet build .\StoryboardDesigner.slnx
   - focused runtime/guardrail test filter in StoryboardDesigner.App.Tests

Current state note:
1. Runtime contract DTOs are now located in Storyboard.Shared.Contracts and consumed from Shared/hosts via project references.
2. Remaining first-pass work is governance/policy closeout and enum convergence completion, not base project scaffolding.

Review checkpoint:
1. Nested-versioning policy is now locked to root-only authority.
2. Next review should decide whether to execute the root-only write-path change now or bundle with next contract maintenance pass.

## 13. End Note - Default Omission Follow-Up (Deferred)

1. After the current contract transition is complete, run a focused exploration on JSON lean-output behavior for default values.
2. First-pass transition priority: apply CLR-default and null omission patterns where deterministic and safe.
3. During this first pass, prefer serializer behavior that omits `null` and CLR default values to reduce output noise.
4. Defer semantic-default omission (schema-level defaults that are not CLR defaults) to a follow-up review after transition stabilization.

## 14. Enum Convergence Rule (Locked)

1. New migrations must prefer enum-typed properties in model and request contracts whenever a corresponding runtime contract enum exists.
2. Do not introduce new NormalizeXxx(string) helpers for enum-backed fields in core model or runtime mapping code.
3. If a legacy string field is still present for compatibility, add a narrow adapter seam and mark it transitional in plan notes/tests.

## 15. Accepted Contract Immutability Rule (Locked)

1. After a contract DTO acceptance event, that accepted `*_contract.cs` shape is locked.
2. Compile/runtime fallout must be resolved in consumers (Shared runtime mapping, Designer/Simulator hosts, tests), not by rolling back accepted contract members.
3. Reopening an accepted contract requires explicit decision capture and same-change parity updates:
- schema edit,
- regenerated staging output,
- baseline refresh,
- focused regression evidence.

## 16. Post-Lock Contract Enum Coalescence Sweep (Queued End-of-Plan Task)

Trigger condition:
1. Start only after all targeted runtime DTO contracts for this effort are accepted and lock-listed.

Required tasks:
1. Run a repo-wide inventory for duplicate enumerations that represent the same concept as a runtime contract enum.
2. For each duplicate, classify usage by boundary:
- contract/runtime core,
- host model/editor state,
- persistence/read-compat adapters,
- UI-only selection/display.
3. Create a per-enum migration slice that converges core/runtime/host model state onto the contract enum type.
4. Keep string or alternate-enum conversion only in explicit boundary adapters (UI parsing and legacy read compatibility), not in core model logic.
5. Remove or deprecate duplicate enums once all non-boundary call paths are coalesced.
6. Add or update guardrail tests to prevent reintroduction of duplicate enum concepts after convergence.

Acceptance evidence:
1. For each migrated enum concept, provide symbol usage before/after counts showing duplicate enum reduction.
2. Focused runtime guardrail suite stays green:
- `GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests`.
3. Schema-emitter drift guardrails remain green for lock-listed DTO contracts.
