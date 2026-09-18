# Simulator Test Automation Enhancement Plan

Status: Proposed
Owner: Pending
Last updated: 2026-06-29

## Purpose
Define a staged plan to add strong simulator-host automation for runtime replay validation without implementing the feature yet.

This plan captures:
1. Command-line startup parameters for simulator-host validation runs.
2. Optional replay and comparison workflows.
3. Deterministic pass/fail behavior suitable for CI and local proving.
4. Separation-safe architecture so simulator remains independent from designer project references.

## Why This Matters
1. Current unit tests validate replay behavior in test harnesses, but we also want host-level proof in simulator execution paths.
2. A parameterized simulator validation mode can provide heavier end-to-end confidence.
3. The same inputs used by regression tests can be replayed through simulator-host orchestration.

## Existing Baseline To Reuse
1. Replay-related coverage in test project:
- StoryboardDesigner.App.Tests/GameSimulatorPlaybackRegressionTests.cs
- StoryboardDesigner.App.Tests/SharedManagerHostFixtureTests.cs
2. Shared runtime manager path already validated for separation:
- StoryboardDesigner.App.Tests/ArchitectureSeparationGuardrailsTests.cs

## Scope
In scope:
1. Simulator startup arguments for project load and optional replay runs.
2. Optional compare baseline inputs and actual-output artifact generation.
3. Live compare option during replay.
4. Deterministic status/exit behavior for automation.

Out of scope for initial slice:
1. New game authoring features.
2. Major UI redesign.
3. Plugin architecture for arbitrary replay adapters.

## Proposed Execution Modes
1. Interactive mode (default): current simulator behavior.
2. Replay mode: load project and replay provided session file.
3. Validate mode: replay with expected-output comparison and automation-oriented result.

## Proposed CLI Surface (Draft)
Required:
1. --project <path>

Optional:
1. --replay <path>
2. --mode interactive|replay|validate
3. --compare-baseline <path>
4. --write-comparison <path>
5. --live-compare true|false
6. --stop-on-first-diff true|false
7. --fail-on-diff true|false
8. --output-format text|json
9. --trace <path>

## Comparison Strategies (Planned)
1. End-of-run comparison:
- Compare final output artifact against baseline.

2. Live comparison:
- Compare step-by-step as commands replay.
- Optional early stop on first mismatch.

3. Comparison outputs:
- Human-readable summary.
- Machine-readable JSON for CI parsing.

## Determinism Requirements
1. Replay processing must produce deterministic ordering and stable output.
2. Output schema should be versioned if persisted.
3. Comparison should normalize known non-deterministic noise if any appears.
4. Exit codes must map to stable result categories.

## Proposed Exit Codes
1. 0: Validation passed.
2. 1: Load/runtime execution error.
3. 2: Comparison mismatch.
4. 3: Invalid arguments/configuration.

## Architecture Direction
1. Keep replay/compare orchestration in reusable shared runtime services where practical.
2. Keep simulator as a host/composition layer, not business-rule owner.
3. Preserve existing boundary rule: simulator depends on Storyboard.Shared and not StoryboardDesigner.App.
4. Extend architecture guardrails if new files/services are added.

## Phased Delivery Plan

### Phase S1: Minimal argument parsing and startup mode wiring
Deliverables:
1. Parse project and optional replay arguments in simulator startup.
2. Support non-interactive replay mode invocation.
3. Emit clear success/failure summary.

Acceptance:
1. Simulator can load project by argument and replay a provided session file.
2. Existing interactive mode remains unchanged when no args are provided.

### Phase S2: Comparison artifact and baseline diff
Deliverables:
1. Add compare-baseline and write-comparison options.
2. Implement deterministic comparison and diff output.
3. Add fail-on-diff behavior.

Acceptance:
1. Replay validation can run headlessly and produce deterministic pass/fail.
2. Mismatch details are actionable.

### Phase S3: Live compare and step diagnostics
Deliverables:
1. Live compare option with optional stop-on-first-diff.
2. Per-step diagnostics and trace output.

Acceptance:
1. Early mismatch detection is available.
2. Diagnostics identify exact command step and output delta.

### Phase S4: CI and workflow integration
Deliverables:
1. Add documented automation command examples.
2. Add CI task or script entry for simulator validation run.
3. Add guardrail test updates as needed.

Acceptance:
1. CI can fail on replay mismatch.
2. Local and CI runs produce consistent results.

## Test Plan
1. Unit tests for argument parsing and mode selection.
2. Integration tests for replay and compare workflows.
3. Regression fixture reuse from existing simulator playback test data.
4. Guardrail tests to ensure no designer dependency is introduced in simulator.

## Risks and Mitigations
Risk: Output instability causes noisy diffs.
- Mitigation: deterministic formatting, stable ordering, and targeted normalization policy.

Risk: Host-specific logic divergence from shared runtime behavior.
- Mitigation: keep orchestration shared where possible and validate against existing fixture paths.

Risk: Scope expansion into a full toolchain.
- Mitigation: stage by phases S1-S4 with explicit acceptance gates.

## Open Questions
1. Should validate mode run within WPF host only, or also via a dedicated console host later?
2. What should the baseline artifact schema include beyond output lines (state deltas, action ids, diagnostics)?
3. Should live compare stop immediately by default or continue to collect all diffs?
4. Should mismatch reporting include strict byte-level output or semantic equivalence mode?

## Definition of Done (for initial automation milestone)
1. Simulator accepts project plus replay arguments.
2. Validation mode supports baseline comparison and deterministic exit code behavior.
3. At least one reusable fixture path demonstrates replay compare pass/fail behavior.
4. Simulator architectural separation guardrails remain green.

## Suggested First Implementation Slice When Resumed
1. Implement S1 only.
2. Add small parser tests and one replay invocation integration check.
3. Keep all compare behavior deferred to S2.
