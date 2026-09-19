# Host Transport Generated Manifest Review Plan

## Objective

Review `HostTransportGeneratedManifest.ManifestJson` and its usage to confirm that the generated embedded manifest is still required, correctly owned, and aligned with the package-delivered `host-transport-manifest.json`.

## Review scope

- Identify every runtime, host, test, and consumer that calls `HostTransportGeneratedManifest.Read()` or depends on `ManifestJson`.
- Compare the embedded manifest with the packaged JSON transport manifest and define which one is authoritative.
- Confirm how assembly/package version changes flow into `ContractAssemblyVersion`.
- Evaluate whether the embedded JSON should remain generated, be loaded from the packaged file, or be replaced by a smaller typed/runtime representation.
- Preserve the existing transport route and contract drift guardrails while evaluating alternatives.

## Deliverables

- Usage and ownership map.
- Recommendation for the embedded manifest and packaged manifest relationship.
- Required code, generator, packaging, and test changes.
- Decision on whether generated artifacts should continue to be committed and validated in CI.

## Completion criteria

- One authoritative manifest source is documented.
- Version propagation and regeneration behavior are explicit.
- Consumers have a stable loading path.
- Any approved migration has a bounded implementation plan and regression coverage.
