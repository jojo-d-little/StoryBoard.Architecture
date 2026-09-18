# Runtime Contract DTO Migration Matrix

Status: Closed 2026-08-01 - First-pass migration complete; matrix retained for historical traceability
Owner: Runtime contracts naming + structure alignment
Last updated: 2026-08-01

## Closeout Snapshot (2026-08-01)

1. Runtime DTO naming migration (`Clean*` -> `Runtime*`) is complete.
2. Runtime schema coverage expanded well beyond the initial pilot.
3. Contract-generated plus partial-class seam pattern is established for runtime DTOs.
4. Root-only schema version authority is locked.
5. Enum convergence remains maintenance-mode and should advance only on clear duplicate-overlap findings.

## Purpose

Editable checklist for DTO naming and structure alignment before mechanical rename/split batches.

Review workflow:
1. Use this file as the post-rename baseline checklist.
2. Confirm any remaining policy choices (especially schema versioning behavior).
3. Execute only the remaining structural/schema steps listed below.

## Conventions (Current Draft)

1. Replace Clean* DTO prefixes with Runtime*.
2. Remove V1 from type names unless versioning in type name is intentionally required.
3. Keep schema version governance at root-level contracts; nested schema versions are legacy-compatible until retired.
4. Use partial split where non-contract/runtime-only members exist.
5. For enum-backed contract members, use enum types end-to-end in host/shared model and request flows; keep string conversion only at boundary adapters.

## Migration Execution Status (Current)

Completed in codebase:
1. All `Clean*` DTO type names were migrated to `Runtime*` names.
2. Scope utility/result types were migrated from `CleanRuntime*` to `Runtime*`.
3. `RuntimePlanetDto` and `RuntimeScopeNodeBaseDto` are aligned to target names.
4. Partial split completed for scope-node DTOs with runtime behavior seams:
	- `RuntimePlanetDto`
	- `RuntimeAreaDto`
	- `RuntimeCountryDto`
	- `RuntimeGameObjectDto`
	- `RuntimeRoomDto`
5. Runtime contract schema coverage has expanded beyond the planet prototype to the broader runtime DTO family in `Storyboard.Shared.Contracts/RuntimeContracts/Schemas`.
6. Generated `*_contract.cs` plus partial `*.cs` seam pattern is in place broadly across runtime DTOs, including root payload DTOs.
7. Current dependency direction is aligned: `Storyboard.Shared` references `Storyboard.Shared.Contracts`; hosts reference both as needed.

Still pending:
1. Execute the locked root-only schema versioning behavior in implementation: nested `SchemaVersion` fields stay read-compatible but are no longer authoritative or required for clean export writes.
2. Keep enum convergence in maintenance mode: only run additional slices when a clear duplicate enum concept overlaps a runtime contract enum in non-boundary model/request paths.

## Enum Convergence Checklist (Apply Per Migrated Enum Field)

1. Contract DTO member type is enum (source of truth).
2. Shared runtime model state uses enum type, not string.
3. Designer/simulator model state uses enum type, not string.
4. Edit/request records use enum type, not string.
5. Mapping code performs string conversion only at explicit external boundaries.
6. Tests assert enum values in model/request layers and string values only in serialized payload assertions.

## Matrix (Historical Mapping)

