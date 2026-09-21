# Simulator Retirement Readiness Plan

Last updated: 2026-09-20
Status: Draft / decision and implementation planning
Owner: Cross-repository architecture workstream

Purpose: close the development-workflow, diagnostics, presentation-control, recording/playback, and runtime-inspection gaps that would otherwise be lost when the WPF simulator is retired, while making WebPortal/Pixi the sole runtime presentation implementation.

Stage baseline source:
1. Use `plans/STANDARD_STAGE_CATALOG.md` for the default stage IDs and ordering.
2. Use `SIMULATOR_RETIREMENT_READINESS_EXECUTION_ROADMAP.md` as the implementation dispatch index. It selects vertical feature passes and their area-profile ownership.
3. The numbered area profiles define ownership, boundaries, compatibility, and validation; feature-area pass handoffs hold execution evidence and are the only documents implementation sessions update.
4. This workstream intentionally uses Stage 05 as a replacement-parity audit and simulator freeze stage, not as authorization to add new WPF rendering behavior.
5. Stage 01 remains additive and backward-compatible whenever a feature pass requires contract work.
6. Stage 06 remains globally gated and is initially skipped because this is a retirement-readiness plan, not the final simulator deletion plan.

## Problem Statement

The system currently maintains three room-oriented visual surfaces:

1. Designer authoring preview, which is required for authoring and layout.
2. WPF simulator runtime presentation, which combines gameplay, diagnostics, replay, runtime inspection, and a second room renderer.
3. WebPortal/Pixi runtime presentation, which is the actual player-facing rendering surface.

The WPF simulator's unique development capabilities are valuable, but its independent room renderer creates recurring parity and troubleshooting work. The strategic direction is to retire the WPF simulator and use WebPortal/Pixi as the sole runtime presentation path. Retirement is not allowed until the replacement development experience satisfies explicit readiness gates.

The highest-priority gap is ad hoc designer launch. A newly created or limited-development project must be runnable without adding it to the production/published game catalog. The developer must be able to save/export in Designer, launch an isolated GameHost against that exact runtime export, and open WebPortal already attached to the intended development game/session.

Additional gaps include presentation-family isolation, richer diagnostics, durable diagnostic export, recording/playback, save/load state, and debugger/runtime-state capabilities. These must be implemented, explicitly waived, or moved to a documented follow-up before simulator retirement can be approved.

## Non-Goals

1. Do not remove the WPF simulator in this workstream.
2. Do not add ad hoc or unfinished projects to the durable production game catalog.
3. Do not make the browser parse Designer source files or runtime-export files directly.
4. Do not reproduce WPF layout, controls, or rendering mechanics in WebPortal.
5. Do not move runtime authority, command parsing, session state, or replay execution into Pixi.
6. Do not expose development-only runtime mutation, file access, or replay controls from a production GameHost configuration.
7. Do not require pixel parity between Designer authoring preview and WebPortal runtime rendering.
8. Do not add new WPF visual features while retirement readiness is being established, except approved blocking fixes needed to validate the replacement.

## Current Baseline (Observed)

### Runtime presentation and WebPortal tooling

1. WebPortal/Pixi already renders room imagery, room objects, movement, transitions, appearance effects, waypoints, HUD/presentation output, and audio from host-driven state.
2. WebPortal already owns host authentication, discovery, session start/join/leave/end, command submission, clarification flow, baseline hydration, and delta polling.
3. WebPortal already provides development tooling for polling cadence, diagnostic category filters, diagnostic verbosity, host URL override, layout/theme inspection, browser asset-cache controls, audio mute/volume, and room-transition cue selection.
4. WebPortal diagnostics can already be filtered by severity, copied to the clipboard, and downloaded as a timestamped text file.
5. WebPortal diagnostic retention is an in-memory client ring limited to a configured range, currently capped at 500 entries.
6. WebPortal command requests currently request `Low` engine diagnostics and delta polling currently requests `Medium` diagnostics; these levels are hard-coded rather than controlled by the development UI.
7. Delta polling counts `HostSessionDataEnvelope.Diagnostics`, but no general path was found that promotes every returned session diagnostic into the WebPortal diagnostic console.

### Ad hoc game launch and registration

1. Designer currently saves, validates, exports, and starts `Storyboard.Simulator.exe` with `--project <runtime-export-path>`.
2. GameHost starts sessions by resolving `GameId` or `GameKey` through a runtime registration catalog.
3. Discovery, session startup, and asset management all depend on registration identity and runtime-project resolution.
4. GameHost currently supports an alternate catalog path through `STORYBOARD_RUNTIME_GAME_DISCOVERY_JSON_PATH`; GameHost socket smoke tests already use this seam.
5. Registration reading is duplicated across GameManager/session startup, discovery, and asset management. A development launch must not allow these consumers to disagree about the active registration source.
6. Runtime export contains display metadata but does not provide the host-assigned registration `GameId` required by the current catalog shape.
7. GameHost can already serve a built WebPortal bundle under a configured `/client` mount.

### WPF simulator capabilities not fully replaced

1. Global, movement, appearance, and disappearance effect toggles.
2. Selectable runtime diagnostics levels: None, Low, Medium, and High.
3. A combined console containing command output, runtime diagnostics, poll traces, sound/effect traces, and preview/image diagnostics.
4. Console archive/review and selected-line copying.
5. Command recording to `.sbe.sim.json`, including command text, correlation ID, clarification answers, timing delta, output, diagnostics, move-leg telemetry, and success.
6. Recording load, timed playback, playback speed, single-step playback, stop/shift-to-manual, and continuation recording.
7. Save-game capture and file export plus save-game file import and session restore.
8. Thick-mode runtime state tree, variable mutation, and move-player debugging.
9. Reinitialize from the current runtime export.
10. Local recent-project and duplicate-launch/restart behavior.

### Existing ownership issue in recording/playback

1. The simulator recording file and step types are private nested types inside `SimulatorViewModel`.
2. Recording occurs in the client after command execution rather than at an authoritative host/session boundary.
3. Playback timing and command submission are driven by the WPF UI process.
4. Existing regression fixtures depend on the `.sbe.sim.json` semantic shape and must not be discarded during ownership migration.

## Architectural Decisions And Decision Gates

### Locked direction

1. WebPortal/Pixi becomes the sole runtime presentation implementation after readiness approval.
2. Designer authoring preview remains; it is not a second runtime renderer and is not subject to Pixi pixel parity.
3. The durable production game catalog remains reserved for deliberately registered games.
4. Ad hoc Designer projects use an ephemeral development registration and must not mutate the production catalog.
5. Runtime project files remain server-side inputs. WebPortal receives host contracts and asset URLs/bytes, not direct filesystem paths.
6. WPF simulator retirement is gated by the parity matrix and acceptance gates in this plan.

### Locked F01 launch-configuration direction

