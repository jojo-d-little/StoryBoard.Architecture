# Sound Effects Library and ResultCode Binding Plan

Status: Complete (initial sound-effects pass closed; ready for archive)

## 0. Current Implementation Snapshot (2026-08-11)

Delivered in code:

1. Scope-owned sound-effect library model and persistence/export wiring are active in Designer and Shared contracts/runtime.
2. `soundEffectLane` ownership is locked on `SoundEffectLibraryEntry` (not on per-outcome cue rows).
3. `repeatMode` is schema-backed by dedicated enum contract (`SoundEffectRepeatMode`) and UI values are aligned to contract tokens.
4. Action outcome sound resolution now follows: specific result-code token first, then generic `Success`/`Failure` fallback.
5. Runtime host payload emits cue metadata and playback detail fields additively (lane, repeat, fade, volume, delay, etc.).
6. Simulator playback uses lane-aware lazy cache and now honors fade/repeat behavior for phase-1 expectations:
1. fade-in on first pass
2. fade-out on final pass for `RepeatCount`
3. best-effort for `RepeatForDuration` (no guaranteed final-pass fade-out)
7. Silent cue-drop behavior was reduced: unresolved/empty-asset cases now surface host diagnostics instead of disappearing silently.

Closeout summary:

1. Core phase-1 implementation and behavior tuning are complete across Designer and Simulator.
2. Regression gates for affected playback paths are green at closeout.
3. Deferred scope remains phase-2 event-system expansion (section 12, phase E).

## 1. Purpose

Deliver sound effects quickly and safely by reusing the existing ActionType + ResultCode authoring pattern already used by outcome echo messages.

This phase intentionally does not introduce a standalone runtime event system. It uses the existing deterministic command outcome surface to drive sound cues.

## 2. Scope and Order

1. Define a reusable sound-effect library model.
2. Add ActionType + ResultCode to SoundEffect selection mapping.
3. Keep playback detail editing in the sound library editor only.
4. Defer generalized event taxonomy to a future phase.

## 3. Non-Goals (Phase 1)

1. No full event bus or event-stream persistence model.
2. No host-specific audio engine implementation details in shared contracts.
3. No redesign of current presentation-cue behavior.
4. No breaking changes to existing runtime/export contracts.

## 4. Design Principles

1. Mirror a proven pattern: use the same authoring shape as per-result-code echo messages.
2. Separation of concerns: sound design details are authored in a library, not in action dialogs.
3. Deterministic runtime behavior: same action outcome always resolves the same sound selection rules.
4. Additive compatibility: new fields and mappings are additive and optional.
5. Shared ownership: trigger semantics and mapping resolution live in shared runtime/contracts.

## 5. Authoring Model

## 5.1 Two-Layer Authoring

Layer A: Sound Effect Library entry (authored once, reused many times).

Layer B: Action result binding (ActionType + ResultCode picks one library entry).

Outcome:

1. Sound designer controls playback behavior centrally.
2. Action result-code editor remains lightweight and focused on selection.
3. Producers avoid repetitive per-action playback tuning.

## 5.2 Sound Effect Library Shape (Initial Draft)

Each library entry should include:

1. SoundEffectId (stable producer-owned token).
2. DisplayName (authoring label).
3. Category (for browsing, example: Ambient, Foley, UI, Voice).
4. AssetRef (host-resolvable asset key, not machine file path).
5. BaseVolumeDb (optional).
6. RepeatMode (None, RepeatForDuration, RepeatCount).
7. RepeatCount (optional, used when RepeatMode=RepeatCount).
8. StartDelayMs (optional).
9. RepeatIntervalMs (optional).
10. MaxPlayDurationMs (optional per-play safety cap).
11. RepeatDurationMs (optional total repeat duration for RepeatForDuration mode).
12. RepeatCooldownMs (optional repeat-cycle retrigger suppression).
13. ConcurrencyGroup (optional polyphony control).
14. ConcurrencyGroupImportance (optional in-group ranking hint).
15. Importance (optional global drop-ranking hint).

