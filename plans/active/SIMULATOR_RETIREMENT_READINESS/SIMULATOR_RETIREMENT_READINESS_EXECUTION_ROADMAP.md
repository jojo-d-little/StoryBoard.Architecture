# Simulator Retirement Readiness Feature Execution Roadmap

Last updated: 2026-09-24
Status: Draft / execution overlay
Owner: Cross-repository architecture workstream

Purpose: sequence simulator-retirement readiness as vertical feature passes that cross contracts, Designer, GameEngine/GameHost, WebPortal, and simulator-audit boundaries. This roadmap is the dispatch index used alongside the main plan, feature packets, pass handoffs, and reusable area profiles.

## Document Roles

1. `SIMULATOR_RETIREMENT_READINESS_PLAN.md` owns architecture direction, complete scope, parity requirements, risks, and final readiness criteria.
2. This roadmap owns feature selection, dependency order, per-feature cross-area sequence, and feature-level completion state.
3. The `area-profiles/01` through `07` documents define reusable ownership, compatibility, allowlist, and validation rules; they are not handoffs or progress ledgers.
4. A working session receives:
   - this roadmap, one selected feature packet, and one selected pass handoff;
   - the area profile inherited by that pass;
   - the main plan for architecture and acceptance constraints.
5. A feature is complete only when its packet exit criteria are satisfied across every required pass. Completing one pass does not complete the feature.

## Plan-Specific Execution Override

The standard stage catalog remains the source for ownership boundaries and contract compatibility, but its normal whole-area sequential cadence is overridden for this program-sized workstream.

1. Do not complete all contract work, then all Designer work, then all engine work, then all portal work.
2. Select one feature packet and carry the smallest coherent vertical slice through its required areas.
3. Reuse the same area profile in later feature passes without turning the profile into a cumulative progress document.
4. Stage 01 rules still apply whenever a feature needs additive contracts: contract/schema work precedes consumers for that feature.
5. Stage 06 rules still apply globally: no non-backward-compatible contract retirement occurs until all affected feature consumers have migrated and an explicit retirement list is approved.
6. Stage 07 remains the final whole-workstream regression and Go/No-Go gate.
7. The no-new-WPF-renderer rule applies to every feature pass.

## Feature-Pass Protocol

For each implementation session:

1. Choose exactly one feature ID from this roadmap unless the roadmap explicitly permits a paired slice.
2. State the feature's in-scope acceptance slice, selected pass handoff, and inherited area profile.
3. Read the selected feature packet, pass handoff, main plan, and inherited area profile.
4. Follow the feature's local area order. Do not assume numeric area/stage order when the packet specifies otherwise.
5. Respect the area-profile allowlist. The effective edit scope is the intersection of the selected feature, pass handoff, and area profile.
6. Update the selected pass handoff with:
   - Feature ID and pass number.
   - Scope attempted and completed.
   - Files changed.
   - Contract/interface effect.
   - Validation results.
   - Remaining feature gaps.
   - Exact next area/pass starting point.
7. Update this roadmap's feature status only after reviewing all required pass handoffs.
8. Do not mark a feature complete until every required pass is complete, explicitly not needed, or moved to an approved follow-up.

## Ordered Area-Pass Rule

Before implementation begins for any feature, its feature packet must contain an ordered area-pass execution matrix modeled on F01.

1. Every implementation row has exactly one product-area owner and one primary area profile; it does not authorize a session to make unrelated cross-repository product edits.
2. Integration and simulator-audit rows are explicitly labeled evidence-only; they identify and assign gaps but return product fixes to the owning area row.
3. Every row names its prerequisite pass(es), allowed product area, required handoff output, focused validation, and exact next pass.
4. A feature may not begin product implementation until its matrix is reviewed; a newly discovered required area pass is added to the matrix before work starts.
5. The matrix records the natural dependency-driven order for that feature. Prefer area-first sequencing when dependencies allow it, so known work for one owner area is completed before context switches; interleave areas only when a named prerequisite genuinely requires it.
6. Revisit an area after an explicit integration checkpoint assigns a real gap to it; do not pre-schedule speculative back-and-forth remediation merely to satisfy a fixed cadence.

## Feature Dependency Map

```text
F00 Decisions and Baseline
  ├─► F01 Ad Hoc Launch and Bootstrap ─┬─► F04 Recording and Playback
  │                                    ├─► F05 Save/Load and Reinitialize
  │                                    └─► F06 Runtime Inspection Decision/Delivery
  ├─► F02 Diagnostics Workflow
  └─► F03 Presentation and Audio Isolation

F01 + F02 + F03 + F04 + F05 + F06
  └─► F07 Parity Evidence, Test Migration, and Readiness Go/No-Go
        └─► F08 Designer Legacy-Simulator Launch Retirement
```