Use the established `runtime-game-registrations.json` format; do not create a new development-launch manifest, command-line schema, or `Storyboard.Contracts` contract for F01.

For each Designer development launch:

1. Designer creates an invocation-scoped temporary `runtime-game-registrations.json` containing one `games` entry for the current exported project.
2. The entry uses the agreed deterministic development `GameId`/`GameKey`, current runtime export path, `isEnabled`, and the established tenant/organization fields.
3. Designer starts the child GameHost with `STORYBOARD_RUNTIME_GAME_DISCOVERY_JSON_PATH` set to that temporary file.
4. The temporary file remains available for the active host lifetime and bounded diagnostic retention, then is cleaned up.
5. The default catalog is never edited and no production game registration is created.

Rationale:

1. The existing registration JSON already supplies the required game identity and runtime-project mapping.
2. GameHost discovery, `GameManager`, and runtime asset management already honor `STORYBOARD_RUNTIME_GAME_DISCOVERY_JSON_PATH`; F01 must prove the same generated one-entry file is used consistently by all three paths.
3. Startup/browser preferences remain Designer-local process-launch settings, not fields added to a new cross-project manifest schema.

### Recommended recording/playback decision (approval required before the F04 contract pass)

Move authoritative recording and playback coordination to GameHost/GameEngine session services, with WebPortal acting as the control and monitoring client.

1. Record accepted command requests at the authoritative session boundary.
2. Preserve raw command text, correlation/idempotency identity, clarification answers, inter-command timing, outcomes, diagnostics, and movement telemetry needed by existing fixtures.
3. Store recordings in a host-owned development recording store, scoped by game and session.
4. Expose capability-gated operations to list, inspect, start, stop, export, import, play, pause, step, change speed, and branch/continue recordings.
5. Playback must submit through the same authoritative command path as live input.
6. WebPortal must not depend on arbitrary browser filesystem access. Browser upload/download may supplement, but not replace, the host-owned recording store.
7. Existing `.sbe.sim.json` files remain importable through an explicit compatibility reader until their migration/retirement is separately approved.

Implementation gate:

1. Before any F04 contract, host, engine, or Portal implementation begins, produce an implementation-level recording/playback interface specification and obtain explicit architecture approval.
2. After approval, define that interface first in `Storyboard.Contracts` as a versioned, backward-compatible canonical contract. Contract/schema generation and compatibility validation complete before GameHost, GameEngine, Designer, or WebPortal implementation begins; consumers must not create parallel local interface shapes.
3. The specification and contract must cover project-scoped library discovery, recording lifecycle, project save/promotion, import/export, timed playback, interactive stepped playback, state/version-token concurrency, errors, authorization, compatibility, and success/failure/cancellation scenarios.
4. Manual playback must let WebPortal preview the authoritative next command and explicitly request each advance; GameHost validates and executes that step through the normal command path.

### Locked diagnostics workflow direction

Diagnostics is a first-class development workflow inside Dev Tools, not a configuration exercise assembled indirectly from layout-slot controls.

1. Dev Tools must expose a top-level `Diagnostics` workspace that is reachable in one action whenever development tools are available.
2. Its primary actions must remain visible at the top of that workspace:
   - `Start Trace` / `Stop Trace`
   - `Show Console` / `Hide Console`
   - `Save Diagnostics`
   - `Clear`
3. Starting or stopping trace capture, showing or hiding the console, and docking or undocking the console are independent states. For example, hiding the console must not stop capture, and stopping capture must not erase the current entries.
4. The current capture state must be unmistakable without opening a secondary panel: `Off`, `Capturing`, `Paused/Stopped`, or `Truncated` as applicable, with captured-entry count and active profile visible.
5. The workspace must provide a concise trace profile selector plus an expandable advanced scope editor:
   - Profile examples: `Off`, `Focused`, `Normal`, `Verbose`, and `Custom`.
   - Scope groups: Portal command/clarification UI, received session/delta/transport activity, renderer, presentation/effects, audio, assets/cache, and client/orchestration.
   - Every scope choice controls Portal-side capture and display only; it does not change backend diagnostic behavior or aggregate backend logs.
6. `Show Console` must directly reveal the diagnostics console in its last docked/undocked location. The user must not edit layout-slot modes merely to make it visible.
7. `Save Diagnostics` must be available directly from the Diagnostics workspace whether or not the console is visible or docked.
8. Saving defaults to the complete captured buffer, not merely the currently filtered view. If saving visible/filtered entries is offered, it must be a separately labeled action.
9. Saved output must state the active trace profile, source/category filters, capture start/end times, entry count, dropped/truncated count, Portal build identity, observed launch/session identifiers where available, and whether the export is complete or filtered.
10. Common diagnostics actions should not require closing Dev Tools first. Console show/hide and capture controls must work while the Dev Tools surface remains open.
11. Development settings may persist locally, but every new capture must have an explicit boundary and must not silently append entries from a prior launch/session without identifying that boundary.
12. Production-facing configurations continue to hide this development workspace unless the host advertises the required development capability.

Recommended interaction target:

1. Open Dev Tools.
2. Select `Diagnostics` from a permanent first-level tab/navigation item.
3. Choose a profile or retain the last development profile.
4. Press `Start Trace`.
5. Press `Show Console` if live output is desired.
6. Reproduce the issue.
7. Press `Stop Trace` and `Save Diagnostics` without navigating elsewhere.

This seven-action sequence is the maximum normal path. With the prior profile retained, trace/show/reproduce/stop/save should require no secondary configuration panels.

### Remaining implementation-level policy for approved contracts

1. The F04 Recording And Playback Interface Specification defines project-library naming, atomic write behavior, concurrent writer behavior, optional scratch-recording retention, and cleanup. Project-owned canonical recordings have no automatic retention cleanup.

### Locked replay-assertion boundary

1. Expected-versus-actual assertion evaluation is owned by automated regression playback and its test harnesses.
2. A P0 WebPortal manual or timed playback run displays ordinary execution progress, result, failure, and diagnostics, but does not evaluate or present recording assertion mismatches.
3. The recording model preserves the data required by existing regression fixtures and their authoritative assertions.
4. Interactive assertion evaluation may be added later only through an approved follow-up; it is not a simulator-retirement readiness requirement.

### Locked runtime-inspection boundary