Importance scale note:

1. Both `ConcurrencyGroupImportance` and `Importance` use the same integer scale (`-100` to `100`).
2. Higher values are more important.
3. Lower values are less important.
4. Equal values require deterministic host tie-break rules.

Notes:

1. Keep defaults explicit to avoid host-dependent behavior drift.
2. Keep the model additive so future controls can be introduced safely.

## 5.3 Action Result Binding Shape (Mirrors Echo Pattern)

Per binding entry:

1. ActionType.
2. ResultCode token.
3. SoundEffectId reference to library.
4. Optional override toggle (future): allow local volume scale only if truly needed.

Resolution rules:

1. Exact ActionType + ResultCode match first.
2. Optional baseline Success/Failure fallback for that ActionType.
3. No match means no sound cue.

This mirrors existing outcome message resolution semantics and keeps mental model consistent.

## 6. UX Direction

## 6.0 Phase UX-01 Checkpoint (Designer Sound Library Authoring)

Status: Lockoff Complete

Purpose:

1. Lock the designer-only authoring UX for scope-owned sound entries before implementing ActionType + ResultCode linkage UX.

Scope of this checkpoint:

1. Where and how sound entries are created and edited in scope editors.
2. Editor dialog shape and validation behavior.
3. Category management UX and filtering readiness.
4. Preview playback UX behavior.

Progress tracker:

1. Completed: core sound entry schema and scope-owned collection model.
2. Completed: first-pass sound editor dialog spec.
3. Completed: category chooser-first UX direction.
4. Completed: preview play or stop UX direction.
5. Completed: duplicate identity policy uses validation rule warnings (non-blocking; save allowed).
6. Completed: category casing policy preserves exact authored case.
7. Completed: category requirement policy locked (required; default `General` offered when chooser has no entries).
8. Completed: preview failure severity locked to non-blocking warnings.
9. Completed: repeat-field persistence rule locked (RepeatMode=None clears repeat values; non-none persists repeat values).
10. Completed: category chooser source locked to current-project in-memory entries.

Exit criteria:

1. All UX-01 lock-off questions below are resolved.
2. Implementation checklist is approved for StoryboardDesigner.App.
3. Minimum regression test set for editor behavior is accepted.

## 6.0.1 UX-01 Lock-Off Questions

Decision status key:

1. Open: needs explicit decision.
2. Locked: approved and implementation-ready.

| ID | Question | Options | Recommended | Status |
| --- | --- | --- | --- | --- |
| UX01-01 | Sound identity strategy and uniqueness scope for authoring validation? | A) Global uniqueness across full project for both immutable `SoundEffectId` (GUID) and readable `SoundEffectKey`; `DisplayName` is non-reference label; B) Uniqueness only within defining scope; C) Uniqueness within resolver-visible chain | A | Locked (2026-08-11) |
| UX01-02 | If duplicate IDs/keys are imported from legacy or external edits, what is editor behavior? | A) Block save until resolved; B) Validation-rule warning, allow save/continue, and apply deterministic resolver precedence | B | Locked (2026-08-11) |
| UX01-03 | Category canonicalization on write? | A) Preserve exact authored case; B) Canonicalize to first-seen case project-wide; C) Canonicalize to title case | A | Locked (2026-08-11) |
| UX01-04 | Category requirement and empty-input handling? | A) Category is required; prompt user when missing; normalize empty/whitespace to missing; if no existing categories are available offer default `General` choice; B) Category optional with null/empty support | A | Locked (2026-08-11) |
| UX01-05 | RepeatMode stale-field behavior on save? | A) If `RepeatMode=None`, clear repeat-related values. If `RepeatMode` is non-none, persist repeat-related values; clear inactive mode-specific field (`RepeatCount` vs `RepeatDurationMs`) deterministically. B) Preserve inactive values but ignore at runtime | A | Locked (2026-08-11) |
| UX01-06 | Preview failure severity in editor? | A) Non-blocking warning with inline status; B) Blocking validation error that prevents save | A | Locked (2026-08-11) |
| UX01-07 | Preview applies unsaved in-form values? | A) Yes, always; B) Preview only last-saved values | A | Locked (2026-08-11) |
| UX01-08 | Category list source for chooser options? | A) In-memory from current project entries only; B) Include global app-level recent categories | A | Locked (2026-08-11) |