Execution guidance:

1. `F00` is first.
2. `F01` is the first implementation foundation because it establishes the real development-host session.
3. `F02` and `F03` may begin after `F00` and can proceed while later `F01` hardening continues, but their end-to-end acceptance requires the development launch path.
4. `F04`, `F05`, and `F06` depend on the development capability/security and session identity foundation from `F01`.
5. `F07` begins only after all prior P0 work is complete and all P1 dispositions are recorded.
6. `F08` begins only after F07 records a `Go`; it removes the Designer's legacy Simulator launch path but does not authorize WPF application/code deletion.

## Feature Status Board

| ID | Feature | Priority | Status | Primary dependency | Areas revisited |
| --- | --- | --- | --- | --- | --- |
| F00 | Decisions, baseline, and traceability | P0 | Not started | None | Architecture, Contracts, Designer, Engine, Portal, Simulator audit |
| F01 | Ad hoc development launch and automatic bootstrap | P0 | Implementation complete; closure pending | F00 | Contracts, Designer, Engine/Host, Portal, Simulator audit |
| F02 | First-class diagnostics and trace workflow | P0 | Complete; follow-up refinements recorded | F00; F01 for full E2E | Contracts if needed, Engine/Host, Portal, Simulator audit |
| F03 | Presentation and audio isolation controls | P1 | Not started | F00; F01 for full E2E | Engine semantics review, Portal, Simulator audit |
| F04 | Authoritative recording and playback | P0 | Not started | F01 | Contracts, Engine/Host, Portal, Designer startup options, Simulator audit |
| F05 | Save/load state and latest-export reinitialize | P0/P1 | Not started | F01 | Designer, Engine/Host, Portal, Simulator audit |
| F06 | Game-scope dump and remaining debug-command disposition | P0 dump; P1/P2 disposition | Not started | F01 | Contracts, Engine/Host, Portal, Simulator audit |
| F07 | Parity evidence, test migration, and retirement-readiness decision | P0 | Not started | F01-F06 | All ledgers, closeout |
| F08 | Designer legacy-Simulator launch retirement | Post-Go cutover | Not started | F07 Go | Designer, Simulator audit, closeout |

## Pass Catalog (Planning-Level Work Units)

This is the one-page inventory used to select and sequence work. A pass is the smallest planned cross-area outcome, not necessarily one developer session; a pass may be divided only by updating this catalog and its feature packet so its evidence and next-pass contract remain clear.

Area legend: 🟦 **Host / Engine** · 🟨 **Designer** · 🟪 **WebPortal** · 🟩 **Contracts** · 🟧 **Simulator Audit** · ⬜ **Integration / Architecture** · 🟥 **Closeout**

### F00: Decisions, Baseline, And Traceability

| Area | Pass | One-sentence outcome |
| --- | --- | --- |
| ⬜ **Architecture** | F00.1 | Record the approved architecture decisions, remaining implementation-level approval gates, and decision ownership in the roadmap and ledgers. |
| ⬜ **Integration** | F00.2 | Capture read-only Designer, Engine, Host, Portal, and Simulator baseline evidence for every parity row and fixture that later passes must replace or preserve. |

### F01: Ad Hoc Development Launch And Automatic Bootstrap

| Area | Pass | One-sentence outcome |
| --- | --- | --- |
| 🟦 **Host / Engine** | F01.1 | Complete the known Host/Engine foundation: prove the one-entry registration-file override resolves one identity through discovery, sessions, and assets; implement needed readiness, lifecycle, restart/reuse, port, and production-isolation behavior. |
| 🟨 **Designer** | F01.2 | Complete the known Designer foundation: add the distinct GameHost/WebPortal action, generate the temporary one-entry registration JSON, launch/manage the host and browser, provide failure/retry/cleanup UX, and preserve the WPF Simulator action unchanged. |
| 🟪 **WebPortal** | F01.3 | Complete the known Portal foundation: automatically attach to the intended development game/session using the existing host flow, with no browser filesystem access or assumed new contract. |
| ⬜ **Integration** | F01.4 | Run the first no-code vertical integration test across the completed Host, Designer, and Portal foundations and assign each discovered gap to one owning area. |
| 🟦 **Host / Engine** | F01.5 | Complete unresolved Host/Engine gaps after F01.4, or record that earlier inline remediation absorbed the pass. |
| 🟨 **Designer** | F01.6 | Complete unresolved Designer gaps after F01.4, or record that earlier inline remediation absorbed the pass. |
| 🟪 **WebPortal** | F01.7 | Complete unresolved Portal gaps after F01.4, or record that earlier inline remediation absorbed the pass. |
| ⬜ **Integration** | F01.8 | Run final vertical fresh-project launch regression, including paths with spaces, invalid export, relaunch, and production-negative cases, while separately proving the legacy Simulator action still works. |
| 🟧 **Simulator Audit** | F01.9 | Compare startup/reset/duplicate-launch behavior and disposition the corresponding tests and fixtures. |