1. The P0 replacement is a development-capability-gated WebPortal request that causes GameEngine to write a complete, read-only, flattened semantic game-scope dump into the Engine flow trace. The Portal receives completion status and dump identity, not the tree payload.
2. Inspection is explicitly manual: a developer requests a dump, inspects that immutable dump in the Engine trace, performs game activity, and requests another dump when needed. No continuous live subscription is required.
3. The dump represents the complete semantic engine game-scope tree, not an unrestricted CLR process-memory dump. Its coverage must be mapped against the simulator's existing game-scope tree so no represented scope branch is silently omitted.
4. The request/acknowledgement contract and dump-event format are defined, reviewed, and approved first in `Storyboard.Contracts` before GameEngine, GameHost, or WebPortal implementation begins. They must declare dump identity/time, session identity, completeness/truncation status, stable semantic property path, node/value kind, typed/null values, collection/reference/cycle behavior, deterministic ordering, summary counts, payload limits, and errors.
5. P1 may retrieve the same approved snapshot into WebPortal and render an interactive tree, but that browser viewer is not required for retirement readiness.
6. Runtime mutation is deferred from this readiness workstream. No mutation endpoint or Portal mutation UI is authorized without a separate approved contract and capability design.

### Locked debug-teleport boundary

1. Move-player/debug teleport is explicitly deferred from simulator-retirement readiness and is not a retirement blocker.
2. The simulator audit must record its current use, tests, and any discovered dependency so the capability is not silently lost.
3. No GameHost/GameEngine endpoint or WebPortal control is authorized in this workstream. A later implementation requires a separate approved contract, destination semantics, authorization/audit model, and session-resynchronization design.

### Locked WebPortal diagnostic-capture boundary

1. The WebPortal Diagnostics workspace controls and captures WebPortal client trace only.
2. `Save Diagnostics` exports that complete client-side capture to a browser download; it does not collect, request, correlate, or package GameHost/GameEngine logs.
3. Backend logs remain independently owned and manually correlated by developers when needed.
4. GameEngine, GameHost, and WebPortal trace events use an approved, versioned correlation envelope defined first in `Storyboard.Contracts`. The shared envelope enables manual alignment now and a future trace-alignment utility, but does not authorize a shared log store or WebPortal log aggregation.

## Parity And Disposition Matrix

Priority definitions:

1. `P0`: retirement blocker.
2. `P1`: expected development parity; may be waived only by an explicit architecture decision with a named follow-up.
3. `P2`: convenience parity; may follow retirement if the replacement workflow remains usable.

| Capability | Current owner/state | Target owner | Priority | Gap / required outcome |
| --- | --- | --- | --- | --- |
| Designer save/validate/export launch | Designer launches WPF simulator | Designer + development launcher | P0 | Add the one-action GameHost/WebPortal workflow alongside the legacy Simulator launch through F01-F07; preserve non-blocking failures and retire the legacy launch only in post-Go F08. |
| Ad hoc unregistered project | WPF loads runtime path directly | GameHost development registration overlay | P0 | Start exact runtime export without durable catalog mutation. |
| Consistent game identity for sessions/assets | Durable catalog entry | Shared injected development registration source | P0 | Discovery, session startup, and asset APIs must resolve the same synthetic identity. |
| Automatic browser bootstrap | Manual normal WebPortal flow | GameHost/WebPortal development bootstrap | P0 | Auto-connect, authenticate development principal, select game, start/reset session, and attach. |
| Host lifecycle and duplicate launch | WPF mutex/restart arbitration | Development launcher | P0 | Deterministic reuse/restart/isolation, readiness wait, port allocation, and orphan cleanup. |
| Production isolation | No ad hoc host mode | GameHost capability/security boundary | P0 | Development registration and control endpoints disabled outside explicit development launch. |
| Version-matched client | Separate host/static configuration | GameHost + packaged WebPortal | P0 | Detect host/contract/client mismatch; packaged bundle is reliable default. |
| Runtime room rendering | Implemented in both WPF and Pixi | WebPortal/Pixi only | P0 | Required functional/visual WebPortal regression set passes before cutover. |
| Global visual-effects toggle | WPF only | WebPortal renderer dev controls | P1 | Disable all renderer-local optional effects without changing runtime state. |
| Movement-effects toggle | WPF only | WebPortal renderer dev controls | P1 | Suppress movement visualization while authoritative movement still occurs. |
| Appearance/disappearance toggles | WPF only | WebPortal renderer dev controls | P1 | Independently suppress effect families with lifecycle-safe cleanup. |
| Sound isolation | WebPortal already has SFX/ambient mute and volume | WebPortal dev controls | P1 / mostly present | Verify all runtime sound lanes obey controls; add one-click all-audio suppression if needed. |
| Room-transition isolation | WebPortal can select transition cue | WebPortal dev controls | P1 / partial | Add explicit disable/bypass control, not only cue selection. |
| HUD/presentation-cue isolation | No equivalent family switch confirmed | WebPortal dev controls | P1 | Allow focused troubleshooting without dropping authoritative cue diagnostics. |
| Engine diagnostics level | WPF selectable; Web commands/polls hard-coded | Existing backend diagnostics workflow | Separate concern | Not a WebPortal trace-control or retirement-readiness requirement; diagnose backend behavior through its own tools. |
| Runtime diagnostic ingestion | WPF merges command/session diagnostics | Existing backend diagnostics workflow | Separate concern | Do not expand WebPortal into a backend-log aggregation client for this workstream. |
| Renderer/client diagnostics | WebPortal already emits structured categories | WebPortal | Present | Retain category, severity, details, and correlation metadata. |
| Cross-layer correlation metadata | Partial request/session IDs | Shared trace envelope + manual developer workflow | P1 | Define common, versioned identifiers so separate engine, host, and Portal trace files can be aligned manually; do not implement automated correlation or aggregation in this workstream. |
| First-class diagnostics workspace | Dev Tools has a diagnostics section, but console visibility/docking is controlled indirectly through layout/slot configuration | WebPortal Dev Tools | P0 | Permanent top-level Diagnostics workspace with direct capture, console, save, clear, profile, and status controls. |
| One-click trace capture | Diagnostics enabled/category/verbosity settings are separate and capture boundaries are not explicit | WebPortal Dev Tools | P0 | Start/stop creates an explicit client-trace capture boundary without clearing prior data unexpectedly. |
| Direct console visibility | Requires indirect slot/panel manipulation in some layouts | WebPortal shell + Dev Tools | P0 | Show/hide console from Diagnostics workspace while preserving dock state and capture state. |
| Trace-scope clarity | Client source categories and display filters are separate concepts | WebPortal Dev Tools | P0 | Profiles and advanced scope editor clearly distinguish Portal capture from display filters. |
| Diagnostic retention | WPF console + archive; WebPortal capped in memory | WebPortal | P1 | Configurable larger client-side retention and no silent loss during long sessions. |
| Diagnostic export | WebPortal copy/download exists inside the console | WebPortal | P0 / partial | Direct save from Diagnostics workspace even when console is hidden; export all vs filtered entries, structured JSON plus readable text, environment/session metadata, and truncation disclosure. |
| Host/runtime log collection | Separate host/runtime files | Existing backend diagnostics workflow | Separate concern | Explicitly excluded from WebPortal Diagnostics capture/export; developers correlate manually when needed. |
| Recording start/stop | WPF client-private | GameHost/GameEngine recording service | P0 | Authoritative per-session recording with status and failure diagnostics. |
| Recording list/select | WPF file dialog | Host recording store + WebPortal | P0 | List recordings for current development game/session and select without arbitrary browser disk access. |
| Recording import/export | WPF direct filesystem | Host API plus browser upload/download | P1 | Preserve `.sbe.sim.json` compatibility and portable fixtures. |
| Timed playback and speed | WPF client-driven | GameHost/GameEngine playback coordinator | P0 | Deterministic timing, speed, pause/stop, status, and cancellation. |
| Step playback | WPF | Host playback coordinator + WebPortal | P0 | Execute exactly one next recorded command and update the same session presentation path. |
| Continue/branch recording | WPF | Host recording service + WebPortal | P1 | Preserve consumed prefix and timestamp rebasing or explicitly replace with a better branch model. |
| Replay regression fixtures | Simulator tests | Shared runtime and WebPortal integration tests | P0 | Port semantic fixtures before deleting simulator-owned tests. |
| Save game capture/download | Host contract exists; WPF writes file | WebPortal + existing host persistence API | P1 | Capture authoritative envelope and download through browser. |
| Load game upload/restore | Host contract exists; WPF reads file | WebPortal + existing host persistence API | P1 | Upload/validate envelope, restore target session, resync baseline. |
| Reinitialize latest export | WPF direct reload | Development launcher/GameHost | P0 | Restart/reset against the newly exported runtime deterministically. |
| Runtime state tree | WPF thick mode only | WebPortal request + GameEngine flattened flow-trace dump | P0 | On manual request, write the complete authoritative semantic game-scope tree to Engine flow trace with clear dump identity and complete/truncated status; Portal reports request completion. |
| Interactive runtime tree viewer | WPF thick mode only | Optional WebPortal follow-up | P1 | Retrieve and render the approved scope snapshot only after the P0 dump path is proven; it is not required for retirement readiness. |
| Runtime variable mutation | WPF thick mode only | Separate future capability | Deferred by decision | Explicitly deferred; no mutation interface or UI is part of readiness work. |
| Move-player/teleport | WPF debug action | Separate future capability | Deferred by decision | Audit and record actual use, but do not implement as simulator-retirement readiness work. |
| Session discovery/management | Present in both | WebPortal | Present | Validate development bootstrap does not regress normal production discovery. |
| Browser asset-cache inspection | WebPortal present | WebPortal | Present | Validate ad hoc registration identity and paths work with cache keys. |
| Recent projects | WPF | Designer owns project history | P2 | Do not recreate in WebPortal unless an actual gap remains. |