Immediate lock sequence proposal:

1. UX01-01 and UX01-02 locked (ID integrity behavior).
2. UX01-05 locked (data persistence predictability).
3. UX01-03 and UX01-04 locked (category consistency).
4. UX01-06 and UX01-07 locked (preview behavior).
5. UX01-08 locked (category convenience scope).

## 6.0.2 UX-01 Implementation Checklist (Designer)

Status key:

1. Open: not started
2. In Progress: active implementation
3. Done: implemented and validated

Checklist:

1. Done: Add or update designer model support for `SoundEffectId` (GUID) + `SoundEffectKey` (readable key) with global uniqueness validation wiring.
2. Done: Add validation rule(s) for duplicate `SoundEffectId` and duplicate `SoundEffectKey` with warning severity and deterministic resolver guidance.
3. Done: Add scope-level UI entrypoint for "Owned Sound Effects" collection in scope editors.
4. Done: Implement `SoundEffect` editor dialog with locked field groups and mode-dependent enablement behavior.
5. Done: Implement category chooser-first control with quick-create and project in-memory category source.
6. Done: Enforce required category behavior with default `General` offered when chooser source is empty.
7. Done: Implement preview play/stop behavior using current unsaved form values.
8. Done: Surface preview failures as non-blocking inline warnings and diagnostics entries (inline non-modal warning/status behavior implemented; preview status exposes optional diagnostics-sink callback with normalized severity levels).
9. Done: Enforce save-time repeat-field persistence rule:
1. RepeatMode=None clears repeat-related values.
2. Non-none repeat modes persist repeat-related values.
3. Inactive mode-specific field is cleared.
10. Done: Add/refresh project save-load mapper coverage for `soundEffectLibraryEntries` and sound entry fields.
11. Done: Add dialog-level regression tests for mode toggles, category required behavior, preview behavior, and duplicate-warning behavior (including repeat normalization, dialog mode-state/category suggestion coverage, and category required/default-save behavior coverage).
12. Done: Run validation gates and document results in this plan section.

## 6.1 Sound Library Editor

Primary responsibilities:

1. Create, edit, duplicate, and retire reusable sound effects.
2. Author playback behavior (repeat mode, repeat count, delay, cooldown, in-group importance, global importance).
3. Preview-audition sounds (host capability permitting).

Preview UX direction (phase 1):

1. Provide a Play Preview action in the sound-effect editor that renders playback using current form values (including repeat settings and timing fields).
2. Provide a Stop Preview action to end long-running or repeated previews immediately.
3. Preview execution should be local editor-host behavior only and must not mutate authored project state.
4. Preview failures (missing asset, decode failure, unsupported format) should surface deterministic, actionable messages.
5. Preview should honor volume and timing settings so designers can validate authored behavior before saving.

## 6.1.1 Category Authoring and Selection UX

Goals:

1. Keep categories useful for large catalogs and picker filtering.
2. Avoid repetitive free-typing of category values.

Recommended behavior:

1. Category input is a chooser-first control backed by known categories.
2. Known categories are built from current project sound entries at load time and updated in-memory as entries are added/edited/deleted.
3. Editor supports quick-create when a new category is needed, then immediately adds it to chooser options.
4. Category values should be normalized for comparison/filtering (case-insensitive match) while preserving display casing.
5. Category is required for every sound entry; if no categories exist yet, chooser offers default `General`.

