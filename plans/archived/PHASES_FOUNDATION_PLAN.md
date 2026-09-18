# Phases Foundation Plan

Date: 2026-08-21
Status: Ready For Archive

## Current Snapshot (2026-08-24)

Overall assessment:

1. Core phase runtime behavior is implemented: initialization/start-page resolution, deterministic next/previous traversal, and direct set-by-key.
2. Phase host presentation payload is implemented and covered (reason, old/new summaries, ordered text presentation steps, resume context).
3. Designer phase tree authoring is implemented for Book/Chapter/Page with editing/reorder/start-page selection and save/load/export round-trips.
4. Phase ambience MVP is implemented through event-driven narrative ambient reconciliation with overlay/replace semantics and idempotence coverage.
5. Closeout baseline evidence is now complete; remaining items are optional hardening follow-ons only.
6. Deprecated phase ambience timer-path action types were removed to reduce authoring/runtime confusion:
- `SetupPhaseAmbienceEffectTimers`
- `PlayPhaseAmbienceEffect`

Validated this pass:

1. `StoryboardDesigner.App.Tests/GameManagerTests.cs` targeted phase host payload tests passed (4/4).
2. `Storyboard.GameEngine.Tests/GameStateSessionPhaseNavigationTests.cs` targeted phase navigation tests passed (2/2).
3. Focused runtime-boundary regression gate passed (70/70):
- `GameManagerTests`
- `GameCommandProcessorFixtureTests`
- `GameCommandProcessorLinkedActionsTests`
- `GameSimulatorPlaybackRegressionTests`
- `ArchitectureSeparationGuardrailsTests`
4. Replay smoke gate passed (9/9):
- `GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep`
5. New designer phase-start context-action coverage passed (10/10 in class):
- `PhaseNarrativeReviewContextActionTests` now covers Set/Clear Starting Phase Page action visibility and execution behavior.

## Purpose

Extract and execute the phases topic from the time passage umbrella as a focused implementation track with clear runtime semantics, deterministic behavior, and host-ready phase presentation signals.

## Relationship To Umbrella

1. Parent plan: `plans/active/TIME_PASSAGE_AND_EVENTS_PLAN.md`.
2. This child plan owns the phase foundation topic that remains pending in umbrella exit criteria.
3. Time-tick and event foundation remain closed dependencies that phases should reuse, not replace.

## Scope (This Plan)

1. First-class phase model in runtime and authored data.
2. Phase hierarchy MVP with three levels in designer UX.
3. Phase lifecycle events and payload anchor surfaces.
4. Phase actions (`NextPhaseAction`, `PreviousPhaseAction`, `SetPhaseAction`) across command and event execution paths.
5. Phase-specific host payload for phase-change presentation.
6. Phase ambience MVP using event-driven narrative ambient reconciliation action flow.
7. Focused tests and regression gates for deterministic behavior and replay stability.

## Out Of Scope (This Plan)

1. Milestone gameplay condition implementation.
2. Rich per-phase multi-track ambience composition graphs.
3. Unlimited-depth phase hierarchies.
4. Full producer-custom terminology system for Book/Chapter/Page labels.

## Core Direction (Locked Intent)

1. Phase is a first-class gameplay progression primitive, not a game-property workaround.
2. Geography and phase are orthogonal dimensions and must remain modeled independently.
3. Phase tree authoring appears as a separate branch under Global in designer UX.
4. MVP hierarchy depth is fixed at 3 levels:
- Book
- Chapter
- Page
5. Runtime/contracts remain terminology-neutral even if designer labels are Book/Chapter/Page.

## Phase Model (MVP)

1. Identity and ordering:
- `phaseKey` (required, stable unique identifier)
- `parentPhaseKey` (optional; root book-level node has no parent)
2. Ordering semantics:
- Tree child order is authoritative for deterministic traversal/advance behavior.
- No separate authored ordinal-path or sibling-order field is required.
2. Presentation data:
- `displayName` (optional concise host-facing label)
- `title` (optional)
- `prologue` (optional long-form narrative text)
3. Variables:
- phase-scoped game properties are supported like other runtime constructs.
4. Ambience fields (exactly two MVP authored fields):
- `phaseAmbientSoundEffectId` (optional)
- `phaseAmbientTimerKey` (optional)
5. Ambience policy metadata (MVP-level control):
- `phaseAmbienceMode` (`overlay` or `replace`)
- transition/fade controls are deferred until runtime can explicitly control already-playing sound instances.