## Proposed Shape

### A. Development launch plane

1. Designer remains responsible for save, validation, and runtime export.
2. A small development-launch service owns GameHost process creation, readiness, reuse/restart policy, and browser opening.
3. Designer creates an invocation-scoped, one-entry `runtime-game-registrations.json` in the established format and starts the child GameHost with `STORYBOARD_RUNTIME_GAME_DISCOVERY_JSON_PATH` pointing to it.
4. The existing registration-file override supplies the ephemeral registration for the target runtime export.
5. Session startup, discovery, and asset management must resolve the same generated registration file and identity.
6. The development registration is process-scoped or launch-scoped and never written into the durable catalog.
7. WebPortal receives a short-lived bootstrap handle, not unrestricted local filesystem paths or permanent credentials.

### B. Development capability plane

1. GameHost advertises explicit development capabilities.
2. Development endpoints are absent or reject requests unless development launch mode is enabled.
3. WebPortal conditionally exposes tools based on advertised capabilities.
4. Runtime state inspection/mutation and recording/playback are separate capabilities so read-only deployments can remain safer.

### C. Diagnostics plane

1. Preserve a structured browser diagnostic stream for client, renderer, audio, orchestration, and transport events.
2. Add a first-class Diagnostics workspace inside Dev Tools rather than requiring layout-slot manipulation for routine tracing.
3. Model client capture state, console visibility, console docking, and display filters as separate explicit state values.
4. Treat diagnostic profiles as named mappings to Portal client source/category capture policy; entering advanced edits changes the active profile to `Custom`.
5. Export readable text and structured JSON/NDJSON with Portal build identity, the approved correlation-envelope fields available to the Portal, active filters, counts, truncation state, and timestamps.
6. Keep filtering non-destructive: changing severity/category display filters must not delete or omit already captured entries from the default full export.
7. When retention drops entries, show a persistent truncation warning and include exact dropped counts in saved metadata.
8. Do not collect, aggregate, or automatically correlate host/runtime logs in this WebPortal workflow; preserve approved correlation-envelope fields so independently captured traces can be aligned manually.

### D. Presentation isolation plane

1. Effect switches are renderer-local development preferences.
2. Disabling presentation must not suppress authoritative state changes or diagnostic receipt.
3. Switch families include master visuals, movement, appearance, disappearance, room transitions, HUD/text presentation, SFX, and ambient audio.
4. Each effect controller must clean up active visual state when disabled and resume deterministically for subsequent cues.
5. Active switch state must be included in diagnostic exports.

### E. Recording/playback plane

1. Engine/session services own recording and playback state machines.
2. GameHost owns development storage and control endpoints.
3. WebPortal owns controls and status presentation only.
4. Recording is per session and does not accidentally capture commands from other sessions.
5. Playback uses normal command processing and normal delta/presentation delivery.
6. Compatibility import protects existing recordings and test fixtures.

## Initial Scope Breakdown

### A) Ad hoc launch and registration foundation

1. Invocation-scoped one-entry established runtime-registration JSON.
2. Ephemeral registration overlay.
3. Shared registration source.
4. Host readiness/bootstrap and process lifecycle.

Estimated effort: High.

### B) Diagnostics and presentation isolation

1. Engine diagnostic-level controls.
2. Returned-diagnostic ingestion and correlation.
3. Retention/archive/export/bundle behavior.
4. Effect-family and audio-family isolation.

Estimated effort: Medium to High.

### C) Host-owned recording/playback

1. Shared recording schema and legacy compatibility.
2. Per-session recorder and playback coordinator.
3. Host recording store and APIs.
4. WebPortal controls and status.

Estimated effort: High.

### D) Remaining developer capability disposition

1. Save/load game browser workflow.
2. Runtime state inspector and controlled mutation decision.
3. Reinitialize/export refresh.
4. Test migration and simulator freeze.

Estimated effort: Medium to High.

## Structured Delivery Order And Session Handoffs (Feature-Oriented Override Locked)

This workstream is a program of related capabilities, not one feature moving once through every repository. Whole-area sequencing would create oversized batches and delay end-to-end learning. Implementation therefore proceeds through the vertical feature packets in `SIMULATOR_RETIREMENT_READINESS_EXECUTION_ROADMAP.md`.

Feature order and dependencies:

