# Enhancement Guidelines

Purpose: durable design guardrails for all future enhancements in this repository.
This is the canonical engineering policy document for architecture, boundaries, validation, and change safety.

## Session Start Checklist

1. Read this file first.
2. Confirm the requested change belongs in the correct project.
3. Preserve project boundaries.
4. Build and run targeted tests after changes.

## Project Boundaries (Must Keep)

1. `Storyboard.Shared`
- Shared domain/runtime/services used by multiple hosts.
- No WPF UI dependencies.
- Safe place for reusable simulation/game logic.

2. `StoryboardDesigner.App`
- Authoring/design-time UX only.
- Can reference `Storyboard.Shared`.
- Must not become a dependency of simulator runtime host.

3. `Storyboard.Simulator`
- Standalone simulation host (WPF MVVM).
- Can reference `Storyboard.Shared` only.
- Must not reference `StoryboardDesigner.App`.

4. `StoryboardDesigner.App.Tests`
- Regression and behavior protection.
- Add or update tests when behavior changes.

## Persistence and Export Invariants

1. Keep authored content and app state separated.
2. Native authored project content lives in project data files.
3. Project-specific UI/session state belongs in `<ProjectName>.sbe.state.json` sidecars.
4. Machine-level state (window placement/recent projects) remains app-local state, not project content.
5. Runtime export is the external contract and must exclude native app/workspace state.

## Runtime Export Contract Rules

1. Treat runtime export as schema-governed interface data.
2. Current baseline schema is `1.0` unless explicitly changed.
3. Breaking changes require explicit version-bump decision and migration notes.
4. Non-breaking additive fields are preferred over renames/removals.
5. Preserve deterministic output ordering to reduce churn and protect snapshot tests.

## Accepted Contract Lock Rule

1. Once a staged `*_contract.cs` DTO is accepted into `Storyboard.Shared.Contracts/RuntimeContracts/Dtos`, treat it as locked contract truth.
2. Do not change accepted contract member types/nullability/default attributes as a compile-fix shortcut in use-site fallout.
3. If fallout appears after acceptance, fix at use sites (mappers, models, tests, host adapters) first.
4. If a true contract issue is discovered, stop and run an explicit contract-change decision before editing accepted contracts.
5. Any approved post-accept contract edit must include: rationale, schema update, regenerated staging parity, and focused regression evidence in the same change.

## Schema-Generated DTO Protection Rules

1. `Storyboard.Shared.Contracts/RuntimeContracts/Dtos/*_contract.cs` files are schema-emitted artifacts and are not hand-authored source.
2. Do not directly edit `*_contract.cs` files to fix runtime/designer/test fallout.
3. Contract shape changes must start in `Storyboard.Shared.Contracts/RuntimeContracts/Schemas/*_contract.schema.json` (and codegen behavior only when intentional).
4. Regenerate contracts via `Storyboard.SchemaCodegen` and accept generated output; do not perform manual DTO surgery.
5. Contract lock coverage is tracked in `CodegenManagment/ContractLock/locked-contract-dtos.txt`; listed files must stay byte-equivalent (normalized line endings) to schema-emitted output.
6. If a property must remain schema-visible but should not be emitted into generated C#, set property metadata `x-csharp-emit-property: false` (or alias `x-csharp-emit: false`) in the schema.

## Runtime Export v1 Output Set

For a project `<ProjectName>.sbe.json`, runtime export outputs are:

1. `<ProjectName>.sbr.runtime.json`
2. `Area/<areaId>.runtime.json`
3. `Room/<roomId>.runtime.json`
4. `Planet/<planetId>.runtime.json`
5. `Country/<countryId>.runtime.json`
6. `GameObject/<objectId>.runtime.json`
7. `Procedure/<procedureId>.procedure.json` (when procedures are authored)

Storage exception for root scope:
1. `RuntimeProjectDto` is the logical `Global` scope node but remains serialized at `<ProjectName>.sbr.runtime.json` as the bootstrap entrypoint.
2. Do not move the global node into a `Global/` scope-kind folder unless an explicit replacement root manifest/index contract is introduced.

## Export Validation Expectations

1. Snapshot coverage should include representative fixtures beyond single-room scenarios (include Birmingham baseline coverage).
2. Contract-impacting changes require snapshot/schema validation updates in the same change.
3. Refresh snapshot baselines only intentionally using `UPDATE_RUNTIME_EXPORT_SNAPSHOTS=1`.

## Runtime Separation Rules

