# Codegen Discovery Decisions

Status: Active
Date: 2026-07-29

## Locked Decisions

1. Scope kind discriminator (`scopeKind` const per concrete schema)
- Deferred for now.
- Will be revisited after first staging-generation diff pass reveals mismatch categories.

2. Partial ownership split
- Generator writes only `*_contract.cs`.
- Runtime partials (`*.cs`) are never generator-overwritten.
- Non-wire/helper behavior belongs in runtime partials.
- Structural/wire members belong in contract partials.

3. Compatibility/runtime logic placement
- Zero compatibility logic in contract partials.
- Limited mitigation logic allowed in runtime partials.

4. Generator file-system boundaries
- Source contract targets: `Storyboard.Shared.Contracts/RuntimeContracts/Dtos/*_contract.cs`
- Staging only: `CodegenManagment/Staging`
- Backups only: `CodegenManagment/Backup`
- No direct generator writes to runtime partials.

5. Manifest/rollback metadata policy
- Deferred until after first discovery pass.

6. Snapshot/test baseline timing
- Chosen policy: reconcile during first codegen staging cycle as part of mismatch triage.

7. Accepted contract immutability
- Once a staged contract file is accepted into live `*_contract.cs`, it is treated as locked.
- Fallout is resolved at use sites unless an explicit contract-change review approves reopening the contract.

## First Staging Cycle Triage Checklist

1. Generate all DTO contract outputs into `CodegenManagment/Staging`.
2. Diff staged outputs against live `*_contract.cs` targets.
3. Categorize mismatches:
- naming/type mapping
- nullability/required mismatches
- enum/discriminator mismatches
- serialization attribute shape
- default value behavior
- contract vs runtime ownership leakage
4. Log each mismatch category with one concrete example file.
5. Prioritize category fixes by blast radius and generation determinism.
6. Apply only low-risk schema/codegen rule adjustments first.
7. Re-run staging generation and diff.
8. During this cycle, evaluate snapshot drift as a triage signal (not final baseline policy change yet).
9. If a mismatch is caused by post-accept edits to a live contract file, stop and classify as governance break before applying more use-site fixes.