1. F00: Decisions, baseline, and traceability.
2. F01: Ad hoc development launch and automatic bootstrap.
3. F02: First-class diagnostics and trace workflow.
4. F03: Presentation and audio isolation controls.
5. F04: Authoritative recording and playback.
6. F05: Save/load state and latest-export reinitialize.
7. F06: Game-scope dump and debug-command disposition.
8. F07: Parity evidence, test migration, and readiness Go/No-Go.
9. F08: Designer legacy-Simulator launch retirement after F07 `Go`.

Execution rules:

1. Select one roadmap feature packet and carry its bounded slice through every area it needs.
2. Pair the roadmap packet with its feature packet, selected pass handoff, and inherited numbered area profile.
3. Update the selected pass handoff; reuse area profiles in later features without appending execution history to them.
4. Within a feature, additive contract work precedes consumers when contracts are needed, but features without contract changes do not wait for unrelated contract work.
5. Stage 06 non-backward-compatible retirement remains unavailable until all affected feature consumers have migrated and a retirement list is explicitly approved.
6. Stage 07 remains the final readiness validation and Go/No-Go gate; post-Go F08 performs the separately gated Designer launch cutover.
7. No feature is marked complete until all of its required pass handoffs are complete, explicitly not needed, or transferred to an approved follow-up.

## Stage Inclusion Matrix

| Stage ID | Stage Name | Inclusion | Reason if not Required | Boundary Override |
| --- | --- | --- | --- | --- |
| 01 | Contracts And Shared Runtime Mapping (Backwards-Compatible) | Required |  | Reusable area profile; pass handoffs inherit its compatibility and boundary rules. |
| 02 | Designer Authoring UX | Required |  | Reusable area profile; pass handoffs inherit its boundary and validation rules. |
| 03 | GameEngine Runtime Integration | Required |  | Reusable area profile; pass handoffs inherit its boundary and validation rules. |
| 04 | Host Interface And Web Portal Runtime Consumption | Required |  | Reusable area profile; pass handoffs inherit its boundary and validation rules. |
| 05 | Simulator Host Parity | Required | Recast as replacement audit, compatibility-fixture migration, and no-new-renderer freeze. | Living audit ledger revisited after each feature; standard simulator edit boundary remains. |
| 06 | Contract Retirement (Non-Backwards-Compatible) | Skipped | No contract retirement is approved for readiness work. Actual simulator/code deletion belongs to a follow-on retirement plan. | Placeholder retained to record the skip decision. |
| 07 | Regression Hardening And Closeout | Required |  | Receives evidence references during feature passes and becomes authoritative only during F07. |

Current area-ledger progress:

- Contracts/Shared ledger: Open; no feature passes started.
- Designer ledger: Open; no feature passes started.
- Area profiles: Active reusable boundary definitions; no execution history is stored in them.
- F01 pass handoffs: First-wave placeholders created as the working execution-model example; no implementation started.
- Other feature pass handoffs: Created when their roadmap pass is scheduled, not as speculative placeholders.
- Feature status: maintained in `SIMULATOR_RETIREMENT_READINESS_EXECUTION_ROADMAP.md`.
- Workstream status: Planning; implementation not authorized by this document alone.

Execution document set:

1. Area profiles: `plans/active/SIMULATOR_RETIREMENT_READINESS/area-profiles/`.
2. Feature packets: `plans/active/SIMULATOR_RETIREMENT_READINESS/features/<FEATURE>/`.
3. Pass-handoff template: `plans/active/SIMULATOR_RETIREMENT_READINESS/features/PASS_HANDOFF_TEMPLATE.md`.
4. Instantiated F01 first-wave pass handoffs: `features/F01_AD_HOC_LAUNCH/F01.1` through `F01.4`.

Execution overlay:

1. `plans/active/SIMULATOR_RETIREMENT_READINESS/SIMULATOR_RETIREMENT_READINESS_EXECUTION_ROADMAP.md`
2. Every implementation prompt must name one roadmap feature ID, feature packet, pass handoff, and inherited area profile.

## Stage 01: Contracts And Shared Runtime Mapping

Goal:

1. Lock diagnostics, recording/playback, game-scope-dump, and other genuine shared contracts as backward-compatible additions; F01 launch uses the established runtime-registration JSON format without a contract change.
2. Establish a canonical recording model outside the simulator.

Codebase context:

1. Primary projects: `Storyboard.Shared.Contracts`, `Storyboard.SchemaCodegen`, transport guardrail tests.
2. Expected first areas: host capability contracts, recording metadata/step/control/result DTOs, recording/playback interface methods, the shared trace-correlation envelope, and game-scope-dump shapes.
3. Boundary constraints: additive only; no accepted member removal, rename, type narrowing, or requiredness tightening. Do not expose machine-local runtime paths through general production contracts.
4. Required validation: shared contract builds, schema/codegen verification, interface/DTO guardrails, JSON round trips, compatibility read of representative `.sbe.sim.json` fixtures.
5. Allowed read scope: `plans/**`, `Storyboard.TransportCodegen.Tests/**`, `Storyboard.GameEngine/Storyboard.Simulator/**`, `Storyboard.GameEngine/Storyboard.Simulator.Tests/Fixtures/**`, `Storyboard.GameEngine/Storyboard.GameEngine/**`, `Storyboard.WebPortal/Storyboard.WebPortal/src/hostApi/**`.
6. Allowed edit scope: `Storyboard.Contracts/Storyboard.Shared.Contracts/**`, `Storyboard.Contracts/Storyboard.SchemaCodegen/**`, `Storyboard.Contracts/Storyboard.TransportCodegen.Tests/**`, `StoryBoard.Architecture/plans/active/SIMULATOR_RETIREMENT_READINESS/**`.

Primary area profile:

- `plans/active/SIMULATOR_RETIREMENT_READINESS/area-profiles/01_CONTRACTS_SHARED_PROFILE.md`

Required pass-handoff outputs:

1. Confirmation that F01 uses the established runtime-registration JSON format without a new contract, plus locked recording ownership decisions.
2. Added contract/schema inventory and compatibility declaration.
3. Legacy recording compatibility evidence.
4. Approved shared trace-correlation and game-scope-dump contract decisions when their feature passes begin.
5. Validation and boundary compliance reports.

Downstream usage:

1. For each selected roadmap feature, only that feature's approved contract slice is handed to its next consumer area.
2. Contract work may be revisited by later feature packets; completing one pass does not close this ledger.

## Stage 02: Designer Authoring UX

Goal:

1. Add development-host launch orchestration alongside the existing simulator-process launch while preserving the one-action producer workflow; F01-F07 do not remove the legacy launch path.

Codebase context:

