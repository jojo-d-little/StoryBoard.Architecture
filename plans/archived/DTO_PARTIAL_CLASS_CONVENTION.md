# DTO Partial Class Convention

Status: Active

## Goal

Keep schema/codegen output deterministic while protecting runtime logic from generator overwrites.

## Required Pattern

1. Every runtime contract DTO must be declared as `partial`.
2. Every DTO must have exactly two files:
- `TypeName_contract.cs` for wire/contract members.
- `TypeName.cs` for runtime behavior and helper logic.

## Contract File Ownership (`*_contract.cs`)

- Class structural declaration for codegen stability (including inheritance and interface list when needed).
- Emitted contract members/properties and serialization attributes.
- Contract discriminator members that are intentionally part of serialized output.

## Runtime File Ownership (`*.cs`)

- Runtime-only behavior, computed properties, and helper methods.
- Traversal/interface adaptation helpers (for example child scope projections).
- Any mutating/internal helper logic used by runtime attachment/bootstrap.

## Rule of Thumb

If a member exists to support runtime operation (even if mild), place it in `TypeName.cs` unless it is intentionally part of emitted contract data.