### F02: First-Class Diagnostics And Trace Workflow

| Area | Pass | One-sentence outcome |
| --- | --- | --- |
| 🟩 **Contracts** | F02.1 | Define the approved shared trace-correlation envelope in `Storyboard.Contracts` for separately produced Engine, Host, and Portal trace files. |
| 🟪 **WebPortal** | F02.2 | Inventory the Portal-only trace sources and categories required for client, renderer, audio, orchestration, and client-observed transport troubleshooting. |
| 🟪 **WebPortal** | F02.3 | Implement the first-level Portal Diagnostics workspace with direct capture, visibility, docking, clear, profile, and filter controls. |
| 🟪 **WebPortal** | F02.4 | Implement full-buffer readable-text and structured/NDJSON Portal trace export with capture metadata and truncation disclosure. |
| 🟧 **Simulator Audit** | F02.5 | Audit the approved Portal troubleshooting workflow against the Simulator console/archive behavior without adding backend-log aggregation. |
| ⬜ **Integration** | F02.6 | Prove the Portal trace workflow end to end and manually align representative Portal, Host, and Engine trace records using the shared envelope. |

### F03: Presentation And Audio Isolation Controls

| Area | Pass | One-sentence outcome |
| --- | --- | --- |
| 🟦 **Host / Engine** | F03.1 | Verify Engine cue/state semantics so Portal suppression controls cannot change authoritative runtime behavior. |
| 🟪 **WebPortal** | F03.2 | Add Portal controls for master visuals plus movement, appearance, and disappearance presentation families. |
| 🟪 **WebPortal** | F03.3 | Add Portal controls for room transitions, HUD/text presentation, all audio, SFX, and ambient audio families. |
| 🟪 **WebPortal** | F03.4 | Prove deterministic cleanup when controls change during active effects, removal, transition, reset, and renderer disposal. |
| 🟧 **Simulator Audit** | F03.5 | Audit each Simulator switch and explicitly disposition unmatched behavior. |
| ⬜ **Integration** | F03.6 | Prove developers can isolate each approved presentation/audio family during a hosted troubleshooting scenario. |

### F04: Authoritative Recording And Playback

| Area | Pass | One-sentence outcome |
| --- | --- | --- |
| ⬜ **Architecture** | F04.1 | Produce and obtain explicit approval for the implementation-level Recording And Playback Interface Specification. |
| 🟩 **Contracts** | F04.2 | Define the approved canonical recording, library, playback, capability, and legacy-compatibility contract in `Storyboard.Contracts`. |
| 🟦 **Host / Engine** | F04.3 | Implement Engine recording of accepted session activity and outcomes using the authoritative command path. |
| 🟦 **Host / Engine** | F04.4 | Implement host-managed project recording-library access, atomic saves/promotions, import/export, and development security. |
| 🟪 **WebPortal** | F04.5 | Implement Portal recording start/stop, project-library list/select/detail, save/promotion, import/export, and status controls. |
| 🟦 **Host / Engine** | F04.6 | Implement host-authoritative timed and manual-step playback, pause/stop, speed, cancellation, and normal command-path execution. |
| 🟪 **WebPortal** | F04.7 | Implement Portal playback controls, next-command preview, manual advance, progress, outcomes, failures, and diagnostics. |
| 🟨 **Designer** | F04.8 | Add Designer support for approved optional startup replay and replay-speed settings by invoking the approved recording/playback host interface after the development session is ready. |
| 🟧 **Simulator Audit** | F04.9 | Run legacy fixtures and Simulator recording/replay semantics through the new owner and disposition the results. |
| 🟩 **Contracts** | F04.10 | Apply only approved additive contract corrections discovered by the completed vertical slice. |

### F05: Save/Load State And Latest-Export Reinitialize

| Area | Pass | One-sentence outcome |
| --- | --- | --- |
| 🟦 **Host / Engine** | F05.1 | Review existing Engine/Host persistence APIs and document the minimum gaps for browser capture, restore, and latest-export reset. |
| 🟪 **WebPortal** | F05.2 | Implement Portal capture/download, upload/validate/restore, baseline resynchronization, and clear failure feedback. |
| 🟨 **Designer** | F05.3 | Implement Designer/launcher re-export and relaunch/reset behavior against the newest runtime export. |
| 🟦 **Host / Engine** | F05.4 | Correct Engine/Host base-game identity, locator, and reset behavior found by vertical validation. |
| 🟧 **Simulator Audit** | F05.5 | Audit Simulator save/load/reinitialize behavior and disposition replacement evidence. |