1. Primary projects: `StoryboardDesigner.App`, designer unit tests, designer smoke tests.
2. Expected first files: `ExternalSimulatorWorkflowService.cs`, launch request/result types, simulator setup model/dialog, main window run command, project launch settings, process/lifecycle adapter.
3. Boundary constraints: Designer may save, validate, export, construct a launch request, start the launcher/host, and report status. It must not implement session runtime, mutate the durable game catalog, or parse host state.
4. Required validation: launch request composition, paths with spaces, fresh-project export, validation-continue behavior, host readiness failure, duplicate/restart behavior, UI responsiveness, smoke launch.
5. Allowed read scope: `StoryBoard.Architecture/plans/**`, `Storyboard.Contracts/Storyboard.Shared.Contracts/**`, `StoryBoard.GameEngine/Storyboard.GameHost/**`.
6. Allowed edit scope: `StoryBoard.Designer/StoryboardDesigner.App/**`, `StoryBoard.Designer/StoryboardDesigner.App.Tests/**`, `StoryBoard.Designer/StoryboardDesigner.App.SmokeTests/**`, `StoryBoard.Architecture/plans/active/SIMULATOR_RETIREMENT_READINESS/**`.

Primary area profile:

- `plans/active/SIMULATOR_RETIREMENT_READINESS/area-profiles/02_DESIGNER_PROFILE.md`

Required pass-handoff outputs:

1. Fresh unregistered project launch evidence.
2. Exact manifest/arguments produced by Designer with secrets and local paths redacted in examples.
3. Process lifecycle and failure UX behavior.
4. Confirmation that the production catalog is untouched.
5. Validation and boundary compliance reports.

Downstream usage:

1. The next area named by the selected feature packet consumes the Designer output from that pass.
2. Later Designer features revisit this ledger independently.

## Stage 03: GameEngine Runtime Integration

Goal:

1. Implement authoritative development registration, recording/playback, approved shared trace-correlation metadata, and approved developer capabilities in GameEngine/GameHost.

Codebase context:

1. Primary projects: `Storyboard.GameEngine`, `Storyboard.Shared`, `Storyboard.GameHost`, `Storyboard.GameEngine.Tests`, `Storyboard.GameClient.Tests`.
2. Expected first areas: runtime registration source/factory, `GameManager` session startup, discovery provider composition, asset-management registration lookup, host capability reporting, session recording service, playback coordinator, development recording store, correlation-envelope emission in existing traces, and host startup options.
3. Boundary constraints: one registration source must serve discovery/session/assets; recording and playback are session-isolated; replay uses normal command processing; development capabilities are disabled in normal production startup; no WPF or browser dependencies enter runtime projects.
4. Required validation: ad hoc registration integration, session start and asset retrieval, multi-session isolation, recording lifecycle, timed and stepped replay, cancellation, legacy import, save/load state regressions, capability/security rejection, GameHost socket smoke.
5. Allowed read scope: `StoryBoard.Architecture/plans/**`, `Storyboard.Contracts/Storyboard.Shared.Contracts/**`, `StoryBoard.Designer/StoryboardDesigner.App.Tests/**`, `StoryBoard.WebPortal/Storyboard.WebPortal/src/hostApi/**`, `StoryBoard.GameEngine/Storyboard.Simulator.Tests/Fixtures/**`.
6. Allowed edit scope: `StoryBoard.GameEngine/Storyboard.GameEngine/**`, `StoryBoard.GameEngine/Storyboard.GameEngine.Tests/**`, `StoryBoard.GameEngine/Storyboard.Shared/**`, `StoryBoard.GameEngine/Storyboard.GameHost/**`, `StoryBoard.GameEngine/Storyboard.GameClient/**`, `StoryBoard.GameEngine/Storyboard.GameClient.Tests/**`, `StoryBoard.GameEngine/Storyboard.GameEngine.Configuration/**`, `StoryBoard.Architecture/plans/active/SIMULATOR_RETIREMENT_READINESS/**`.

Primary area profile:

- `plans/active/SIMULATOR_RETIREMENT_READINESS/area-profiles/03_HOST_ENGINE_PROFILE.md`

Required pass-handoff outputs:

1. Registration-source ownership and production/development composition evidence.
2. Recording/playback state machine and storage behavior.
3. Development capability and security behavior.
4. Correlation/diagnostic behavior exposed to clients.
5. Focused and socket-level validation results.

Downstream usage:

1. The selected feature's Portal pass consumes only the host behavior and evidence completed for that feature.
2. Later engine/host features revisit this ledger independently.

## Stage 04: Host Interface And WebPortal Runtime Consumption

Goal:

1. Deliver the browser-based development experience required to replace day-to-day simulator use.

Codebase context:

1. Primary projects: `Storyboard.WebPortal`, WebPortal tests/visual tests, shared host interfaces where additive consumption fixes remain necessary.
2. Expected first areas: development bootstrap workflow, DevTools sections, diagnostics pipeline and export, command/poll diagnostic-level settings, renderer effect controllers, audio controls, recording/playback UI, save/load UI, capability-gated runtime inspector.
3. Boundary constraints: Pixi never calls host APIs directly; hooks/workflows own transport; effect toggles do not mutate runtime truth; browser never receives unrestricted machine filesystem access; production UI hides or disables unsupported development tools.
4. Required validation: WebPortal build/test, host-client contract verification, component/workflow tests, renderer lifecycle tests for toggles, recording/playback controls, diagnostic export tests, visual baselines, browser-host smoke.
5. Allowed read scope: `StoryBoard.Architecture/plans/**`, `Storyboard.Contracts/Storyboard.Shared.Contracts/**`, `StoryBoard.GameEngine/Storyboard.GameHost/**`, `StoryBoard.GameEngine/Storyboard.GameEngine/**`, `StoryBoard.GameEngine/Storyboard.GameClient.Tests/**`, `StoryBoard.GameEngine/Storyboard.Simulator/**`, `StoryBoard.GameEngine/Storyboard.Simulator.Tests/**`.
6. Allowed edit scope: `StoryBoard.WebPortal/Storyboard.WebPortal/**`, `StoryBoard.WebPortal/Storyboard.WebPortal.Tests/**`, `StoryBoard.GameEngine/Storyboard.Shared/**`, `Storyboard.Contracts/Storyboard.Shared.Contracts/**`, `StoryBoard.Architecture/plans/active/SIMULATOR_RETIREMENT_READINESS/**`.

Primary area profile:

- `plans/active/SIMULATOR_RETIREMENT_READINESS/area-profiles/04_WEBPORTAL_PROFILE.md`

Required pass-handoff outputs:

1. Automatic development bootstrap evidence.
2. Completed parity-matrix rows and screenshots/test references where relevant.
3. Diagnostics-level, ingestion, correlation, retention, and export evidence.
4. Diagnostics-workspace usability evidence proving trace, show/hide, save, clear, profiles, and docking do not require indirect slot configuration.
5. Presentation/audio isolation behavior.
6. Recording/playback and save/load behavior.
7. Any runtime-inspector decision or remaining explicit gap.