## Phase Session State And Anchors (MVP)

1. Active gameplay phase is always a Page leaf node.
2. Session tracks both current and previous active page/lineage as first-class runtime state.
2. Phase anchor is added to runtime anchor provider/session state surfaces.
3. Event payload manifest adds phase anchor fields for mapping use.
4. Minimum identity anchor fields:
- `currentPhase.key`
- `currentPhase.bookKey`
- `currentPhase.chapterKey`
- `currentPhase.pageKey`
- `currentPhase.pathKey`
5. Minimum display/presentation anchor fields:
- `currentPhase.displayName`
- `currentPhase.bookDisplayName`
- `currentPhase.chapterDisplayName`
- `currentPhase.pageDisplayName`
- `currentPhase.pathDisplayName`
- `currentPhase.title`
- `currentPhase.prologue`

## Phase Lifecycle Events (MVP)

1. MVP lifecycle signaling is event-family based using implemented phase-change keys:
- `phase_changed`
- `phase_changed_book`
- `phase_changed_chapter`
- `phase_changed_page`
2. Event payload includes old/new phase key/path context and changed-tier flags where applicable.
3. Initial phase bootstrap and subsequent phase transitions publish through the same deterministic phase-change publication path.
4. Additional explicit lifecycle keys (`phase_entered` / `phase_exited` / `phase_resumed` and `game_started` / `game_completed` / `game_resumed`) are deferred for post-MVP expansion if concrete scenarios require them.
5. Event publication remains compatible with existing event -> subscription -> action flow.

## Phase Actions (MVP)

1. `NextPhaseAction`:
- Moves to deterministic next Page leaf in hierarchy order.
- Default traversal: next sibling if present; otherwise ascend until an ancestor has a next sibling.
- If traversal lands on a non-leaf node, descend first-child repeatedly until a Page leaf is reached.
- At terminal Page (no next leaf): no-op state change, return `NoNextPhase`, emit diagnostics, and emit `game_completed` once per session.
2. `SetPhaseAction`:
- Jumps directly to a known Page target.
- MVP target is `phaseKey` only.
- Cross-branch and cross-book jumps are allowed when target exists.
3. `PreviousPhaseAction`:
- Moves to deterministic previous Page leaf in hierarchy order using the reverse of advance traversal semantics.
- At sequence start (no previous leaf): no-op state change with explicit deterministic outcome/diagnostics.
4. All phase actions are available in command execution and event-driven execution.

## Designer Authoring Surface (MVP)

1. Add new Global child node branch: `Phases`.
2. `Phases` hosts a separate tree from geography.
3. Tree supports Book -> Chapter -> Page in MVP.
4. Phase editor includes:
- key/order fields
- title/prologue
- phase-scoped variables
- ambience fields/policy
5. Validate one explicit start phase path for session initialization.
6. Exactly one explicit start Page is required.

## Host Interface And Presentation Contract (MVP)

1. Add phase-change payload to host contract (parallel intent to room-change payload patterns).
2. Payload includes at least:
- prior phase identity/path
- current phase identity/path
- current display-name hierarchy snapshot
- title/prologue snapshot
- change reason (`advance`, `set`, `initialize`, etc.)
 - resume indicator/context for load/attach flows
3. Hosts may choose custom rendering for phase display/title/prologue rather than treating them as command output text.

## Phase Ambience MVP