Picker integration expectations:

1. Action-result sound pickers should support category filtering using these known categories.
2. Category filter behavior must be deterministic across casing variants.

## 6.2 Action ResultCode Editor

Primary responsibilities:

1. Show supported result codes for selected ActionType.
2. Let user pick a SoundEffectId from the library.
3. Display compact defined or undefined status per code.

Explicitly out of scope in this editor:

1. Fine-grained playback timing controls.
2. Raw asset-path editing.
3. Host playback implementation settings.

## 6.5 Sound Effect Editor Dialog Spec (Phase 1)

Purpose:

1. Provide one focused editing surface for creating and tuning a `SoundEffectLibraryEntry`.
2. Keep this editor responsible for all playback-shape fields so action-result editors stay lightweight.

Suggested layout:

1. Header row:
1. `DisplayName` (required text)
2. `SoundEffectId` (required text, editable with uniqueness validation)

2. Asset row:
1. `AssetRef` picker or browse-assist control
2. Preview controls: `Play Preview`, `Stop Preview`

3. Classification row:
1. `Category` chooser-first combo with quick-create
2. If chooser has no existing values, present default `General` option

4. Playback behavior group:
1. `BaseVolumeDb`
2. `StartDelayMs`
3. `RepeatMode`
4. `RepeatCount` (shown or enabled only for `RepeatCount` mode)
5. `RepeatDurationMs` (shown or enabled only for `RepeatForDuration` mode)
6. `RepeatIntervalMs`
7. `MaxPlayDurationMs`
8. `RepeatCooldownMs`

5. Concurrency and ranking group:
1. `ConcurrencyGroup`
2. `ConcurrencyGroupImportance`
3. `Importance`

6. Footer:
1. Validation summary line (inline warnings or errors)
2. `Save`
3. `Cancel`

Dynamic field behavior rules:

1. If `RepeatMode=None`, disable `RepeatCount` and `RepeatDurationMs`.
2. If `RepeatMode=RepeatCount`, enable `RepeatCount`, disable `RepeatDurationMs`.
3. If `RepeatMode=RepeatForDuration`, enable `RepeatDurationMs`, disable `RepeatCount`.
4. Disabled mode-specific fields must not emit stale values on save (clear or ignore deterministically).
5. Preview uses current in-form values, including unsaved edits.

Validation rules (designer-facing):

1. `SoundEffectId` required and unique within resolver-visible scope set.
2. `DisplayName` required.
3. `AssetRef` required and non-empty.
4. `Category` required; empty or whitespace input is treated as missing.
5. Numeric fields respect schema ranges.
6. Mode-specific requiredness:
1. `RepeatCount` required when `RepeatMode=RepeatCount`.
2. `RepeatDurationMs` required when `RepeatMode=RepeatForDuration`.

Category UX details:

1. Category chooser suggestions are built from current project entries.
2. New category creation is available inline from the chooser.
3. Category matching for grouping and filtering is case-insensitive.
4. Display preserves original casing from first-created form unless user explicitly renames.

Preview UX details:

1. `Play Preview` disabled when required inputs are missing (`AssetRef`, `DisplayName`, `SoundEffectId`).
2. Preview errors show inline in dialog status and optionally diagnostics panel.
3. `Stop Preview` is available while preview is active.
4. Closing dialog always stops active preview.

Accessibility and keyboard behavior:

1. Tab order follows visual group order.
2. Numeric controls have labels with units (`ms`, `dB`).
3. Play and Stop actions are keyboard reachable.

Minimum regression tests for this dialog:

1. Mode switching toggles the correct fields and requiredness.
2. Save persists expected values and does not persist disabled mode-specific stale values.
3. Category chooser shows existing categories and supports quick-create.
4. Preview starts with in-form values and stops correctly.
5. Validation blocks save for missing required fields.