Downstream usage:

1. The selected feature's simulator-audit pass compares only that replacement slice against the WPF baseline.
2. Later Portal features revisit this ledger independently.

## Stage 05: Simulator Replacement Parity Audit And Freeze

Goal:

1. Prove the replacement covers the approved simulator capability set, preserve reusable semantic evidence, and prevent new WPF renderer investment.

Codebase context:

1. Primary projects: `Storyboard.Simulator.Tests`, `Storyboard.Simulator.SmokeTests`, simulator compatibility adapters only where necessary for fixture migration.
2. Expected first areas: simulator capability inventory, recording fixtures, replay-speed tests, diagnostics behavior tests, launch smoke tests, architecture guardrails.
3. Boundary constraints: no new WPF renderer feature work; do not make the simulator consume new contracts merely to prolong parity unless needed for compatibility validation; move tests to the owning runtime/WebPortal repository before deleting or weakening them.
4. Required validation: simulator baseline remains usable during the audit, every matrix row has Present/Replaced/Waived status, existing recording fixtures pass through the new engine-owned playback path, WebPortal visual tests own runtime rendering assertions.
5. Allowed read scope: `StoryBoard.Architecture/plans/**`, `StoryBoard.GameEngine/Storyboard.Shared/**`, `Storyboard.Contracts/Storyboard.Shared.Contracts/**`, `StoryBoard.WebPortal/Storyboard.WebPortal/**`, `StoryBoard.Designer/StoryboardDesigner.App/**`.
6. Allowed edit scope: `StoryBoard.GameEngine/Storyboard.Simulator/**`, `StoryBoard.GameEngine/Storyboard.Simulator.Tests/**`, `StoryBoard.GameEngine/Storyboard.Simulator.SmokeTests/**`, `StoryBoard.Architecture/plans/active/SIMULATOR_RETIREMENT_READINESS/**`.

Primary area profile:

- `plans/active/SIMULATOR_RETIREMENT_READINESS/area-profiles/05_SIMULATOR_AUDIT_PROFILE.md`

Required pass-handoff outputs:

1. Final capability disposition matrix.
2. List of simulator tests ported, retained temporarily, or approved for eventual deletion.
3. Explicit no-new-WPF-renderer freeze statement.
4. Remaining P0/P1 gaps and waiver decisions.
5. Readiness recommendation to Stage 07.

Downstream usage:

1. Each feature audit feeds its evidence reference into the closeout ledger.
2. F07 consolidates the cumulative audit into the retirement Go/No-Go recommendation.

## Stage 06: Contract Retirement (Skipped)

Goal:

1. No work is authorized initially.
2. Reopen only if completed feature passes produce an explicit, approved non-backward-compatible contract retirement list and all affected consumers have migrated.

Codebase context:

1. Primary projects if reopened: `Storyboard.Shared.Contracts`, `Storyboard.SchemaCodegen`.
2. Boundary constraints: consumer migration must already be complete; actual WPF application deletion does not belong in this contract-only stage.
3. Required validation if reopened: contract guardrails, schema/codegen verification, downstream impact report.
4. Allowed read scope: `plans/**`, `Storyboard.TransportCodegen.Tests/**`.
5. Allowed edit scope if explicitly reopened: `Storyboard.Contracts/Storyboard.Shared.Contracts/**`, `Storyboard.Contracts/Storyboard.SchemaCodegen/**`, `Storyboard.Contracts/Storyboard.TransportCodegen.Tests/**`, `StoryBoard.Architecture/plans/active/SIMULATOR_RETIREMENT_READINESS/**`.

Primary area profile:

- `plans/active/SIMULATOR_RETIREMENT_READINESS/area-profiles/06_CONTRACT_RETIREMENT_PROFILE.md`

Required pass-handoff outputs:

1. Skip confirmation, or approved retirement list and compatibility evidence if reopened.
2. Downstream impact and boundary compliance report.

Downstream usage:

1. F07 confirms whether this ledger remained skipped or records the approved retirement evidence.

## Stage 07: Regression Hardening, Retirement Readiness Decision, And Closeout

Goal:

1. Validate the complete Designer-to-GameHost-to-WebPortal development loop and issue a documented go/no-go decision that authorizes post-Go F08 Designer launch retirement; full WPF application/code removal remains a separate workstream.

Codebase context:

1. Primary projects: repository-wide validation and workstream documentation.
2. Expected first areas: cross-repository integration tests, process/port failure cases, visual baselines, replay fixtures, diagnostic exports, security/capability tests, plan/handoff status.
3. Boundary constraints: product fixes discovered here must return to the owning stage unless they are narrow test stabilization; no simulator deletion in this readiness closeout.
4. Required validation: all commands in Validation Gates, manual acceptance scenarios, parity matrix with no unapproved P0 gaps, and explicit P1 waivers/follow-ups.
5. Allowed read scope: repository-wide for validation evidence.
6. Allowed edit scope: tests/snapshots needed for stabilization and `StoryBoard.Architecture/plans/active/SIMULATOR_RETIREMENT_READINESS/**`.

Primary area profile:

- `plans/active/SIMULATOR_RETIREMENT_READINESS/area-profiles/07_INTEGRATION_CLOSEOUT_PROFILE.md`

Required pass-handoff outputs:

1. End-to-end validation results.
2. Completed parity and disposition matrix.
3. Production-isolation/security evidence.
4. Go/no-go recommendation and rationale.
5. If go: exact scope for the follow-on simulator removal/governance-update plan.
6. If no-go: blocking owner, failing acceptance criterion, and next corrective stage.

## Risk Register