### F06: Game-Scope Dump And Debug-Command Disposition

| Area | Pass | One-sentence outcome |
| --- | --- | --- |
| 🟧 **Simulator Audit** | F06.1 | Inventory the Simulator game-scope tree and related debugger use/tests, producing a branch-coverage map for the required dump. |
| ⬜ **Architecture** | F06.2 | Define and obtain approval for the game-scope-dump request/acknowledgement and flattened Engine flow-trace event format. |
| 🟩 **Contracts** | F06.3 | Define the approved additive dump and development-capability contract in `Storyboard.Contracts`. |
| 🟦 **Host / Engine** | F06.4 | Implement Engine/Host manual on-demand complete semantic game-scope dump emission into Engine flow trace with production rejection. |
| 🟪 **WebPortal** | F06.5 | Implement Portal `Capture Game Scope Dump` request/status UX without browser tree retrieval or mutation controls. |
| 🟧 **Simulator Audit** | F06.6 | Verify dump branch coverage, manual-request behavior, and the deferred mutation/teleport dispositions against the Simulator. |

### F07: Parity Evidence, Test Migration, And Readiness Go/No-Go

| Area | Pass | One-sentence outcome |
| --- | --- | --- |
| 🟩 **Contracts** | F07.1 | Review contract compatibility and confirm the Stage 06 retirement ledger remains skipped or has explicit approved reopening evidence. |
| 🟨 **Designer** | F07.2 | Run Designer fresh-project launch and lifecycle regression against the completed development workflow. |
| 🟦 **Host / Engine** | F07.3 | Run Engine/Host full regression, fixture, capability, and production-negative security validation. |
| 🟪 **WebPortal** | F07.4 | Run Portal build, unit, contract, visual, and browser-host integration regression. |
| 🟧 **Simulator Audit** | F07.5 | Finalize Simulator test/fixture disposition and verify the no-new-WPF-renderer freeze. |
| ⬜ **Integration** | F07.6 | Execute and record end-to-end manual acceptance across the completed replacement workflow. |
| 🟥 **Closeout** | F07.7 | Reconcile all ledgers and parity rows, record Go/No-Go, and define the scope of any follow-on removal/governance work. |

### F08: Designer Legacy-Simulator Launch Retirement

| Area | Pass | One-sentence outcome |
| --- | --- | --- |
| ⬜ **Integration** | F08.1 | Confirm the F07 `Go`, the exact Designer launch artifacts to remove, and the release/rollback posture for the cutover. |
| 🟨 **Designer** | F08.2 | Remove the Designer's legacy Simulator command, UI/settings path, process orchestration, and launch-specific tests. |
| 🟨 **Designer** | F08.3 | Update Designer help, smoke tests, and workflow language so the GameHost/WebPortal path is the sole supported development launch. |
| 🟧 **Simulator Audit** | F08.4 | Prove no Designer path invokes the old Simulator and record remaining WPF application/code removal as separate follow-on scope. |

## F00: Decisions, Baseline, And Traceability

Outcome:

1. Convert open architectural questions into explicit decisions or bounded feature experiments.
2. Capture a reproducible baseline of current simulator and WebPortal capabilities.

Required area order:

1. Architecture review.
2. Contracts inventory only; no speculative additions.
3. Designer, engine, portal, and simulator read-only baseline evidence.
4. Create/update the F00 pass handoffs with baseline notes.

Required decisions:

1. Development identity stability policy.
2. Host-per-project versus reusable local development host.
3. System browser baseline versus any optional shell.
4. Recording-store location and retention direction.
5. Confirm the locked complete, manually requested Engine flow-trace game-scope dump boundary and defer mutation.
6. Confirm the locked WebPortal client-trace capture boundary, enumerate its client sources, and approve the shared trace-correlation envelope for separate-file alignment.

Exit criteria:

1. Every decision is locked or assigned to a named feature experiment.
2. Parity matrix rows have baseline statuses and evidence locations.
3. No implementation contract is added without a selected downstream feature.

## F01: Ad Hoc Development Launch And Automatic Bootstrap

Outcome:

1. From a fresh unregistered Designer project, one action exports the project, starts or reuses an isolated development GameHost, opens WebPortal, and attaches to the intended new/reset session without touching the production catalog.
2. The existing Designer WPF Simulator launch path remains available and behaviorally unchanged through F01-F07 as the transition fallback.

### F01 Transition Compatibility Rule

During F01-F07, Designer exposes two distinct supported launch paths:

| Designer action | Destination | F01-F07 rule |
| --- | --- | --- |
| New development launch action | Current project export → GameHost → WebPortal | Add and validate this path in F01. |
| Existing Simulator launch action | Current WPF Simulator workflow | Preserve it unchanged as the transition fallback. Do not rename, redirect, disable, or remove it. |

Only post-Go F08 may remove the existing Simulator launch action. F01 must include focused evidence that both actions remain independently usable.

### F01 Ordered Area-Pass Execution Matrix

The first wave is deliberately area-first: complete the known Host/Engine foundation, then Designer, then WebPortal, without bouncing back for anticipated tweaks. Each owner pass may iterate on defects discovered during its own implementation and focused validation. `F01.4` is the integration checkpoint; only unresolved owner-specific gaps after that checkpoint open separate remediation passes F01.5–F01.7. `F01.8` is the final acceptance gate.

| Order | Pass | Session owner / ledger | Authorized product area | Prerequisite | Required handoff output | Next pass |
| --- | --- | --- | --- | --- | --- |
| 1 | F01.1 | 🟦 **Host / Engine** / 03 | 🟦 `StoryBoard.GameEngine` host/runtime/test projects | Existing one-entry registration JSON fixture | Complete known Host/Engine registration, readiness, lifecycle, port, and production-isolation foundation with focused validation | F01.2 |
| 2 | F01.2 | 🟨 **Designer** / 02 | 🟨 `StoryBoard.Designer` app/test/smoke projects | F01.1 host semantics | Complete known Designer launch/process/browser/failure/retry/cleanup behavior and separately prove the preserved legacy Simulator action | F01.3 |
| 3 | F01.3 | 🟪 **WebPortal** / 04 | 🟪 `StoryBoard.WebPortal` app/test projects | F01.1 host behavior; F01.2 launch result | Complete automatic active-game/session attachment using existing host flow, or document the first genuine interface gap | F01.4 |
| 4 | F01.4 | ⬜ **Integration** / 07 | ⬜ No product edits; all three areas are test subjects | F01.1–F01.3 | Fresh-project vertical test report with each gap assigned to one owner: Host/Engine, Designer, Portal, or audit | Needed F01.5–F01.7, then F01.8 |
| 5a | F01.5 | 🟦 **Host / Engine** / 03 | 🟦 `StoryBoard.GameEngine` host/runtime/test projects | Unresolved F01.4 Host/Engine gaps only | Complete assigned remediation, or record that earlier inline remediation absorbed the pass | F01.8 |
| 5b | F01.6 | 🟨 **Designer** / 02 | 🟨 `StoryBoard.Designer` app/test/smoke projects | Unresolved F01.4 Designer gaps only | Complete assigned remediation, or record that earlier inline remediation absorbed the pass | F01.8 |
| 5c | F01.7 | 🟪 **WebPortal** / 04 | 🟪 `StoryBoard.WebPortal` app/test projects | Unresolved F01.4 Portal gaps only | Complete assigned remediation, or record that earlier inline remediation absorbed the pass | F01.8 |
| 6 | F01.8 | ⬜ **Integration** / 07 | ⬜ No product edits except approved narrow test stabilization | F01.1–F01.7 required remediations | Final vertical acceptance results, with no hidden legacy-launch regression | F01.9 |
| 7 | F01.9 | 🟧 **Simulator Audit** / 05 | 🟧 Simulator audit/test/fixture areas only | F01.8 | Startup/reset/duplicate-launch parity disposition and fixture/test ownership record | Feature closure review |

Recommended vertical sequence:

The ordered matrix above is the authoritative F01 session sequence. Each implementation pass is single-area-owned; integration and audit passes are evidence-only unless a boundary-approved test stabilization is required.

Affected ledgers:

1. `02_DESIGNER`
2. `03_GAMEENGINE`
3. `04_HOST_WEBPORTAL`
4. `05_SIMULATOR_AUDIT`
5. `07_CLOSEOUT` for evidence references only

Feature exit criteria:

1. No durable catalog mutation.
2. Discovery/session/assets share one development identity.
3. WebPortal arrives at the correct active session without manual discovery.
4. Host and client version mismatch fails clearly.
5. Lifecycle and production-isolation tests pass.
6. The existing Designer Simulator launch workflow remains available and covered by its existing focused smoke behavior.
7. The F01 handoff identifies the exact two Designer actions and their separate test evidence, so F08 has an unambiguous cutover target.

## F02: First-Class Diagnostics And Trace Workflow

Outcome:

1. Dev Tools provides a first-level Diagnostics workspace where trace scope, start/stop, console show/hide, save, clear, capture status, and truncation are direct and coherent.
2. The console captures the required WebPortal client, renderer, audio, orchestration, and client-observed transport/session activity without becoming a backend-log aggregation surface; its structured export preserves approved correlation-envelope fields for separate-file alignment.