## 6.3 Scope-Based Authoring Location (Designer)

Recommended direction:

1. Sound-effect library entries are scope-owned and can be authored at any scope node.
2. Supported definition scopes in phase 1:
1. Global
2. Planet
3. Area
4. Room
5. GameObject
3. Most reusable soundscape entries are expected at Global, but lower-scope authoring remains allowed by design.
4. Scope ownership should be explicit in the authored data model so provenance is deterministic.

Authoring UX implications:

1. Scope editor surfaces an "Owned Sound Effects" collection similar to other scope-owned authoring assets.
2. Picker experiences should support inherited discovery (current scope plus ancestor scopes) while distinguishing where each sound is defined.
3. Name collisions across scopes must be handled deterministically (recommended: ID-based reference, display disambiguation by scope path).

## 6.4 Persistence Strategy for Project JSON

Question:

1. Should sound entries be persisted inline with their defining scope, or extracted into a separate project-level sound library folder/files similar to procedures?

Phase-1 recommendation:

1. Persist sound entries inline with the defining scope node in authored project JSON.
2. Do not introduce a separate sound-library file/folder set in phase 1.

Why this is preferred now:

1. Matches scope ownership semantics directly (definition lives where it is authored).
2. Lower implementation risk and less migration complexity for initial rollout.
3. Aligns with your requirement that object-local sounds remain true properties of an object when desired.

Tradeoffs and mitigations:

1. Tradeoff: global-level libraries can grow large inside project JSON.
Mitigation: add designer filtering/grouping and optional future extraction if scale demands it.

2. Tradeoff: cross-scope reuse lookup is required.
Mitigation: resolver/picker uses ancestor traversal with deterministic ordering.

3. Tradeoff: future external sound-pack workflows may prefer isolated files.
Mitigation: keep references ID-based so storage extraction is an additive later migration.

Future option (deferred):

1. Add an optional extracted sound-library artifact model if project scale or collaboration workflows justify it.
2. Keep backward compatibility by supporting both inline and extracted representations during transition, then converge intentionally.

## 7. Runtime and Boundary Placement

1. Shared contracts store library definitions and ActionType + ResultCode bindings.
2. Shared runtime resolves effective sound selection from action outcomes.
3. Host layer performs actual playback via host audio services.
4. No designer-simulator direct coupling.

## 7.1 Asset Export and Reference Rewrite Strategy

Sound assets must follow the same high-level pattern as runtime image export handling: copy to runtime-rooted output folders and rewrite references for runtime portability.

Locked phase-1 direction:

1. Designer-side `assetRef` is authoring-facing identity, not a machine-specific absolute file path.
2. Clean export copies referenced sound assets into a runtime-rooted audio folder (for example, `Audio/`).
3. Clean export emits runtime contracts with runtime-portable sound references (relative runtime path or runtime asset key), not raw authoring references.
4. Runtime host resolves playback from exported runtime references only; it must not depend on original authoring file locations.
5. Export behavior for missing assets is deterministic and diagnostic-rich (warning/error policy to be locked during implementation).

Current implementation gap to close before runtime host rollout:

1. Clean export currently copies image assets but does not yet emit a dedicated runtime audio assets folder with copied sound files.
2. Add an export slice that creates runtime audio asset output (for example, `Audio/`) and rewrites emitted runtime sound refs to that folder.

Implementation notes:

1. Keep the rewrite/copy behavior centralized in export mapping/services to avoid host drift.
2. Preserve deterministic output naming/location to reduce snapshot churn.
3. Prefer additive metadata fields when introducing explicit runtime reference semantics.

## 7.2 Asset Reference Semantics (Authoring vs Runtime)

1. Authoring contracts keep producer-managed references suitable for editing workflows.
2. Runtime contracts carry resolved runtime-playback references suitable for packaged/runtime execution.
3. If both forms must coexist temporarily, name them explicitly and document precedence.