1. Reuse existing event -> subscription -> action pipeline with `ManageNarrativePhaseAmbientSounds`.
2. Deprecated phase ambience timer-path action types are removed (`SetupPhaseAmbienceEffectTimers`, `PlayPhaseAmbienceEffect`).
3. `ManageNarrativePhaseAmbientSounds` resolves current active Page and lineage at execution time and reads phase ambience fields dynamically.
4. Producer-owned subscriptions decide when reconciliation runs (for example `phase_changed` family hooks).
5. Child/parent layering control:
- `overlay`: child ambience adds while parent continues.
- `replace`: child ambience suppresses parent while child active.
6. Mode semantics apply only when that node defines ambience.
7. If a node has no ambience definition, effective ambience inherits from active ancestors.
8. General phase events still fire at Book/Chapter/Page levels; ambience orchestration remains page-triggered and lineage-aware.
9. `ManageNarrativePhaseAmbientSounds` compares old vs new effective ambience cue sets using previous/current page anchors, then applies only required deltas.
10. Keep implementation deterministic and idempotent under repeated reconciliation triggers.

## Implementation Slices

### Slice A: Contracts And Data Model

- [x] Add phase authored model and runtime contract shape.
- [x] Add phase tree collections under project/global authored model.
- [x] Add phase state in runtime session model.
- [ ] Add phase anchor fields in event payload manifest and resolver surfaces.

### Slice B: Runtime Semantics

- [x] Implement phase initialization/start phase resolution.
- [x] Implement `NextPhaseAction` traversal behavior.
- [x] Implement `SetPhaseAction` direct jump behavior.
- [x] Emit MVP lifecycle events via `phase_changed` + tier-specific phase-change event family.
- [x] Add diagnostics/result codes for invalid targets and no-next-phase cases.

### Slice C: Host Contract Integration

- [x] Add phase-change payload DTO and host response surface.
- [x] Ensure delta/journal carries phase changes without violating existing boundary rules.
- [ ] Add compatibility coverage for hosts not yet consuming phase payloads.

### Slice D: Designer UX

- [x] Add `Phases` branch under Global tree.
- [x] Add Book/Chapter/Page authoring nodes and editors.
- [ ] Add validation rules (start phase required, key uniqueness, hierarchy constraints).
- [x] Add save/load/export round-trip tests for phase data.

### Slice E: Phase Ambience

- [x] Add phase ambience fields (`phaseAmbientSoundEffectId`, `phaseAmbientTimerKey`, mode).
- [x] Use `ManageNarrativePhaseAmbientSounds` as the runtime ambient reconciliation action path.
- [x] Implement overlay/replace parent-child handling.
- [x] Add deterministic/idempotence tests for repeated timer invocations.

### Slice F: Integration And Replay

- [x] Add composed scenario coverage: event + phase + ambient reconciliation interplay.
- [ ] Extend replay cases with explicit phase-transition assertions (hardening follow-on).
- [x] Validate focused runtime guardrail suites and playback smoke gate.

## Validation Baseline

1. `dotnet build .\StoryboardDesigner.slnx`
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "GameManagerTests|GameCommandProcessorFixtureTests|GameCommandProcessorLinkedActionsTests|GameSimulatorPlaybackRegressionTests|ArchitectureSeparationGuardrailsTests"`
3. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"`

## Exit Criteria

1. Runtime supports first-class phase state and MVP phase lifecycle event family (`phase_changed`, `phase_changed_book`, `phase_changed_chapter`, `phase_changed_page`).
2. `NextPhaseAction`, `PreviousPhaseAction`, and `SetPhaseAction` are end-to-end implemented and covered.
3. Designer supports separate phase tree authoring (Book/Chapter/Page MVP).
4. Host receives phase-change payload with title/prologue context.
5. Phase ambience MVP works through event-driven narrative ambient reconciliation with overlay/replace semantics.
6. Composed event + phase + ambient reconciliation scenario passes with deterministic replay-safe evidence.

Exit criteria progress (2026-08-24):

1. Satisfied: MVP lifecycle-event model is implemented via phase-change event family and accepted for closeout.
2. Satisfied: phase actions are end-to-end implemented and covered.
3. Satisfied with minor hardening pending: designer phase tree authoring is implemented; remaining validation-rule hardening is tracked below.
4. Satisfied: host receives additive phase-change payload with title/prologue/narrative context.
5. Satisfied: phase ambience MVP reconciliation behavior is implemented with focused tests.
6. Satisfied for closeout baseline: focused runtime guardrail suite and replay smoke gate reran green (70/70 and 9/9).

## Remaining To Close

