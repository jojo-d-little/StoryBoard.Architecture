# Multi-Part Command Leg Travel Plan

Status: Complete - ready for archive
Owner: Storyboard.Shared runtime + host command binders
Last updated: 2026-07-28

## 0. Plan Status Update - 2026-07-27

Completed now:

1. Decision set remains locked (Sections 8, 14, 16, 17).
2. Producer-owned grammar guardrails were enforced in runtime/shared paths.
3. Hidden directional fallback grammar paths were removed.
4. Runtime-focused regression filter is passing after explicit test grammar seeding updates.
5. Phase A general preprocessor gates were expanded for deterministic multi-verb/full-token/producer-direction behavior.
6. Phase B binder normalization is active with mismatch/default/max-leg enforcement and canonical leg direction token emission.
7. Phase C runtime multi-leg executor behavior is active and validated for full/partial/first-leg-fail result-code paths with per-leg telemetry.
8. Full solution build is passing.

Completed:

1. Resume sequence is active from Section 20.

1. Phase E - Multi-leg regression and playback telemetry coverage expansion.

Completed since last update:

1. Phase D host compact leg summary output path is active in simulator host command console for multi-leg results.
2. Simulator recording schema now persists per-leg telemetry payloads for replay inspection.
3. Simulator-focused regression tests validate telemetry persistence and compact leg summary rendering.
4. Playback regression now captures and strictly compares per-step move-leg telemetry when strict mode is enabled.
5. First-leg-fail terminal path formatting is covered for compact host leg-summary output.
6. Producer-owned move action parameter now controls travel visualization mode (LegByLeg vs Direct) per action/object, with designer authoring UI and runtime/simulator wiring.

Close-out decisions and completion notes:

1. Added representative strict playback recording coverage with explicit multi-leg move command (`move red crate east east`) so telemetry snapshots include non-empty leg data.
2. Runtime-focused regression filter and strict simulator playback regression suite both pass after baseline refresh.
3. Workshop playback baseline keeps runtime-manager output expectations only; compact host leg-summary output remains validated in simulator host-focused tests (`SimulatorReplaySpeedSemanticsTests`) rather than manager playback snapshots.

## 1. Purpose

Define a minimal, extensible approach for ordered multi-part commands where move actions can execute N travel legs in a single command cycle.

## 2. Core Direction

1. Keep command preprocessing domain-agnostic.
2. Preprocessor captures ordered argument parts only (objects, directions, numbers, modifiers).
3. Move action binder interprets ordered parts and constructs ordered legs.
4. Runtime executor processes legs in order and returns per-leg outcomes.

## 3. v1 Scope

In scope:

1. Preserve argument order end-to-end for command parts.
2. Normalize legacy single-leg move into a one-leg list.
3. Add move binder logic that zips ordered directions and distances into legs.
4. Execute leg sequence in one command cycle.
5. Return result payload with both summary (from/to) and per-leg detail.

Out of scope for v1:

1. New movement restrictions or validators specific to knights.
2. Parser awareness of leg semantics.
3. Cross-action generalized leg semantics outside movement.

## 4. Proposed Runtime Shapes (Conceptual)

1. Ordered command parts (generic):
- objects[]
- directions[]
- numbers[]
- modifiers[]

2. Move action normalized payload (movement-specific):
- primaryObjectId
- legs[] where each leg has:
- direction
- distance
- optional per-leg options (future)

3. Move execution result:
- success
- fromPosition
- toPosition
- legs[] where each leg has:
- index
- requestedDirection
- requestedDistance
- from
- to
- success
- resultCode

## 5. Execution Policy

1. Support mode-selectable behavior.
2. Default mode is progressive stop-on-failure.
3. Atomic all-or-none remains supported as a selectable mode.

## 6. Implementation Phases (Small)

1. Phase A - Ordered argument preservation:
- ensure current command part capture preserves stable author/input order.

2. Phase B - Move binder normalization:
- convert single direction/distance into one leg.
- support multiple directions/distances as ordered legs.
- define mismatch handling policy (question set below).

3. Phase C - Runtime leg executor:
- iterate existing single-leg move mechanics per leg.
- emit per-leg result telemetry.

4. Phase D - Host output wiring:
- keep existing summary fields.
- add optional leg-details rendering path.

5. Phase E - Regression tests:
- binder tests for ordering and mismatch behavior.
- runtime tests for 2-leg and 3-leg sequences.
- playback/command output tests with leg telemetry.

## 7. Risks and Mitigations

1. Risk: parallel list mismatch bugs.
- Mitigation: normalize to a single leg list before execution.