## 7.3 Runtime Sound Host Interface Lock Phase (New)

Status: Implemented (phase-1 shape active; refinement items remain)

Purpose:

1. Lock the host-facing runtime sound cue contract before simulator playback integration.
2. Keep shared runtime transport-only (no playback in shared) and host-owned playback execution.
3. Keep phase-1 timing semantics strictly action-result immediate to avoid introducing event-model blur.

Locked constraints for this phase:

1. Shared runtime emits sound cue intent only; shared runtime does not play audio.
2. Cue emission source in phase 1 is action-result processing only.
3. Timing intent in phase 1 is immediate only.
4. Do not add or imply event-timing semantics (for example after room transition, after animation) in phase 1 contracts.

### 7.3.1 Design Lock-Off Questions

Decision status key:

1. Open: needs explicit decision.
2. Locked: approved and implementation-ready.

| ID | Question | Options | Recommended | Status |
| --- | --- | --- | --- | --- |
| RSHI-01 | Emission boundary for phase 1? | A) Internal emission source stays action-result only, but host payload remains source-agnostic as a command-response sound-cue list; B) Add runtime event-hook emission in phase 1 | A | Locked (2026-08-11) |
| RSHI-02 | Cue identity fields required in host payload? | A) command correlation + action id + result code + ordered index; B) minimal id only | A | Locked (2026-08-11) |
| RSHI-03 | Runtime sound lookup fields in host payload? | A) `soundEffectId` + `soundEffectKey` + runtime `assetRef`; B) `assetRef` only | A | Locked (2026-08-11) |
| RSHI-04 | Playback detail surface for phase 1? | A) include repeat/fade/volume details from authored sound entry; B) key-only and host-side defaults | A | Locked (2026-08-11) |
| RSHI-05 | Multi-cue behavior in one action result? | A) preserve authored order and emit all; B) first cue only | A | Locked (2026-08-11) |
| RSHI-06 | Host conflict policy in phase 1 contract? | A) include explicit policy fields (replace/layer/stop); B) defer to host defaults with no contract field | B | Locked (2026-08-11) |
| RSHI-07 | Missing runtime sound asset behavior at playback time? | A) host warning diagnostic + continue; B) fail command; C) silent skip | A | Locked (2026-08-11) |
| RSHI-08 | Host payload location for emitted cues? | A) dedicated top-level sound cue list on process results; B) only embed under room object changes | A | Locked (2026-08-11) |
| RSHI-09 | `GetCurrentPresentation` support for sound cues? | A) include currently relevant immediate cues only when present in current payload surface; B) omit sound cues from presentation refresh | B | Locked (2026-08-11) |
| RSHI-10 | Versioning strategy for this contract slice? | A) additive optional fields under existing result contracts; B) new parallel result contract type | A | Locked (2026-08-11) |

### 7.3.2 Lock Sequence

1. RSHI-01 boundary and immediate-only confirmation.
2. RSHI-08 payload location and RSHI-10 versioning shape.
3. RSHI-02 identity fields and RSHI-03 lookup fields.
4. RSHI-04 playback detail surface and RSHI-05 multi-cue behavior.
5. RSHI-06 conflict policy and RSHI-07 missing-asset behavior.
6. RSHI-09 `GetCurrentPresentation` inclusion decision.

### 7.3.3 Exit Criteria

1. All RSHI questions are locked with explicit decisions.
2. Contract draft targets and file touch-list are approved.
3. Runtime + simulator implementation slices are sequenced after lock-off.

RSHI-01 lock notes:

1. Host interface should not need to know whether cues originated from action-result logic or any future source.
2. Host receives a command-response payload containing sound cues to play with required playback detail.
3. Phase-1 runtime behavior still restricts emitted cues to action-result processing only.

RSHI-08 lock notes:

1. Host payload shape should include a dedicated top-level `soundCues` list on command responses.
2. Sound cue delivery is not coupled to room-object change payload semantics.
3. Keep optional future parity consideration for current-presentation surfaces as a separate lock question.