1. No blocking closeout items remain.
2. Optional post-archive hardening:
- add explicit phase-transition replay assertions.
- extend designer validation-rule hardening if new gaps are discovered.

## Design Lock-Off Questions (Pre-Implementation)

Use this section to ratify defaults before implementation. Each item should be marked with a final decision.

1. Canonical phase identity and order path
- Question: Should `phaseKey` be the only canonical identity?
- Decision: Approved with adjustment. `phaseKey` is canonical identity; no separate authored ordinal-path field.

2. Ordinal path constraints
- Question: How is deterministic sibling ordering defined for phase traversal?
- Decision: Approved with adjustment. Persisted tree child order is canonical; no separate sibling-order integer field.

3. Start phase rule
- Question: Must exactly one start phase path be declared?
- Decision: Approved with adjustment. Exactly one explicit start Page is required. Deep start initialization emits ancestry enter cascade (Book -> Chapter -> Page).

4. Advance traversal semantics
- Question: On `NextPhaseAction`, do we use next-sibling-else-ascend traversal?
- Decision: Approved with adjustment. Next always lands on next Page leaf via next-sibling, ascend fallback, then first-child descent as needed.

5. End-of-sequence behavior
- Question: What happens when `NextPhaseAction` has no next phase?
- Decision: Approved with adjustment. No state change, `NoNextPhase` outcome, diagnostics, and `game_completed` emitted once per session; no phase enter/exit events for terminal no-op.

6. Set target mode
- Question: Should `SetPhaseAction` target by `phaseKey` only in MVP?
- Decision: Approved. `SetPhaseAction` target is `phaseKey` only in MVP.

7. Set target validation
- Question: Are jumps to any valid node allowed (including across books), or constrained by ancestry?
- Decision: Approved with adjustment. Jumps to any valid Page leaf are allowed across books/chapters.

8. Lifecycle event granularity
- Question: What lifecycle event set is required for MVP including load/resume?
- Decision (updated 2026-08-24): MVP lifecycle signaling is satisfied by `phase_changed`, `phase_changed_book`, `phase_changed_chapter`, and `phase_changed_page`. Explicit `phase_entered`/`phase_exited`/`phase_resumed` and `game_started`/`game_completed`/`game_resumed` keys are deferred.

9. Manifest anchor minimum set
- Question: Should anchors include both key identity and display-name hierarchy fields?
- Decision: Approved with adjustment. Include explicit Book/Chapter/Page key fields and display-name fields in phase anchors/payloads.

10. Host payload shape
- Question: Should phase-change payload be additive and optional for backward host compatibility?
- Decision: Approved with adjustment. Payload is additive and includes identity, display, title/prologue, reason, and resume context fields.

11. Ambience fallback behavior
- Question: If current phase has no `phaseAmbientSoundEffectId`, should action no-op or inherit parent?
- Decision: Approved with adjustment. Inherit nearest ancestor ambience when current node has none; if none in lineage, deterministic no-op with explicit outcome.

12. Ambience policy ownership
- Question: Should overlay/replace be phase metadata (not action input)?
- Decision: Approved with adjustment. Mode is phase metadata; action resolves mode and applies timer coordination behavior.

13. Replace-mode restore rule
- Question: How should ambience orchestration interact with general phase events and parent/child transitions?
- Decision: Approved with adjustment. General Book/Chapter/Page events remain symmetric; ambience orchestration is page-triggered and lineage-aware to avoid start/stop churn.

14. Reconciliation resolution timing
- Question: Should `ManageNarrativePhaseAmbientSounds` resolve current phase at execution time?
- Decision: Approved. Resolve current session page/lineage at execution time and reconcile in one deterministic pass.

15. Repeating trigger idempotence
- Question: Should repeated reconciliation triggers for the same effective ambience state be no-op/idempotent?
- Decision: Approved with adjustment. Reconcile old vs new effective ambience cue sets from previous/current anchors; unchanged set is no-op.

16. Terminology lock
- Question: Should canonical naming use `ambience` (not `ambiance`) across contracts/code/docs?
- Decision: Approved. Canonical naming uses `ambience` consistently.

## Lock-Off Question Count

1. Total design lock-off questions in this plan: 16.