2. Risk: breaking existing content.
- Mitigation: one-leg backward-compatible normalization path.

3. Risk: host/UI churn from richer outputs.
- Mitigation: preserve from/to summary and make leg details additive.

## 8. Locked Decisions

1. Execution mode is selectable; default is progressive stop-on-failure.
2. Direction and distance count mismatch fails binding (no pad/truncate behavior).
3. If command supplies zero distances, use action-configured default distance for all directions.
4. If zero distances are supplied and no action default is configured, binding fails.
5. Single target object applies to all legs (permanent design rule).
6. Maximum legs per move command is 10.
7. Per-leg telemetry is persisted in replay/session outputs (additive to summary fields).
8. Partial completion is not a generic failure; use distinct action result codes for full success, partial success, and failure.
9. Keep existing move action name; extend payload additively for legs and preserve strong single-move compatibility.
10. Parser/preprocessor remains filler-tolerant and domain-agnostic; move binder constructs legs from ordered parsed parts.
11. Multi-leg commands do not require explicit separators; ordered extraction of objects/directions/numerics drives binding.
12. Distances must be positive integers; zero/negative values fail binding.
13. Movement restrictions are evaluated per leg using current single-leg rule semantics.
14. Host UI shows compact per-leg summaries by default with expandable detail.

## 9. Question Count

Total decision questions: 12

## 10. Next Discussion

1. Resume from Section 19 checkpoint and execute Section 20 first-step sequence before starting new multi-leg binder/executor code.

## 11. Command Preprocessor Refinements (Locked Direction)

1. Verb detection is no longer position-locked to the first token.
2. Parser scans the full input and selects the best verb match anywhere in the string.
3. Command preprocessing remains domain-agnostic; it extracts ordered matched parts and does not construct movement legs.
4. Accepted matches are masked in-place in a working buffer to prevent duplicate rematch while preserving original spatial offsets.
5. Token order is derived from original start offsets captured at match time.

Implementation guardrails:

1. Keep original raw input immutable for diagnostics and user-visible messages.
2. Maintain a separate mutable working buffer used for match-and-mask passes.
3. Mask exactly the accepted span length so downstream ordering remains stable.
4. Apply word-boundary rules and longest-match-first behavior for dictionary-driven token classes to reduce partial collisions.
5. If multiple candidate verbs are present, use deterministic tie-breaking:
- highest-confidence canonical verb first
- then earliest start position
- then longest match

Locked scope notes:

1. Multiple verbs in one command is in scope now and must have deterministic selection behavior.
2. Quoted-text parsing behavior is out of scope for this slice and deferred until a concrete requirement exists.
3. Name-substring conflict handling beyond current token boundary logic is out of scope for this slice and deferred until a concrete requirement exists.

Tokenizer/matcher rule additions:

1. Full-token matching is required; partial substring matches are not allowed (for example move does not match remove).
2. Longest-match-first is required for object/entity phrase matching so multi-token names are captured as a unit when available (for example red box should match as one object phrase, not two independent tokens).

## 12. Design Questions - Determinism and Validation

1. When multiple object phrases are present in one command, what deterministic selection/ordering policy should binders use?
2. How should direction alias precedence be resolved (for example north vs n, northeast vs north east) under full-token and longest-match rules?
3. Should punctuation be normalized to whitespace before tokenization, and if so what characters are included in v1 normalization?
4. What explicit error-code taxonomy should separate parse/bind errors from runtime move outcomes?
5. What minimum v1 golden test matrix is required before implementation completion?

## 13. Design Question Count

Additional design questions: 5

## 14. Locked Clarification - Object Mentions

1. Existing command semantics are preserved: the first mentioned object is primary and subsequent mentioned objects are ordered secondary objects.
2. Secondary objects remain supported as an ordered list (N secondaries).
3. Multi-leg travel changes do not alter this object-mention model.

## 16. Locked Boundary - Preprocessor vs Action Responsibilities

1. Command preprocessing remains generic and action-agnostic.
2. Preprocessor responsibilities are limited to:
- token matching
- match masking
- ordered part extraction
- emitting structured parsed results
3. Preprocessor must not enforce movement-leg semantics such as:
- leg count rules
- direction/distance completeness checks
- default leg distance application
- movement-specific validation policies
4. All move/leg semantics are action-layer responsibilities (binder and runtime), not preprocessor responsibilities.
5. This boundary applies to current and future actions to prevent command-preprocess drift into action-specific parsing.

## 17. Locked Rule - Verb Cardinality