Recommended vertical sequence:

1. **Portal UX prototype/test pass:** establish the state model and direct workflow without changing host contracts unnecessarily.
2. **Contracts pass:** define and approve the versioned shared trace-correlation envelope in `Storyboard.Contracts`; it standardizes metadata only and does not create a log-aggregation service.
3. **Portal source-inventory pass:** identify the client-side sources and categories required for useful WebPortal troubleshooting; do not add backend log-collection contracts.
4. **Portal integration pass:** connect named profiles to client capture scopes and display filters, and emit the approved correlation-envelope fields available to the Portal.
5. **Portal export pass:** full-buffer text/NDJSON save, filtered-save distinction, capture metadata, and truncation disclosure.
6. **Simulator-audit pass:** compare the approved client-side troubleshooting workflow, console/archive behavior, and relevant Portal-visible sources.
7. **Portal validation pass:** execute the streamlined workflow against an F01 development session and prove no layout-slot configuration is required; manually align representative Portal, Host, and Engine trace records by their common fields.

Affected ledgers:

1. `01_CONTRACT_RUNTIME`
2. `04_HOST_WEBPORTAL`
3. `05_SIMULATOR_AUDIT`
4. `07_CLOSEOUT` for evidence references only

Feature exit criteria:

1. Trace capture, console visibility, docking, filtering, and saving are independent.
2. Routine tracing requires no layout-slot edits and no closing of Dev Tools.
3. Client capture and display filtering are visibly distinct.
4. Hiding the console does not stop capture; stopping does not clear; default save exports the complete buffer.
5. Capture boundaries and truncation are visible and exported.
6. All P0 diagnostics rows in the main plan are satisfied.

## F03: Presentation And Audio Isolation Controls

Outcome:

1. A developer can narrow visual/audio presentation without changing authoritative runtime state or losing cue diagnostics.

Recommended vertical sequence:

1. **Engine semantics review:** confirm cues/state remain authoritative and no host suppression contract is required.
2. **Portal controller pass A:** master visuals plus movement, appearance, and disappearance switches.
3. **Portal controller pass B:** room transitions, HUD/text presentation, all-audio, SFX, and ambient controls.
4. **Portal lifecycle test pass:** disable mid-effect, re-enable, object removal, room transition, session reset, renderer disposal.
5. **Simulator-audit pass:** compare current WPF switches and explicitly disposition any unmatched behavior.
6. **F01-hosted acceptance pass:** troubleshoot one behavior with each family isolated.

Affected ledgers:

1. `03_GAMEENGINE` for semantics review only unless a genuine runtime gap is found
2. `04_HOST_WEBPORTAL`
3. `05_SIMULATOR_AUDIT`
4. `07_CLOSEOUT` for evidence references only

Feature exit criteria:

1. Each approved family can be independently suppressed.
2. Runtime state and diagnostic delivery continue while presentation is suppressed.
3. Active effect resources clean up deterministically.
4. Current switch state is included in diagnostic export metadata.

## F04: Authoritative Recording And Playback

Outcome:

1. GameEngine/GameHost owns per-session recording and playback; WebPortal controls it; existing approved recordings remain usable.

### Mandatory Interface-Design Approval Gate

No shared contract, GameHost/GameEngine implementation, Designer integration, or WebPortal recording-library implementation is authorized until an implementation-level `Recording And Playback Interface Specification` is reviewed and explicitly approved. The approved specification is then expressed as the authoritative, versioned, backward-compatible recording/playback contract in `Storyboard.Contracts`; no consumer may invent, duplicate, or use a replacement local interface shape.

The specification must define:

1. Capability advertisement and development-mode authorization.
2. Project-scoped recording-library discovery without exposing raw host paths.
3. Exact request/result shapes for list, filter, details, validation, start/stop recording, project save/promote, import/export, and refresh.
4. Recording identity, metadata, schema/version compatibility, project-library versus scratch disposition, and atomic write/error behavior.
5. Playback-run lifecycle, ownership, concurrency rules, state/version tokens, and all terminal/cancellation states.
6. Exact manual-step protocol: next-step preview, `Advance` request validation, normal command-path execution, returned outcome/diagnostics, and stale/double-advance handling.
7. Timed-playback controls: start, pause, resume, speed, stop, failure, and branch/continue semantics.
8. Pagination, payload size limits, diagnostics/correlation fields, and error codes suitable for Portal UX.
9. Legacy `.sbe.sim.json` import/compatibility behavior and regression-fixture ownership.
10. Security, production-negative behavior, and browser upload/download boundaries.

Approval record requirements:

1. The reviewed specification has a version and a decision-record reference.
2. It includes request/response examples, state-transition diagrams, and representative success, failure, cancellation, and concurrency scenarios.
3. Architecture approval is recorded in the F04 entries of the Contracts, GameEngine, Host/WebPortal, Simulator Audit, and Closeout ledgers.
4. Any contract change after approval requires a recorded additive-change review before implementation.

Recommended vertical sequence:

1. **Interface-design pass:** produce and obtain approval for the mandatory Recording And Playback Interface Specification; no consumer implementation occurs in this pass.
2. **Contracts pass A:** express the approved interface exclusively in `Storyboard.Contracts` as the canonical, versioned, backward-compatible recording schema, capability, list/status/control/result shapes, and legacy compatibility rules; validate generation/compatibility before any consumer work.
3. **Engine pass A:** record accepted commands, clarification answers, correlation, timing, outcomes, diagnostics, and movement telemetry per session.
4. **Host/storage pass:** recording list/store/import/export and development security.
5. **Portal pass A:** start/stop/list/select/status/import/export controls.
6. **Engine pass B:** timed playback, speed, pause/stop, step, cancellation, and normal command-path execution.
7. **Portal pass B:** playback controls and live progress/failure diagnostics.
8. **Designer pass:** apply approved optional startup replay and speed through the approved recording/playback host interface after the development session is ready.
9. **Simulator-audit pass:** run legacy fixtures, continuation/branch behavior, and replay-speed semantics through the new owner.
10. **Contracts pass B only if required:** approve and add only backward-compatible contract corrections found by the vertical slice before affected consumer changes; no retirement.

Affected ledgers:

1. `01_CONTRACT_RUNTIME`
2. `02_DESIGNER`
3. `03_GAMEENGINE`
4. `04_HOST_WEBPORTAL`
5. `05_SIMULATOR_AUDIT`
6. `07_CLOSEOUT` for evidence references only

Feature exit criteria:

1. Recording/playback is session-isolated and host-authoritative.
2. Playback uses the normal command/session/delta path.
3. Timed, stepped, paused/stopped, speed-adjusted, and failure behavior is deterministic.
4. Approved `.sbe.sim.json` fixtures import and retain their required semantics.
5. Browser filesystem limitations do not block normal list/select/play workflows.

## F05: Save/Load State And Latest-Export Reinitialize

Outcome:

1. WebPortal can capture/download and upload/restore session state, and the development launch path can reset to the latest Designer export deterministically.

Recommended vertical sequence:

1. **Engine/Host review pass:** validate existing persistence APIs and identify only genuine gaps.
2. **Portal pass:** capture/download, upload/validate/load, resync, and clear success/failure UX.
3. **Designer/launcher pass:** re-export/relaunch or reset semantics against the latest runtime export.
4. **Engine/Host correction pass:** address base-game identity/locator and reset issues found end to end.
5. **Simulator-audit pass:** compare WPF save/load and reinitialize behavior.

Affected ledgers:

1. `02_DESIGNER`
2. `03_GAMEENGINE`
3. `04_HOST_WEBPORTAL`
4. `05_SIMULATOR_AUDIT`
5. `07_CLOSEOUT` for evidence references only

Feature exit criteria:

1. Capture/download and upload/restore work without arbitrary host filesystem exposure.
2. Restore forces a correct baseline resync.
3. Reinitialize demonstrably uses the newest exported content.
4. Base-game mismatch fails safely with actionable diagnostics.

## F06: Game-Scope Dump And Debug-Command Disposition

Outcome:

1. WebPortal provides a development-capability-gated, manually requested command that produces a read-only, complete authoritative semantic game-scope dump in the Engine flow trace.
2. Every remaining thick-simulator debugger capability has an explicit `Implement`, `Replace`, or `Retire` decision; debug teleport is already deferred by decision.

Recommended vertical sequence:

1. **Simulator-audit pass A:** inventory the existing runtime-state tree, variable mutation, move-player, and related usage/tests; produce a branch-coverage map for the dump.
2. **Dump-contract design/approval pass:** define the request/acknowledgement and complete flattened semantic dump-event format, including completeness, identity, references, payload, and error rules; obtain explicit approval.
3. **Contracts pass:** add the approved additive dump/capability shapes in `Storyboard.Contracts` and validate compatibility before consumer implementation.
4. **Engine/Host pass:** materialize the complete immutable semantic dump on request, emit it into the Engine flow trace, advertise the development capability, and reject it in production mode.
5. **Portal pass:** provide `Capture Game Scope Dump`, report dump identity/time/completion/failure, and direct the developer to the Engine flow trace; do not fetch/render the tree payload and do not expose mutation controls.
6. **Simulator-audit pass B:** verify scope-branch coverage, manual-request behavior, and record the already-deferred mutation/teleport dispositions.

