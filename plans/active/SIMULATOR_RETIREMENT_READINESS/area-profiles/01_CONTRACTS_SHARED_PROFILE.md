# Contracts / Shared Area Profile

Purpose: boundary profile for any pass that changes a shared schema, DTO, transport interface, or shared runtime contract.

## Ownership And Compatibility

1. `Storyboard.Contracts` is the authoritative home for approved cross-project contracts.
2. Contract-design approval precedes contract implementation; contract implementation and compatibility validation precede consumer work.
3. Additive compatibility is mandatory. Non-backward-compatible retirement is globally gated and outside readiness work unless the explicit retirement profile is reopened.

## Default Scope

- Primary edit areas: `Storyboard.Contracts/Storyboard.Shared.Contracts/**`, `Storyboard.Contracts/Storyboard.SchemaCodegen/**`, `Storyboard.Contracts/Storyboard.TransportCodegen.Tests/**`.
- Read-only dependencies are named by the individual pass; do not infer consumer implementation scope.

## Required Pass Evidence

1. Approved interface/specification reference.
2. Added/changed contract inventory and compatibility declaration.
3. Schema/code-generation and contract guardrail results.
4. Exact next consumer pass and its permitted contract surface.

## Exclusions

1. No consumer implementation work in a Contracts pass.
2. F01 uses the established runtime-registration JSON format and does not create a contract pass unless a real new shared boundary is discovered.