RSHI-10 lock notes:

1. Add sound cue payload fields additively on existing host process-result contracts.
2. Preserve backward compatibility for hosts that do not yet consume `soundCues`.
3. Avoid parallel result-contract forks in phase 1.

RSHI-02 lock notes:

1. Include cue correlation metadata: `commandCorrelationId`, `actionId`, `resultCode`, and `sequenceIndex`.
2. Treat these fields as host correlation and diagnostics metadata, not host-facing source-provenance semantics.
3. Hosts may ignore any field not needed for playback behavior while still retaining deterministic ordering.

RSHI-03 lock notes:

1. Emit `soundEffectId` for stable cue identity and deterministic correlation.
2. Emit `soundEffectKey` for host diagnostics/readability.
3. Emit runtime `assetRef` as the concrete host lookup/playback reference.

RSHI-04 lock notes:

1. Emit authored playback details in host cue payload (for example repeat mode, repeat count/duration, volume, fade-in/fade-out, delay, cooldown when present).
2. Preserve producer-authored behavior intent as data, not as hidden host defaults.
3. Keep host implementation ownership of actual playback engine behavior.

RSHI-05 lock notes:

1. Emit all authored cues for the matched result code.
2. Preserve authored cue order in emitted payload.
3. Keep host playback scheduling/execution decisions separate from cue emission ordering.

RSHI-06 lock notes:

1. Do not add explicit replace/layer/stop conflict-policy fields in phase-1 sound cue contracts.
2. Host applies local defaults for conflict handling.
3. Revisit explicit policy fields only after phase-1 integration and telemetry feedback.

RSHI-07 lock notes:

1. Missing runtime sound assets should not fail command execution.
2. Host emits deterministic warning diagnostics and continues processing.
3. Avoid silent skips to preserve debuggability for producers.

RSHI-09 lock notes:

1. Phase-1 sound cues are immediate command-response payload data only.
2. `GetCurrentPresentation` remains focused on visual/state hydration and does not replay sound intent.
3. Prevent accidental re-fire/replay behavior during presentation refresh.

### 7.3.4 Host Interface Draft (Review Before Implementation)

Status: Proposed for review

This section translates locked RSHI decisions into a concrete interface proposal.

Phase-1 interface intent:

1. Additive-only updates to existing process-result contracts.
2. Immediate command-response sound cue payload only.
3. No sound cue payload on `GetCurrentPresentation` in phase 1.

Proposed host-facing additions:

1. Add `SoundCues` to host command result contract (`IHostProcessCommandResult`).
2. Add matching `SoundCues` to shared processing result contract (`IGameCommandProcessingResult`).
3. Add dedicated sound cue interfaces and concrete models.