1. **Ad hoc identity drift:** session startup and asset lookup may use different game identities. Mitigation: one injected registration source and integration tests spanning discovery, session creation, and asset retrieval.
2. **Production exposure of developer controls:** replay, mutation, or local file capabilities could be dangerous. Mitigation: explicit development startup mode, loopback default, capability advertisement, authorization, and negative production tests.
3. **Launcher fragility:** ports, readiness, stale processes, and browser startup can erode the one-click workflow. Mitigation: launch correlation IDs, readiness endpoint, deterministic lifecycle policy, bounded retries, and actionable Designer output.
4. **Client/host version skew:** independently built WebPortal can silently mismatch host contracts. Mitigation: host-served packaged bundle by default and bootstrap version handshake.
5. **Recording nondeterminism:** client-driven timing and concurrent session activity can make replay unreliable. Mitigation: authoritative per-session recorder, serialized playback coordinator, normal command path, and fixture-based assertions.
6. **Legacy recording loss:** private simulator schema can be accidentally abandoned. Mitigation: characterize existing files, add compatibility reader tests, and preserve fixtures before moving ownership.
7. **Diagnostic overload:** richer diagnostics can flood the browser or hide important signals. Mitigation: structured categories, levels, bounded retention with truncation disclosure, sampling, and downloadable bundles.
8. **False diagnostic parity:** a console can look rich while dropping engine/session diagnostics. Mitigation: source-by-source coverage tests and correlation-based acceptance scenarios.
9. **Effect-disable state corruption:** disabling an effect mid-animation may leave Pixi nodes or timers alive. Mitigation: controller lifecycle tests for disable, re-enable, scene change, object removal, and disposal.
10. **Browser file limitations:** save/load and recording portability may be weakened. Mitigation: host-owned stores plus explicit browser upload/download flows.
11. **Runtime inspector scope creep:** recreating a live WPF debugger or exposing arbitrary process memory could create a second major UI project and security boundary. Mitigation: use an on-demand complete semantic game-scope dump in the existing Engine flow trace for P0, defer the Portal tree viewer to P1, and defer mutation.
12. **Premature simulator neglect:** blocking defects may make the existing fallback unusable during migration. Mitigation: permit critical fixes, but require architecture approval for new WPF visual behavior.
13. **Active-plan governance drift:** existing plans assume a simulator parity stage. Mitigation: after readiness go decision, update the standard stage catalog and active workstreams in the dedicated retirement/governance plan.

## Validation Gates

Commands must be run from their owning repository unless otherwise stated. Exact solution names may be adjusted to repository-local guidance, but the coverage intent is mandatory.

### Contracts

1. Build the contracts solution/projects.
2. `dotnet test .\Storyboard.TransportCodegen.Tests\Storyboard.TransportCodegen.Tests.csproj`
3. Run contract/schema lock and generated-manifest verification.

### Designer

1. Build the Designer solution.
2. `dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj`
3. `dotnet test .\StoryboardDesigner.App.SmokeTests\StoryboardDesigner.App.SmokeTests.csproj`
4. Focused fresh-project launch test proving no durable catalog entry is needed.

### GameEngine and GameHost

1. Build the GameEngine solution/projects.
2. `dotnet test .\Storyboard.GameEngine.Tests\Storyboard.GameEngine.Tests.csproj`
3. `dotnet test .\Storyboard.GameClient.Tests\Storyboard.GameClient.Tests.csproj`
4. Focused registration-source, session-isolation, persistence, recording/playback, legacy fixture, capability, and socket-smoke suites.

### WebPortal

1. `npm test`
2. `npm run build`
3. `npm run verify:contracts`
4. `npm run test:visual`
5. Focused component tests for client diagnostics profiles/export, correlation-envelope fields, effect toggles, recording/playback, save/load, and development bootstrap.

### End-to-end manual acceptance

1. Create a brand-new Designer project that has never been added to the production catalog.
2. Save/export and use the Designer launch action.
3. Confirm an isolated host starts, WebPortal opens, the correct game/session attaches automatically, and assets load.
4. Make an authoring change, relaunch/reset, and confirm the new export is active.
5. Toggle each presentation and audio family independently while confirming runtime state still advances.
6. Exercise None/Low/Medium/High diagnostics and verify command, session, host, and renderer correlation.
7. Open Dev Tools, enter the first-level Diagnostics workspace, start trace, show the console, hide and re-show it, dock/undock it, stop trace, and save without visiting layout-slot configuration or closing Dev Tools.
8. Verify hiding the console does not stop capture, stopping capture does not clear entries, docking does not change trace scope, and display filters do not change the default full export.
9. Save readable and structured diagnostic output from the Diagnostics workspace while the console is hidden; verify metadata, active profile/scope, capture boundary, and truncation disclosure.
10. Record commands including at least one clarification and one movement sequence; stop, list, replay timed, pause/stop, step, change speed, and continue/branch as approved.
11. Save and restore game state through WebPortal.
12. Confirm production-mode GameHost rejects or does not expose development registration, recording, replay, and mutation operations.

## MVP / Completion Acceptance Criteria

1. Designer can launch an unregistered fresh project end-to-end without modifying the durable game catalog.
2. Development registration identity is consistent across discovery/bootstrap, session startup, diagnostics, and asset APIs.
3. WebPortal automatically reaches the correct active development session with no manual discovery sequence.
4. Host lifecycle, port selection, restart/reuse, readiness, and failure cleanup are deterministic and diagnosable.
5. WebPortal/Pixi is the only renderer used in the replacement development workflow.
6. All P0 rows in the parity matrix are `Replaced` or `Present`; none may be waived.
7. Every P1 row is `Replaced`, `Present`, or has an explicit approved waiver and named follow-up owner.
8. Diagnostics provide direct WebPortal client trace controls, bounded-retention disclosure, and both readable-text and structured/NDJSON export; each trace preserves approved correlation-envelope fields for manual alignment with separately captured Host and Engine traces.
9. Dev Tools provides a permanent first-level Diagnostics workspace with direct trace start/stop, console show/hide, save, clear, profile/scope, capture status, entry count, and truncation status.
10. The normal diagnostics workflow requires no layout-slot edits and no closing of Dev Tools; capture, visibility, docking, filtering, and saving remain independent and predictable.
11. Saving the complete capture works while the console is hidden and includes enough metadata to reproduce the diagnostic configuration.
12. Presentation and audio families can be isolated without changing authoritative game behavior.
13. Recording and playback are authoritative, session-isolated, controllable from WebPortal, and compatible with approved existing fixtures.
14. Save/load game behavior is available from WebPortal or has an explicit approved non-blocking waiver.
15. A development-capability-gated WebPortal request produces a manually requested, complete read-only game-scope dump in the GameEngine flow trace; mutation remains explicitly deferred and is not exposed.
16. WebPortal renderer semantic and visual regression coverage owns all critical runtime presentation behavior previously guarded only by simulator tests.
17. Development-only capabilities are unavailable in production-mode host validation.
18. Stage 07 records a formal readiness go/no-go. A `Go` authorizes F08 to remove the Designer legacy-Simulator launch path; it does not authorize WPF application/code deletion, which remains a separate removal and governance-update workstream.

## Final Closeout Checklist

1. All required roadmap features and affected area-ledger passes are complete, Stage 06 skip/reopen status is recorded, and F08 is either completed after a F07 `Go` or explicitly not started because F07 recorded `No-Go`.
2. All required feature packets and pass handoffs exist, contain final status, and inherit the applicable area profile.
3. Decision gates are locked or explicitly deferred with owners.
4. Parity matrix contains a final disposition for every row.
5. All validation outcomes and manual acceptance evidence are recorded.
6. No unapproved P0 gap remains.
7. P1 waivers have named follow-up plans and owners.
8. Production-isolation and negative security tests pass.
9. Go/no-go decision is recorded.
10. If `Go`, create the follow-on simulator removal/governance-update plan before deleting product code or changing the standard stage catalog.
11. Archive this entire workstream folder as one unit only after readiness work is complete.