1. v1 requires exactly one actionable verb per command input.
2. Zero matched verbs is an error.
3. More than one matched verb is an error (no heuristic first-verb fallback in v1).
4. Multi-verb errors should report matched verbs and positions to support author correction.

## 18. Locked v1 Test Gates (Split Lists)

General command preprocessor tests:

1. Verb can be matched anywhere in input, not only first token.
2. Exactly one actionable verb is required; zero verbs errors.
3. More than one actionable verb errors deterministically with matched positions.
4. In-place masking prevents already-matched spans from being re-matched.
5. Full-token matching blocks substring collisions (for example move does not match remove).
6. Longest-match phrase capture works for multi-token object names (for example red box as one object token).
7. Ordered extraction preserves original mention order across token types.
8. Primary/secondary object ordering is preserved from mention order.
9. Producer-defined directional vocabulary is authoritative for direction matches.
10. Hardcoded direction fallback only applies when producer list does not match.
11. Punctuation normalization uses Shared configuration mapping.
12. Filler words are ignored without changing extracted token order.
13. No action-specific validation is performed in preprocessing.

Movement action specific tests:

1. Legacy single-move compatibility remains unchanged for one direction plus one distance.
2. Legacy single-move normalization emits exactly one leg.
3. Zero supplied distances with configured action default applies that default to all directions.
4. Zero supplied distances with no action default fails at bind stage.
5. Direction and distance count mismatch fails at bind stage.
6. Zero or negative distance fails at bind stage.
7. Leg cap of 10 is enforced.
8. Full multi-leg completion returns full-success result code.
9. Partial multi-leg completion returns partial-success result code with stopped leg index.
10. First-leg blocked path returns failure with no movement.
11. Overall toPosition equals last successful leg destination.
12. Per-leg movement restrictions reuse existing single-leg evaluation semantics.
13. Per-leg telemetry is emitted and persisted in replay/session outputs.
14. Existing summary result fields remain stable for one-leg commands.
15. Host compact leg summary output is correct for full and partial outcomes.

## 15. Locked Clarification - Punctuation Normalization Configuration

1. Punctuation normalization behavior is configuration-driven, not hardcoded.
2. Configuration lives in Shared so both hosts consume the same normalization policy.
3. Config explicitly defines:
- characters/tokens to normalize
- replacement value per token (default expected to be space, but configurable)
4. Parser consumes this configuration at runtime when building the working normalization buffer.

## 19. Session Checkpoint - 2026-07-27 (Preprocessor Foundation Refactor)

Status impact:

1. Multi-leg travel implementation was intentionally paused while command preprocessing foundations were cleaned up.
2. This refactor is complete enough to use as the new baseline for resuming the multi-leg plan.
3. Solution validation is currently green at checkpoint time.

Preprocessor and command-pipeline changes that affect this plan:

1. Legacy post-verb boundary markers were retired from active flow.
2. Matching behavior moved further toward consumed-token semantics instead of boundary-index semantics.
3. Directional handling now distinguishes navigational direction from non-navigational directional qualifier capture.
4. Object and synonym resolution precedence was adjusted so object interpretation happens before adjacent-room qualifier capture.
5. Action-trigger directional matching moved to semantic parsed fields (direction token set and qualifier fields) rather than depending on residual-token heuristics.
6. Workshop playback baselines were refreshed to align with the corrected diagnostics index behavior after preprocessing changes.

Test and fixture baseline updates relevant to resume confidence:

1. Several tests were adjusted to explicitly seed verb vocabulary in fixtures (removal of implicit/default command support assumptions).
2. Runtime movement and selection fixture updates were completed and validated.
3. Simulator smoke selector drift was corrected and full suite returned to green.

Architecture note captured during this checkpoint:

1. A deferred design path was captured to allow action-owned requirement clarifications (processor-orchestrated), rather than preprocessor action-specific branching.
2. See plans/active/ACTION_OWNED_CLARIFICATION_REQUESTS_PLAN.md for deferred follow-up discussion.

## 20. Resume First - Ordered Restart Steps

1. Re-read this plan Sections 2, 8, 16, 17, 18, and 19 to re-lock boundary assumptions before coding.
2. Run the full suite once to confirm baseline parity before new multi-leg edits.
3. Start implementation at Phase A with no leg semantics added yet:
- verify ordered argument preservation and deterministic extraction behavior under current preprocessor baseline.
4. Add or confirm focused tests from Section 18 (general preprocessor list) before binder work.
5. Proceed to Phase B binder normalization only after Phase A tests are stable.
6. Keep clarification-system redesign deferred; do not mix ACTION_OWNED_CLARIFICATION_REQUESTS_PLAN work into initial multi-leg execution slices.