Affected ledgers:

1. `05_SIMULATOR_AUDIT` first
2. `01_CONTRACT_RUNTIME` if required
3. `03_GAMEENGINE`
4. `04_HOST_WEBPORTAL`
5. `07_CLOSEOUT` for evidence references only

Feature exit criteria:

1. The full semantic game-scope tree is available through an explicitly manual, immutable, read-only Engine flow-trace dump requested from WebPortal.
2. The dump declares whether it is complete; no scope branch is silently omitted or confused with arbitrary process memory.
3. Mutation is absent from the readiness interface and remains a separately approved future capability.
4. No remaining debugger capability disappears silently; each has an explicit disposition.
5. Production-mode negative tests pass.

## F07: Parity Evidence, Test Migration, And Readiness Go/No-Go

Outcome:

1. Produce evidence that the WebPortal development experience can replace the WPF simulator and decide whether a separate removal plan may start.

Required area order:

1. Contracts compatibility and Stage 06 skip/reopen review.
2. Designer launch regression.
3. Engine/Host full regression and security-negative tests.
4. WebPortal unit/build/contract/visual regression.
5. Simulator test disposition and no-new-renderer freeze audit.
6. End-to-end manual acceptance.
7. Stage 07 closeout and Go/No-Go.

Affected ledgers:

1. All area profiles and all required pass handoffs.
2. `07_CLOSEOUT` becomes the authoritative final record.

Feature exit criteria:

1. No P0 gap remains.
2. Every P1 gap is replaced/present or has an approved named follow-up.
3. All durable semantic fixtures have an owner outside the soon-to-be-retired WPF implementation.
4. The standard stage catalog and active-plan simulator-parity assumptions are listed for the follow-on governance update.
5. A formal `Go` or `No-Go` is recorded with evidence.

## F08: Designer Legacy-Simulator Launch Retirement

Outcome:

1. After an evidence-backed F07 `Go`, Designer no longer offers or invokes the WPF Simulator launch workflow; the development GameHost/WebPortal workflow becomes its sole supported run experience.
2. The WPF Simulator application/code remains outside this feature's deletion scope and is handled only by a separately authorized removal/governance workstream.

Required area order:

1. Confirm the F07 `Go`, complete parity evidence, release/rollback posture, and the exact legacy launch artifacts to remove.
2. Remove the Designer command, menu/dialog/settings path, process orchestration, and tests that specifically launch the WPF Simulator.
3. Update Designer help text, smoke tests, and transition documentation so `Run`/development launch means the GameHost/WebPortal workflow.
4. Audit that no Designer path can launch the old Simulator and that the new workflow remains available for fresh and existing projects.
5. Record the completed launch retirement and the remaining separate WPF application/code-removal scope.

Feature exit criteria:

1. The Designer has no supported legacy Simulator launch action or hidden fallback invocation.
2. Every Designer run/development command uses the approved development GameHost/WebPortal launch path.
3. Focused Designer regression and smoke tests pass for fresh and existing projects.
4. The Simulator audit records the launch-path retirement and identifies any remaining non-launch WPF code for the separate removal plan.
5. No WPF application/code deletion occurs as part of F08.

## Area Profile Feature Coverage

| Area profile | Expected feature passes |
| --- | --- |
| 01 Contracts/Shared | F00, F02, F04, F06, and F07 review; F01 uses the established runtime-registration JSON format without a new contract pass |
| 02 Designer | F00, F01, F04 startup replay, F05, F07, F08 launch retirement |
| 03 GameEngine/GameHost | F00, F01, F02, F03 semantics, F04, F05, F06, F07 |
| 04 Host/WebPortal | F00, F01, F02, F03, F04, F05, F06, F07 |
| 05 Simulator Audit | F00 through F07 as replacement evidence is produced, plus F08 launch-retirement audit |
| 06 Contract Retirement | F07 review only unless explicitly reopened |
| 07 Closeout | Evidence references during F01-F06, authoritative readiness decision in F07, and F08 launch-cutover record |

## Pass Handoff Template

Create a pass-specific file from `features/PASS_HANDOFF_TEMPLATE.md` when the roadmap schedules that pass. The handoff inherits one area profile and records one bounded execution outcome; it is never appended to an area profile.

## Roadmap Closeout Rule

1. Feature completion is recorded here; execution evidence is recorded in pass handoffs; boundary rules remain in area profiles.
2. A feature cannot be declared complete based on one repository's implementation alone.
3. An area profile has no completion state and is not a substitute for pass evidence.
4. Stage 07 cannot issue `Go` until this roadmap, the parity matrix, and every required pass handoff agree on status.