1. Runtime manager/session/command flow must depend on runtime contracts, not designer model graph types.
2. Keep Designer-only models and editor abstractions out of Shared runtime execution paths.
3. Keep adapters/bridges in host projects when crossing boundary seams.
4. Add guardrail tests when introducing new runtime seams.

## Placement Rules for New Work

1. Put runtime/domain behavior in `Storyboard.Shared` when both hosts need it.
2. Put host-specific UI and UX in the host project (`StoryboardDesigner.App` or `Storyboard.Simulator`).
3. Keep adapters/composition roots in host projects, not in shared.
4. Avoid duplicate business logic across host projects.

## Architectural Guardrails

1. Prefer MVVM in WPF hosts.
2. Keep ViewModels free of direct file system/UI-dialog code where practical.
3. Use service abstractions for runtime loading/export/IO seams.
4. Favor additive changes over risky rewrites.
5. Preserve existing JSON/export contracts unless explicitly changing them.

## Change Safety Standards

1. Keep patches minimal and scoped.
2. Do not break existing solution build.
3. Add/update tests for new logic and regressions.
4. Document meaningful behavioral changes in relevant plan/review markdown.
5. Avoid mixing large extraction/refactor moves with behavioral feature changes in one patch.

## Validation Commands

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`
3. `dotnet test .\Storyboard.TransportCodegen.Tests\Storyboard.TransportCodegen.Tests.csproj`

## Per-Change Smoke Gate

Run this after every code change:

1. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`

## Focused Regression Gate (Runtime Work)

Use this focused suite during runtime-boundary changes:

1. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests|SchemaCodegenHardcodedDtoGuardrailsTests|SchemaEmittedContractDriftGuardrailsTests|TransportArtifactGuardrailsTests"`
2. `dotnet test .\Storyboard.TransportCodegen.Tests\Storyboard.TransportCodegen.Tests.csproj`

## Contract and Interface Guardrail Gate

Run this gate for schema/contract/interface-affecting changes (including transport route/interface updates):

1. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "SchemaCodegenHardcodedDtoGuardrailsTests|SchemaEmittedContractDriftGuardrailsTests|ArchitectureSeparationGuardrailsTests|TransportArtifactGuardrailsTests"`
2. `dotnet test .\Storyboard.TransportCodegen.Tests\Storyboard.TransportCodegen.Tests.csproj`

## Operational Notes

1. If build reports locked output (for example MSB3026 retries), close running app/simulator processes before rebuilding.
2. Runtime-export snapshot baseline refresh uses `UPDATE_RUNTIME_EXPORT_SNAPSHOTS=1` intentionally, never by default.

## Decision Defaults

1. If a feature can live in shared without UI leakage, choose shared.
2. If ambiguous, prefer least-coupled design and ask for confirmation before cross-project coupling.
3. If a refactor is large, stage it behind small safe commits with tests.

## Plan Authoring Standard (Default)

1. For non-trivial enhancements, use the staged handoff pattern defined in `plans/PLAN_AUTHORING_STANDARD.md`.
2. Start new active plans from `plans/templates/STAGED_HANDOFF_PLAN_TEMPLATE.md`.
3. Create one handoff file per locked stage using `plans/templates/STAGE_HANDOFF_TEMPLATE.md`.
4. Do not start downstream stages until the upstream stage handoff is complete.
5. Close a workstream only after all planned stage handoffs and required validation gates are recorded.
6. Use `plans/STANDARD_STAGE_CATALOG.md` as the default stage set/order and boundary source.
7. In each new plan, include a stage inclusion matrix (`Required`/`Optional`/`Skipped`) rather than redefining stage semantics.
8. Use compatibility split for contracts: Stage 01 for backward-compatible additions, Stage 06 for non-backward-compatible contract retirement, then Stage 07 for closeout.

## Stage Boundary Enforcement (Required For Staged Plans)

When a workstream uses staged handoffs, each stage must define explicit:

1. Allowed read scope.
2. Allowed edit scope.

Default deny rule:

1. Any path not explicitly listed in allowed read/edit scope is out of scope.

Edit-implies-read rule:

1. Any path in allowed edit scope is automatically readable.
2. Allowed read scope should list only read-only dependencies outside edit scope.

Execution rules:

1. Treat stage edit scope as a hard boundary.
2. Keep reads constrained to stage read scope except narrow validation/debug reads.
3. Record any out-of-scope reads/edits in that stage handoff boundary compliance report.
4. If an out-of-scope edit seems required, stop and run a stage-boundary decision before continuing.
5. Do not perform contract removals/refactors during Stage 01; schedule them in Stage 06 retirement.