| Current Type | Proposed Name (Editable) | Kind | Scope Node Derived | SchemaVersion Field | Partial Split Needed | Reviewer Notes |
|---|---|---|---|---|---|---|
| CleanAreaDto | RuntimeAreaDto | class | yes | no | yes | |
| CleanAreaNavigationDto | RuntimeAreaNavigationDto | class | no | no | no | |
| CleanCompositePartRequirementDto | RuntimeCompositePartRequirementDto | class | no | no | no | |
| CleanCompositeRecipeDto | RuntimeCompositeRecipeDto | class | no | no | no | |
| CleanCountryDto | RuntimeCountryDto | class | yes | no | yes | |
| CleanGameObjectAppearanceDto | RuntimeGameObjectAppearanceDto | class | no | no | no | |
| CleanGameObjectDto | RuntimeGameObjectDto | class | yes | no | yes | |
| CleanGamePropertyDefinitionDto | RuntimeGamePropertyDefinitionDto | class | no | no | no | |
| CleanLockKeyRequirementDto | RuntimeLockKeyRequirementDto | class | no | no | no | |
| CleanLockOperationRequirementsDto | RuntimeLockOperationRequirementsDto | class | no | no | no | |
| CleanLockParticipantVariableRequirementDto | RuntimeLockParticipantVariableRequirementDto | class | no | no | no | |
| CleanNavigationExportV1Dto | RuntimeNavigationDto | class | no | yes (root) | no | Consider keeping version property as authoritative for navigation payload root. |
| CleanObjectImageVariantDto | RuntimeObjectImageVariantDto | class | no | no | no | |
| CleanObjectMovementRestrictionCategoryDto | RuntimeObjectMovementRestrictionCategoryDto | class | no | no | no | |
| CleanObjectMovementRestrictionRuleDto | RuntimeObjectMovementRestrictionRuleDto | class | no | no | no | |
| CleanObjectMovementRestrictionsDto | RuntimeObjectMovementRestrictionsDto | class | no | no | no | |
| CleanProcedureDefinitionDto | RuntimeProcedureDefinitionDto | class | no | no | no | |
| CleanProcedureParticipantMutationDto | RuntimeProcedureParticipantMutationDto | class | no | no | no | |
| CleanProcedureParticipantRequirementDto | RuntimeProcedureParticipantRequirementDto | class | no | no | no | |
| CleanProjectExportV1Dto | RuntimeProjectDto | class | no | yes (root) | no | Selected target from discussion. |
| CleanCommandActionDto | RuntimeCommandActionDto | class | no | no | no | |
| CleanCommandActionLinkDto | RuntimeCommandActionLinkDto | class | no | no | no | |
| CleanCommandActionReferenceDto | RuntimeCommandActionReferenceDto | class | no | no | no | |
| CleanRoomCommandDto | RuntimeRoomCommandDto | class | no | no | no | |
| CleanRoomExportV1Dto | RuntimeRoomDto | class | yes | yes (nested legacy) | yes | Nested SchemaVersion should be treated legacy/non-authoritative if root-only policy is adopted. |
| CleanRoomImageDto | RuntimeRoomImageDto | class | no | no | no | |
| CleanRoomLinkDto | RuntimeRoomLinkDto | class | no | no | no | |
| CleanRoomPlacementDto | RuntimeRoomPlacementDto | class | no | no | no | |
| CleanRuntimeScopeAttachmentResult | RuntimeScopeAttachmentResult | class | no | no | no | Utility result type; confirm desired Runtime prefix. |
| CleanRuntimeScopeAttachmentUtility | RuntimeScopeAttachmentUtility | static class | no | no | no | Utility type; confirm desired Runtime prefix. |
| RuntimePlanetDto | RuntimePlanetDto | partial class | yes | no | done | Aligned and split. |
| RuntimeScopeNodeBaseDto | RuntimeScopeNodeBaseDto | abstract class | n/a | no | n/a | Aligned rename complete. |

## Excluded (Not DTO Contracts)

1. FlexibleStringJsonConverter
- Current: internal utility converter
- Recommendation: keep name as-is; not part of DTO rename matrix

## Next Batch Order (Post-Rename)

1. Batch D: Schema policy lock + targeted compatibility notes
- Decide root-only authoritative versioning and nested `SchemaVersion` write/read policy.
2. Batch E: Enum convergence completion slices
- Execute per-enum convergence for shared/host model and request surfaces; keep string conversion only at explicit boundary adapters.
3. Batch F: Closeout evidence
- Capture before/after symbol counts for duplicate enum reduction and attach focused regression evidence.

## Versioning Decision Placeholder

Locked decision:
1. Root-level only authoritative versioning: yes.
2. Nested SchemaVersion write policy: stop writing nested schemaVersion in clean export payloads; keep read tolerance for existing artifacts.
3. Type naming and version suffix policy: allow V1 in type names only when parallel major versions coexist.