Proposed contract members (draft C# shape):

```csharp
public interface IHostProcessCommandResult
{
	// Existing members...
	IReadOnlyList<IHostSoundCue> SoundCues { get; }
}

public interface IGameCommandProcessingResult
{
	// Existing members...
	IReadOnlyList<IGameCommandSoundCue> SoundCues { get; }
}

public interface IHostSoundCue
{
	Guid SoundEffectId { get; }
	string SoundEffectKey { get; }
	string RuntimeAssetRef { get; }

	int CommandCorrelationId { get; }
	Guid ActionId { get; }
	string ResultCode { get; }
	int SequenceIndex { get; }

	string RepeatMode { get; }
	int? RepeatCount { get; }
	int? RepeatDurationMs { get; }
	int? RepeatIntervalMs { get; }
	int? StartDelayMs { get; }
	int? MaxPlayDurationMs { get; }
	int? RepeatCooldownMs { get; }

	double? BaseVolumeDb { get; }
	int? FadeInMs { get; }
	int? FadeOutMs { get; }
}
```

Notes on this draft shape:

1. `ActionId` and `CommandCorrelationId` are correlation metadata; host playback does not need to infer source semantics.
2. `SequenceIndex` preserves authored order for multi-cue emission.
3. `RuntimeAssetRef` is the host lookup/playback source reference from clean export rewrite output.
4. Conflict-policy fields are intentionally excluded from phase 1.

Emission semantics:

1. Runtime emits zero or more sound cues on each processed command result.
2. Runtime emits all matched cues in authored order.
3. Missing runtime asset resolution in host should produce warning diagnostics and continue.

Compatibility and rollout:

1. New `SoundCues` members are additive; older hosts can ignore them.
2. No parallel result-contract fork is introduced.
3. Existing visual presentation payload contracts remain unchanged.

Pending naming/type review points before implementation:

1. Should `RepeatMode` be `string` or a shared enum type in host contracts.
2. Should `CommandCorrelationId` and `ActionId` be nullable for defensive compatibility.
3. Whether to include optional `CueDiagnostics` text field in each cue now or defer.
4. Final model type names (`IHostSoundCue`/`IGameCommandSoundCue`) before codegen or manual interfaces are introduced.

## 8. Validation and Testing Strategy

Minimum regression coverage:

1. Binding resolution: exact match, baseline fallback, no-match silent behavior.
2. Token stability: supported result-code token matching remains case-neutral and deterministic.
3. Library reference integrity: missing SoundEffectId produces deterministic warning, not crash.
4. Deterministic playback directives: identical input outcomes produce identical cue directives.
5. Serialization roundtrip: library entries and bindings persist/load without loss.
6. Export copy/rewrite parity: sound assets are copied to runtime output and emitted runtime references point to copied locations.
7. Missing source asset handling: export emits stable diagnostics and does not crash.
8. Deterministic export paths: repeated exports produce stable runtime audio reference paths for unchanged inputs.

## 9. Risks and Mitigations

1. Risk: too many per-action bindings to manage.
Mitigation: baseline Success/Failure fallback and reusable library entries.

2. Risk: library bloat and duplicates.
Mitigation: stable IDs, categories, and optional usage reporting in editor.

3. Risk: phase-1 model insufficient for ambient or timer-driven cues.
Mitigation: keep resolver seam generic so future event-trigger inputs can be added.

4. Risk: authoring reference and runtime reference drift during export.
Mitigation: single export rewrite pipeline with focused contract + snapshot regression tests.

## 10. Future Event-System Follow-Up (Deferred)

When needed for richer scenarios (ambient zones, timers, non-command world activity):

1. Introduce event-trigger source alongside ActionResult trigger source.
2. Reuse the same SoundEffect library and cue resolution pipeline.
3. Allow TriggerKind union: ActionResult or RuntimeEvent.

This keeps phase-1 investment valid and avoids rewrite churn.

## 11. Immediate Decisions to Lock

1. Final list of phase-1 sound library fields (minimum viable set).
2. Whether ActionType-level Success/Failure fallback is enabled at launch.
3. Missing reference behavior: warning-only vs validation error.
4. Whether result-code sound selection is single-select or multi-select in phase 1.
5. Naming convention for SoundEffectId tokens.
6. Runtime sound reference representation: relative path vs runtime asset key token.
7. Missing-asset export policy: warning + skip, warning + placeholder, or hard error.

## 12. Suggested Implementation Sequence

1. Phase A: contract draft for SoundEffect library + ActionResult binding. Status: Done.
2. Phase B: shared resolver implementation and diagnostics. Status: Done (with specific->generic fallback and unresolved-cue diagnostics hardening).
3. Phase C: designer sound library editor and Action ResultCode picker integration. Status: Done (phase-1 behavior active).
4. Phase D: simulator host playback adapter and smoke verification. Status: Done (including lane-aware cache and repeat/fade behavior updates).
5. Phase E: evaluate adding RuntimeEvent trigger source using same library. Status: Deferred.
