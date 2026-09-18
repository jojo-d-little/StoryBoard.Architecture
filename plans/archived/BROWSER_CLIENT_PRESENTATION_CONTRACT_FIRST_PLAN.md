# Browser Client Presentation Contract-First Plan

## Goal
Stand up a browser-based end-user game client with polished UX by defining orchestration, state transitions, and top-level screen composition in external configuration contracts before implementation, and by enabling GameHost to serve the initial client page bundle as static content.

## Why This Plan Starts With Contracts
1. Human review of UI control logic should not require deep code inspection.
2. State/transition and layout responsibilities should be explicit and durable.
3. Rendering complexity (PixiJS and logical-to-physical scaling) should plug into a stable shell, not define it.

## Layer Ordering (Locked for Documentation and Design Discussion)
Use this order when documenting, reviewing, and implementing config contracts so dependencies are explained in build order.

1. Experience State Layer
   1. Defines which features are available in each experience state.
   2. Defines transition rules between experience states.
   3. Does not define layout geometry or form-factor-specific implementations.

2. Skeleton Layout Layer
   1. Defines skeletal UI regions and their geometry behavior.
   2. Defines min/max/preferred dimensions, collapse behavior, and reflow rules.
   3. Is reusable across form factors and composition profiles.

3. Experience Composition Layer
   1. Maps features to skeletal regions for a given experience state.
   2. Default composition is required; optional alternate compositions are allowed.
   3. Is not inherently form-factor-specific.

4. Form-Factor Feature Implementation Layer
   1. Maps each feature key to a concrete implementation key per form factor.
   2. This is the only inherently form-factor-specific layer.
   3. Does not define geometry or transition logic.

Resolver model summary:
1. Inputs: experience state, skeleton layout, form factor.
2. Optional input: composition profile.
3. Output: concrete rendered composition (which implementation is shown in which region).

## Primary Deliverables
1. Config contracts for state orchestration, scenes, sections, and layout profiles.
2. Validator and test harness for contract correctness.
3. Shell runtime that interprets contracts and drives top-level UI composition.
4. Session-active framework region with non-rendering placeholder behavior for lifecycle plumbing validation.
5. GameHost static-content bootstrap path for browser client files (HTML/CSS/JS/config), independent from in-game asset APIs.

## Browser Framework Recommendation (Initial)
1. Recommended default stack for v1 shell and workflow plumbing:
   1. React with TypeScript
   2. Vite build pipeline
   3. State-machine and orchestration interpreter in app code driven by external config contracts
2. Why this is the default recommendation:
   1. Mature ecosystem and strong long-term maintainability for browser app shell concerns.
   2. Excellent compatibility with canvas-heavy rendering integration while keeping shell UI decoupled.
   3. Fast local iteration and straightforward static-file output for GameHost static serving.
3. Why not Razor/Blazor as the default for this effort:
   1. Additional JS interop complexity for high-control rendering and browser-native ecosystem integrations.
   2. Larger risk of friction in modern front-end UX tooling for this specific client-facing shell initiative.
4. This remains a lock-off decision, but React plus TypeScript is the proposed baseline unless the lock review overturns it.

## Project Structure Recommendation (v1)
1. Start with a single new browser client project for authoring/build output (working name examples: `Storyboard.WebPortal`, `Storyboard.WebPresentation`, `Storyboard.WebInterface`).
2. Keep GameHost as a separate project responsible for:
   1. `/api/v1/*` host APIs
   2. static file serving for browser bootstrap artifacts
3. Keep shared runtime/domain contracts in shared projects; do not duplicate contract ownership in the web project.
4. This single-project model is the default for v1 because it reduces coordination overhead while the shell/state contracts stabilize.

### Split Triggers (When to Add More Web Projects)
1. Multiple independently deployed browser clients are required.
2. A reusable shared UI/component package is needed by multiple applications.
3. A standalone TypeScript SDK/client package is required for external consumers.
4. Build/test/release cadence divergence creates sustained delivery friction in one combined project.

### v1 Project Structure Exit Criteria
1. Browser client source and config contracts are owned in a single web project.
2. GameHost static serving integration works without introducing a second browser project.
3. Re-evaluation checkpoint occurs at Phase 3.5 hardening gate before any rendering-surface expansion.

## TypeScript and Vite Workflow Baseline
1. Development workflow (inner loop):
   1. Run Vite dev server for instant local browser iteration.
   2. Author in TypeScript with editor diagnostics and source maps.
   3. Use hot module replacement for rapid shell/state UX iteration.
2. Type safety workflow (quality gate):
   1. Run explicit type-check command as a separate gate from dev transpile speed.
   2. Treat type-check failures as blocking for merge.
3. Production packaging workflow:
   1. Run Vite production build to emit optimized static bundle files.
   2. Publish/copy emitted files into GameHost static client root according to Phase 0.5 contract.
4. Runtime separation workflow:
   1. Browser bootstrap files come from GameHost static serving.
   2. Runtime gameplay/session data continues through `/api/v1/*` host APIs.
5. v1 TypeScript style constraints for maintainability:
   1. Prefer simple explicit interfaces for DTOs and config models.
   2. Avoid advanced type-system metaprogramming patterns in v1.
   3. Keep strict mode enabled, but prioritize readability over clever type tricks.

## Immediate Execution Focus (Intentionally Pre-Rendering)
1. Complete all core plumbing from browser bootstrap through authentication, session discovery, start session, join session, active-session shell, and leave session.
2. Reach a stable and comfortable operational baseline for shell/state/transport behavior before introducing room rendering complexity.
3. Intentionally defer gameplay room rendering surface and PixiJS integration until core workflow contracts and framework management are validated.

## Cross-Cutting Host Requirement: Static Client Bootstrap Serving
1. GameHost must support serving static browser client files from a configured web root/folder so a browser can perform initial load.
2. GameHost must remain non-opinionated about client implementation details beyond static-file hosting and optional fallback routing.
3. This static client-file serving is separate from game runtime asset APIs (images/sounds/etc) and must not replace or blur those interfaces.
4. Initial scope is local/trusted development and deployment packaging support; auth hardening for public internet exposure remains a separate concern.
5. Hosting must support cache/version strategy suitable for deterministic updates (for example fingerprinted static bundle files).
6. Static serving behavior is intentionally dumb pass-through only: accept request, map to configured static mount file path, return file bytes or not-found.
7. GameHost static serving must not include SPA-aware route fallback logic, client-router awareness, or any client-specific route rewriting.

## Plan Phases

## Phase 0: Contract Agreement and Realistic Drafts (First Step)
1. Draft realistic v1 configuration files and review them as a contract package before app implementation.
2. Record strict boundaries for what is and is not allowed in each file.
3. Lock naming conventions for states, scenes, sections, and layout profiles.
4. Build schema validation and startup validation as merge gates.

### Contract Package v1
1. `Storyboard.WebPortal/config/orchestration/experience-state-featuremap.v1.json`
2. `Storyboard.WebPortal/config/orchestration/skeleton-layouts.v1.json`
3. `Storyboard.WebPortal/config/orchestration/experience-state-compositions.v1.json`
4. `Storyboard.WebPortal/config/orchestration/form-factor-feature-implementations.v1.json`
5. `Storyboard.WebPortal/config/orchestration/feature-catalog.v1.json`
6. `Storyboard.WebPortal/config/orchestration/ui-slots.v1.json`
7. `Storyboard.WebPortal/config/orchestration/diagnostics-policy.v1.json`
8. `Storyboard.WebPortal/config/orchestration/theme-contract.v1.json`

### What Each File Owns
1. `experience-state-featuremap.v1.json`
   1. Owns: experience state catalog, initial experience state, transition matrix, event bindings, guardRef/actionRef references.
   2. Does not own: layout geometry, slot placement, form-factor implementation mapping.
2. `skeleton-layouts.v1.json`
   1. Owns: skeletal arrangement, breakpoints, slot placement, size constraints, collapse order.
   2. Does not own: state transitions, feature availability, side effects, data fetching behavior.
3. `experience-state-compositions.v1.json`
   1. Owns: experience-state-to-composition mapping and feature-to-slot assignments with slot modes.
   2. Does not own: geometry rules or form-factor implementation choice.
4. `form-factor-feature-implementations.v1.json`
   1. Owns: feature-to-implementation mapping by form factor.
   2. Does not own: geometry, transitions, or state definitions.
5. `feature-catalog.v1.json`
   1. Owns: canonical feature registry and feature metadata.
   2. Does not own: where features render or how they are implemented by form factor.
6. `ui-slots.v1.json`
   1. Owns: canonical UI slot registry and allowed slot mode vocabulary.
   2. Does not own: experience-state-specific composition assignments.
7. `diagnostics-policy.v1.json`
   1. Owns: global diagnostics toggle policy, per-state defaults, allowed channels.
   2. Does not own: business workflow state transitions.
8. `theme-contract.v1.json`
   1. Owns: producer-facing style tokens (chrome accents, border colors, font family choices, base spacing/radius hints).
   2. Does not own: content text, gameplay behavior, executable logic.

### Contract Rules
1. Config is declarative only. No expression language and no inline executable logic.
2. `guardRef` and `actionRef` are named references to code-owned registries.
3. Unknown experience-state/feature/slot/skeleton/form-factor keys fail validation.
4. Duplicate transition handlers for the same `(state,event)` require explicit priority and must pass determinism checks.
5. Every experience state must resolve to exactly one default composition and one skeleton layout context.
6. Transitional async operations (sign-in, discovery, start/join/leave, reconnect) are modeled as operation status metadata, not as top-level macro states.
7. Feature availability, composition, skeleton geometry, and form-factor implementation mapping are separate layers and must not leak responsibilities across files.
8. Collapsed sections must fully release layout real estate and allow configured expansion/reflow of remaining visible sections.

### Macro-State and Operation Model (Locked)
1. v1 top-level macro states are:
   1. `Bootstrapping`
   2. `SignedOut`
   3. `SignedIn`
   4. `SessionActive`
   5. `SessionInterrupted`
2. v1 operation status keys are:
   1. `None`
   2. `SigningIn`
   3. `DiscoveringGames`
   4. `StartingSession`
   5. `JoiningSession`
   6. `LeavingSession`
   7. `Reconnecting`
3. v1 operation phase keys are:
   1. `Idle`
   2. `Running`
   3. `Failed`
4. `SignedInNoSession` naming is retired in favor of `SignedIn`.

### Phase 0 Exit Criteria
1. Contract package files exist with realistic starter definitions.
2. Team signoff confirms ownership boundaries for all files.
3. Validation gates run in CI and block invalid changes.
4. Reviewer artifact can be generated from contracts (state graph plus state-to-section activity table).

## Phase 0.5: GameHost Static Serving Contract Agreement
1. Define and review a host static-serving contract that is intentionally minimal and non-opinionated.
2. Define configured locations for browser client files and orchestration config files intended for browser bootstrap.
3. Define routing behavior:
   1. API routes stay under `/api/v1/*`.
   2. Static client files are served from configured dedicated path prefixes mapped to configured local/static folders.
   3. No SPA fallback routes are implemented in GameHost static serving.
4. Define packaging expectations so host publish output can include static client files when desired.
5. Define local development integration expectations:
   1. Vite dev server hosts local UI iteration.
   2. API calls target GameHost endpoints.
   3. Production-like validation uses built static files served by GameHost.
6. Define static mount configuration shape that supports one or more client entries (for example route prefix `/client` mapped to a configured folder path).

### Phase 0.5 Exit Criteria
1. Agreed host/static boundary document exists in plan notes.
2. Agreed URL and folder conventions for static client bootstrap files are locked.
3. Agreed separation statement from in-game asset APIs is locked.

## Phase 1: Shell Runtime and State Interpreter
1. Build persistent app shell with global regions:
   1. top bar
   2. main scene outlet
   3. optional side region
   4. diagnostics drawer
   5. modal/toast overlays
2. Implement state interpreter that executes contract transitions and scene/layout resolution.
3. Enforce section activation modes (`visible`, `hidden`, `collapsed`, `disabled`, `readonly`) from config.
4. Implement resolver override inputs for development testing:
   1. form factor override
   2. composition profile override
   3. optional skeleton layout override

## Phase 1.25: Layout Lab Overrides (Laptop-Friendly Form-Factor Testing)
1. Add a development-only Layout Lab panel in the shell:
   1. effective form factor selector (`auto`, `desktop`, `mobilePortrait`, `mobileLandscape`)
   2. composition profile selector
   3. optional skeleton layout selector
   4. reset-to-defaults action
2. Add URL override support for reproducible scenarios:
   1. `ff` (form factor)
   2. `cp` (composition profile)
   3. `sk` (skeleton layout, optional)
3. Lock deterministic override precedence:
   1. URL override
   2. Layout Lab selection
   3. local persisted development preference
   4. config defaults from `experience-state-featuremap.v1.json`
4. Show active effective values in diagnostics/dev UI so testers can confirm actual runtime resolution.
5. Keep overrides disabled in production mode unless explicitly enabled by a deployment flag.
6. Primary objective: mobile form-factor behavior must be testable on a standard Windows laptop without requiring physical mobile hardware.

## Phase 1.5: GameHost Static Serving Implementation
1. Add static-file middleware and configuration in GameHost startup with explicit route-prefix-to-folder mount settings.
2. Ensure API endpoint routes and static routes do not conflict.
3. Add startup diagnostics indicating static root presence and serving mode.
4. Add smoke tests:
   1. host serves index/bootstrap page
   2. host serves referenced static bundle files
   3. API endpoints remain reachable while static serving is enabled
5. Add one end-to-end packaging check:
   1. Vite production output is host-loadable through the agreed static route.
6. Add multi-mount smoke validation:
   1. distinct route prefixes can map to distinct client folders
   2. route prefixes remain isolated from `/api/v1/*`
   3. unknown static route prefixes return expected not-found behavior

## Phase 2: Authentication and Game Discovery Flow
1. Implement signed-out and signed-in workflows against HTTP host APIs.
2. Implement explicit authentication state transitions:
   1. sign-in requested
   2. sign-in succeeded
   3. sign-in failed
   4. sign-out requested
3. Implement game discovery list retrieval and refresh behavior with state-aware busy/error handling.
4. Keep diagnostics globally available in all states.

## Phase 3: Session Lifecycle Plumbing Without Room Rendering
1. Implement session lifecycle workflows through contract-driven events:
   1. start session
   2. join session
   3. attach/re-attach by session id when explicitly requested
   4. leave session and return to signed-in
2. Implement session-active scene with placeholder play-surface section and non-canvas status widgets.
3. Wire session status, lifecycle controls, and diagnostics to real transport.
4. Validate behavior for interrupted session and basic reconnect path (without gameplay render state recovery concerns).

## Phase 3.5: Core Plumbing Hardening Gate
1. Run focused UX-state and transport reliability pass before any PixiJS work begins.
2. Validate that all key flows are stable:
   1. bootstrap to sign-in
   2. sign-in to discovery
   3. discovery to start/join session
   4. session-active to leave session
3. Lock v1 framework contracts unless critical defects require targeted adjustments.

## Phase 4: Deferred Rendering Surface Track (Do Not Start Until Phase 3.5 Passes)
1. Integrate PixiJS inside the session play-surface section only.
2. Implement logical resolution mapping (for example 800x600 logical space) to physical viewport.
3. Preserve aspect ratio by default and support responsive resizing within layout constraints.
4. Map pointer input from physical coordinates to logical coordinates consistently.

## Phase 5: Theme Contract Adoption and UX Polish
1. Apply producer-defined presentation tokens from `theme-contract.v1.json` with safe defaults.
2. Validate fallback behavior for missing/invalid tokens.
3. Tune transitions, focus behavior, and accessibility for end-user quality.

## Validation and Governance
1. Add JSON schema validation for each config file.
2. Add graph validation:
   1. reachable states
   2. no orphan states
   3. deterministic transition resolution
3. Add golden workflow tests:
   1. signed-out to signed-in
   2. signed-in to session-active
   3. session-active to signed-in
4. Add rendering-shell tests for section activation and layout resolution.
5. Add pre-render lifecycle flow tests for:
   1. sign-in success and failure handling
   2. discovery refresh and selection handling
   3. start session and join session contract paths
   4. leave session and return-to-discovery behavior
6. Add toolchain validation gates for browser client:
   1. lint gate
   2. TypeScript type-check gate
   3. Vite production build gate
   4. static-host smoke verification gate via GameHost
7. Add resolver override validation tests:
   1. valid form-factor override changes effective resolved implementation mapping
   2. valid composition override changes effective slot assignment plan
   3. invalid override keys fail fast (or follow explicit fallback policy)
   4. state transition behavior remains deterministic while overrides are active
8. Add layout matrix smoke tests for laptop-driven QA:
   1. desktop plus default composition
   2. mobilePortrait plus default composition
   3. mobileLandscape plus default composition
   4. at least one alternate composition profile per form factor when present

## Design Lock-Off Questions (Resolve Before or During Phases 0-3)
1. (Locked) What exact v1 state list and canonical event names will be locked for implementation?
2. (Locked) Which transitions are mandatory versus optional in v1, and which transitions are explicitly disallowed?
3. (Locked) What is the canonical section-key registry for the persistent shell?
4. (Locked) Which section activation modes are permitted in v1 (`visible`, `hidden`, `collapsed`, `disabled`, `readonly`), and do any require state-specific constraints?
5. (Locked) What is the minimum v1 scene set and which states map to each scene?
6. (Locked) What is the minimum v1 layout-profile set (desktop and narrow) needed before implementation begins?
7. (Locked) What is the route strategy for hosted client files (`/` root-hosted or dedicated prefix such as `/client`)?
8. (Locked) Is SPA fallback enabled in v1, and if yes, what fallback path rules avoid shadowing `/api/v1/*` endpoints?
9. (Locked) Are orchestration config files served as separate static JSON resources, bundled into JS at build time, or supported in both modes?
10. (Locked) What is the authoritative source for authentication status in browser state evaluation (transport response, cached token, or both)?
11. (Locked) What is the session discovery freshness model (manual refresh only, interval refresh, or event-triggered refresh) for v1?
12. (Locked) What fields are required in discovery items and session summaries for v1 UX decisions?
13. (Locked) What exact start-session and join-session workflows are required in v1, including explicit user intent points?
14. (Locked) What interrupted-session behavior is required in v1 (auto-reconnect attempt count, user prompts, and timeout policy)?
15. (Locked) What diagnostics channels are always available, which are state-scoped or capability-scoped, and what is the v1 show/hide interaction policy?
16. (Locked) What producer-theme tokens are in initial scope and what fallback behavior is mandatory for missing/invalid values?
17. (Locked) What caching/version policy is required for hosted static client files and externally served config files?
18. (Locked) What pre-render hardening gate criteria must pass before the plan may start deferred Phase 4 rendering work?
19. (Locked) Which browser framework stack is locked for v1 shell implementation (React plus TypeScript default recommendation, or an approved alternative)?
20. (Locked) What is the minimum accepted TypeScript strictness profile for v1, and are any temporary relaxations allowed?
21. (Locked) What are the required toolchain gates in CI for browser client changes (lint, type-check, build, host static-smoke)?
22. (Locked) What is the locked v1 browser project name and location in the solution (`Storyboard.WebPortal` default recommendation unless another name is approved)?

## Design Lock Decisions
1. Lock 1 (2026-08-26): Use macro-state plus operation-status modeling to avoid transitional state explosion.
   1. Accepted.
   2. Top-level state machine uses only macro UI modes.
   3. Transitional async workflows are represented as operation status and phase metadata.
   4. `SignedInNoSession` is replaced with `SignedIn`.
2. Lock 2 (2026-08-26): Lock v1 macro-state transitions and transition boundaries.
   1. Accepted mandatory macro-state transitions:
      1. `Bootstrapping` -> `SignedOut`
      2. `SignedOut` -> `SignedIn` (after successful sign-in operation)
      3. `SignedIn` -> `SessionActive` (after successful start or join operation)
      4. `SessionActive` -> `SignedIn` (after successful leave operation)
      5. `SessionActive` -> `SessionInterrupted` (interruption detected)
      6. `SessionInterrupted` -> `SessionActive` (successful reconnect operation)
      7. `SessionInterrupted` -> `SignedIn` (reconnect failure or user-cancel)
   2. Accepted optional transitions:
      1. `SignedIn` -> `SignedOut` (explicit user sign-out)
      2. `SessionInterrupted` -> `SignedOut` (when auth or session invalidation requires re-auth)
      3. `Bootstrapping` -> `SignedIn` (only if explicit persisted-auth policy is approved)
   3. Explicitly disallowed transitions:
      1. `SignedOut` -> `SessionActive` direct jump
      2. `Bootstrapping` -> `SessionActive` direct jump
      3. `SessionActive` -> `SignedOut` direct jump without leave or invalidation flow
      4. any session-mutating operation while in `SignedOut`
3. Lock 3 (2026-08-26): Lock canonical persistent-shell section registry with explicit scene variants.
   1. Accepted canonical section-key registry:
      1. `appFrame`
      2. `topBar`
      3. `primarySurface`
      4. `secondaryPanel`
      5. `utilityPanel`
      6. `diagnosticsDrawer`
      7. `modalLayer`
      8. `toastLayer`
      9. `statusStrip`
   2. `secondaryPanel` and `utilityPanel` are structural slots only, not generic catch-all content sinks.
   3. Scene definitions must use explicit panel variant keys to name purpose by context (for example `discoveryListPanel`, `sessionToolsPanel`, `identitySummaryPanel`, `connectionHealthPanel`).
   4. If a use case requires parallel secondary surfaces that cannot be represented by variants, add a new explicit section key through lock review rather than overloading existing slots.
4. Lock 4 (2026-08-26): Lock v1 section activation modes with collapse-and-reflow semantics.
   1. Accepted v1 section activation modes:
      1. `visible`
      2. `hidden`
      3. `collapsed`
      4. `disabled`
      5. `readonly`
   2. Required constraints:
      1. `appFrame` and `primarySurface` cannot be `hidden` or `collapsed`.
      2. `modalLayer` and `toastLayer` remain infrastructure-mounted layers; they are not removed from DOM structure by state transitions.
      3. `diagnosticsDrawer` must remain globally available, but may be closed or collapsed by default in any macro state.
   3. Collapse and expansion behavior:
      1. When a section enters `collapsed`, it releases real estate completely.
      2. Layout profiles define deterministic reflow/expansion rules for remaining visible sections.
      3. `SessionActive` is allowed to collapse peripheral sections (for example `topBar`, `secondaryPanel`, `utilityPanel`, `statusStrip`) so `primarySurface` can expand to dominant use.
   4. Clarification boundary:
      1. This lock governs section activation and geometry behavior only.
      2. Specific content mapped into a section slot is defined by scene mappings, not by activation mode.
5. Lock 5 (2026-08-26): Lock minimum v1 scene set and macro-state mappings with shell-level diagnostics availability.
   1. Accepted minimum v1 scene set:
      1. `bootstrapScene`
      2. `authScene`
      3. `lobbyScene`
      4. `sessionScene`
      5. `sessionInterruptedScene`
   2. Accepted macro-state to scene mapping:
      1. `Bootstrapping` -> `bootstrapScene`
      2. `SignedOut` -> `authScene`
      3. `SignedIn` -> `lobbyScene`
      4. `SessionActive` -> `sessionScene`
      5. `SessionInterrupted` -> `sessionInterruptedScene`
   3. Accepted baseline scene content variants:
      1. `bootstrapScene`:
         1. `primarySurface` -> `bootstrapStatusSurface`
         2. `statusStrip` -> `startupStatusStrip`
      2. `authScene`:
         1. `primarySurface` -> `authSignInSurface`
         2. `secondaryPanel` -> `authHelpPanel` (optional)
      3. `lobbyScene`:
         1. `primarySurface` -> `gameDiscoverySurface`
         2. `secondaryPanel` -> `gameDetailsPanel`
         3. `utilityPanel` -> `identitySummaryPanel` (optional)
      4. `sessionScene`:
         1. `primarySurface` -> `sessionPlayFrameSurface` (placeholder before deferred rendering phase)
         2. `secondaryPanel` -> `sessionToolsPanel`
         3. `statusStrip` -> `sessionConnectionStatusStrip`
      5. `sessionInterruptedScene`:
         1. `primarySurface` -> `interruptionRecoverySurface`
         2. `secondaryPanel` -> `reconnectDetailsPanel` (optional)
         3. `topBar` -> `reconnectActionBar` variant
   4. Diagnostics availability position:
      1. Diagnostics console is positioned as an app-shell level fixture, not a per-scene bespoke requirement.
      2. All scenes inherit diagnostics availability through shell infrastructure.
      3. Exact v1 show/hide interaction model (for example hotkey, command palette, affordance visibility during immersive session mode) is intentionally deferred to Lock Question 15 and must not block core scene contract implementation.
6. Lock 6 (2026-08-26): Lock v1 layout profile family baseline, including mobile proof-of-concept orientation validation.
   1. Accepted v1 layout profile families:
      1. `desktopStandard`
      2. `desktopImmersiveSession`
      3. `mobileProofOfConcept`
   2. `mobileProofOfConcept` must define both orientation variants:
      1. `mobilePortrait`
      2. `mobileLandscape`
   3. Layering and ownership clarification:
      1. Section registry defines shell slots.
      2. Scene definitions define content variants assigned to slots.
      3. Layout profiles define physical geometry, breakpoints, collapse rules, and reflow behavior.
   4. Mobile feasibility validation intent:
      1. Implement and test both portrait and landscape variants early.
      2. Treat portrait viability as a hypothesis to prove or reject.
      3. If portrait fails defined usability criteria, formally retire portrait support from v1 and lock landscape-only mobile support.
   5. Baseline behavior expectations per family:
      1. `desktopStandard`: balanced shell with visible top/navigation/context sections.
      2. `desktopImmersiveSession`: primary-surface-dominant layout with peripheral collapse capability.
      3. `mobileProofOfConcept`: constrained layout emphasizing session clarity, deterministic collapse/reflow, and diagnostics availability through compact access patterns.
7. Lock 7 (2026-08-26): Lock GameHost static-client route strategy as configurable mount mappings.
   1. Route namespace boundary:
      1. API remains isolated under `/api/v1/*`.
      2. Static browser client files are served under dedicated configured route prefixes (for example `/client`).
   2. Configuration model:
      1. GameHost supports configured route-prefix-to-local-folder mappings.
      2. Initial v1 baseline requires at least one mapping.
      3. Model permits multiple mappings so more than one browser client can be hosted without entering API URL space.
   3. Routing behavior:
      1. Optional root redirect from `/` to a configured default client mount is allowed.
      2. Static-route fallback behavior is disabled in v1 by design.
      3. Unknown mount prefixes return not-found by default unless explicitly configured.
   4. Non-opinionation constraint:
      1. GameHost serves mounted static content generically and does not embed client-specific UX assumptions.
8. Lock 8 (2026-08-26): Lock GameHost static serving to dumb pass-through behavior with no SPA smarts.
   1. Accepted static-serving behavior:
      1. Request path is evaluated against configured static mount prefixes.
      2. On file hit, return file bytes.
      3. On file miss or unknown mount path, return not-found.
   2. Explicitly disallowed behaviors:
      1. SPA/history-route fallback to index files.
      2. client-router-aware path rewriting.
      3. request-time dynamic path synthesis based on UI semantics.
   3. Browser-client implication:
      1. v1 client routing must use file-safe patterns (for example hash-based routing) so deep navigation does not require host fallback behavior.
9. Lock 9 (2026-08-26): Lock orchestration config hosting model for v1.
   1. Accepted v1 default:
      1. Orchestration contracts are stored and served as separate static JSON files under the configured client mount.
      2. Client boot sequence loads and validates these JSON files explicitly.
   2. Optional future mode:
      1. Bundled-in-JS config packaging is allowed as a later optimization path.
      2. Any bundled mode must preserve schema validation and deterministic version identity.
   3. Rationale:
      1. Maximizes human reviewability of contract definitions.
      2. Aligns with dumb static file serving constraints in GameHost.
      3. Keeps contract updates independent from application code rebuild in v1 workflows.
10. Lock 10 (2026-08-26): Lock authentication authority model to host-verified identity outcomes.
   1. Authoritative source:
      1. Identity state transitions are driven by host API outcomes through `IHostIdentityManagementClient`.
      2. `Authenticate`, `GetCurrentPrincipal`, and `Logout` responses are the authority for `SignedIn` versus `SignedOut` decisions.
   2. Local cache role:
      1. Cached credential handle or auth hint data is advisory only.
      2. Client cache cannot independently assert authenticated state.
   3. Startup behavior:
      1. When cached credential material exists, client must validate with `GetCurrentPrincipal` before entering `SignedIn`.
      2. Validation failure or ambiguity resolves to `SignedOut`.
   4. Runtime behavior:
      1. Auth-expired or unauthorized host responses trigger state correction out of authenticated flows.
      2. `Logout` success or explicit credential invalidation transitions to `SignedOut` and clears local credential artifacts.
11. Lock 11 (2026-08-26): Lock session discovery freshness to server-authoritative refresh with no client-side session-list caching.
   1. Source-of-truth rule:
      1. Available session/discovery lists are always sourced from server responses.
      2. Client must not persist or reuse cached available-session lists as a data source.
   2. Required refresh points for v1:
      1. Refresh immediately when entering the signed-in discovery view after successful sign-in.
      2. Provide explicit user-controlled manual refresh action in discovery UX.
   3. Optional event-triggered refreshes:
      1. Additional event-triggered refresh points (after join/start/leave/reconnect) are allowed but not required for v1.
      2. Manual refresh remains the primary recovery and staleness-control mechanism.
   4. UX expectations:
      1. Show loading state during refresh operations.
      2. Show refresh failure state with retry action.
      3. Surface last-refresh timestamp or equivalent freshness cue.
12. Lock 12 (2026-08-26): Lock end-user display-first discovery/session metadata contract.
   1. UX display principle:
      1. End-user surfaces prioritize human-readable names, artwork, and context.
      2. Raw technical IDs and GUIDs are not shown in primary discovery and session-selection UI.
      3. IDs may be exposed only in diagnostics/developer tooling views.
   2. Required game-discovery display fields for v1:
      1. `gameDisplayName`
      2. `gameSubtitle` (short descriptive line)
      3. `gameThumbnailUri` (or equivalent image reference)
      4. `canStartSession` (boolean)
   3. Required session-list display fields for v1:
      1. `sessionDisplayName`
      2. `gameDisplayName`
      3. `sessionThumbnailUri` (fallback to game thumbnail when session-specific image absent)
      4. `hostDisplayName`
      5. `participantSummary` (for example current and max players)
      6. `sessionAvailabilityStatus` (joinable/full/closed/unavailable)
      7. `lastActivityDisplayHint` (human-readable freshness cue)
      8. `canCurrentPrincipalJoin` (boolean)
   4. Identifier handling rule:
      1. Stable IDs (gameId/sessionId) remain in transport and action payloads for command correctness.
      2. UI logic may consume IDs internally but must render friendly labels by default.
   5. Fallback behavior:
      1. Missing thumbnails resolve to a polished placeholder image, not a broken-image icon.
      2. Missing display text resolves to curated fallback labels, not raw IDs.
13. Lock 13 (2026-08-26): Lock v1 start-session and join-session workflows with explicit user intent.
   1. Start-session workflow:
      1. Entry context is `SignedIn` lobby scene with selected game.
      2. Explicit user intent action is required (`Start New Session`).
      3. Required pre-submit checks: authenticated principal and valid game selection.
      4. Success transitions to `SessionActive` with session summary hydration.
      5. Failure remains in `SignedIn` lobby with actionable error and retry path.
   2. Join-session workflow:
      1. Entry context is `SignedIn` lobby scene with server-refreshed joinable session list.
      2. Explicit user intent action is required (`Join Session`) after selecting a session.
      3. Required pre-submit checks: joinability status, principal join authorization, authenticated principal.
      4. Success transitions to `SessionActive` with session summary hydration.
      5. Failure remains in `SignedIn` lobby with actionable error and optional refresh.
   3. Shared v1 workflow rules:
      1. No implicit auto-start or auto-join in user-driven flows.
      2. Start and Join remain distinct actions.
      3. In-flight request disables duplicate-submit actions.
      4. Outcome diagnostics are emitted for success/failure transitions.
   4. Leave-session return rule:
      1. Successful leave returns to `SignedIn` lobby.
      2. Discovery refresh occurs on return.
14. Lock 14 (2026-08-26): Lock interrupted-session recovery flow for v1.
   1. Transition behavior:
      1. On interruption detection, transition `SessionActive` -> `SessionInterrupted`.
      2. Display interruption recovery surface immediately.
   2. Automatic reconnect behavior:
      1. Perform one immediate automatic reconnect attempt.
      2. Use a short bounded timeout window for the automatic attempt.
   3. Outcome behavior:
      1. Automatic reconnect success returns to `SessionActive` with recovered status cue.
      2. Automatic reconnect failure remains in `SessionInterrupted` and presents user actions.
   4. User-directed recovery actions:
      1. Retry reconnect.
      2. Return to lobby (`SignedIn`).
      3. Sign out.
   5. Retry discipline:
      1. User-triggered retries are allowed.
      2. Silent infinite reconnect loops are disallowed.
   6. Diagnostics and UX:
      1. Recovery status and failure reason are surfaced to user.
      2. Diagnostics events are emitted for interruption, each reconnect attempt, and final resolution.
15. Lock 15 (2026-08-26): Lock diagnostics channel availability and shell-level display model.
   1. Display location:
      1. Diagnostics are displayed in `diagnosticsDrawer` as an app-shell fixture.
      2. Availability is global across all scenes and macro states.
      3. Immersive layouts may minimize entry affordance but must not remove access.
   2. Logging and storage model:
      1. Primary source is client-side in-memory diagnostics stream (ring-buffer style).
      2. Development-mode console mirroring is optional.
      3. Server-side persistence of browser diagnostics is not required in v1.
   3. Always-available channels:
      1. state transitions
      2. authentication lifecycle events
      3. session lifecycle events
      4. transport request/response summaries
      5. warning and error stream
   4. Capability-scoped channels:
      1. session-delta detail stream
      2. reconnect attempt timeline
      3. performance timing samples
   5. Show/hide interaction policy:
      1. Drawer defaults to closed in all states.
      2. Toggle is available through explicit UI affordance and keyboard shortcut.
      3. State transitions cannot force diagnostics to become inaccessible.
   6. Safety and readability:
      1. Sensitive credential material must be redacted.
      2. Default verbosity is concise with user-expandable detail.
16. Lock 16 (2026-08-26): Lock minimal v1 producer-theme token scope for end-to-end flow validation.
   1. v1 token scope is intentionally small:
      1. Color tokens (three total):
         1. `color.chromePrimary`
         2. `color.accentPrimary`
         3. `color.textPrimary`
      2. Font tokens (two total):
         1. `typography.fontFamilyPrimary`
         2. `typography.fontFamilyDisplay` (optional)
   2. Out-of-scope in v1:
      1. Expanded color palettes and per-component token matrices.
      2. Advanced typography and spacing systems.
      3. Theme-driven animation behavior.
   3. Fallback behavior:
      1. Missing token uses deterministic client default.
      2. Invalid token value logs diagnostics warning and falls back to default.
      3. Unsupported font falls back to approved safe font stack.
      4. Theme token issues must not block app startup or session workflows.
   4. Expansion policy:
      1. Additional theme tokens are introduced incrementally after v1 plumbing stabilizes.
      2. Any expansion requires explicit lock review tied to proven UX need.
17. Lock 17 (2026-08-26): Lock v1 static-client caching and version policy.
   1. Static asset strategy:
      1. Build outputs for JS/CSS/images use fingerprinted filenames.
      2. Fingerprinted assets are served with long cache lifetime headers.
   2. Bootstrap HTML strategy:
      1. Client bootstrap HTML is served with no-cache or must-revalidate policy.
      2. Browser revalidates bootstrap document to discover newest asset fingerprints.
   3. External config JSON strategy:
      1. Orchestration/theme config JSON files are served with short cache plus revalidation policy.
      2. Config changes must become visible quickly without requiring hard browser cache clears.
   4. Diagnostics supportability:
      1. Client exposes build/config version identity in diagnostics stream.
   5. Constraint alignment:
      1. Policy is implemented through filenames and HTTP cache headers only.
      2. No smart host-side rewrite or SPA-aware serving behavior is introduced.
18. Lock 18 (2026-08-26): Lock pre-render hardening gate criteria for Phase 4 entry.
   1. Contract and schema readiness:
      1. No unresolved lock questions in the pre-render control surface.
      2. Contract schemas validate in CI.
   2. Core flow readiness:
      1. sign-in success and failure flows pass.
      2. discovery initial refresh and manual refresh flows pass.
      3. start-session and join-session flows pass.
      4. leave-session return-to-lobby flow passes.
   3. Interruption and recovery readiness:
      1. interruption transition to `SessionInterrupted` passes.
      2. single automatic reconnect attempt behavior passes.
      3. user-directed recovery actions pass.
   4. Host static-serving readiness:
      1. configured static mounts serve expected content.
      2. API/static namespace isolation remains intact.
      3. no-SPA-fallback behavior remains intact.
   5. Diagnostics readiness:
      1. diagnostics drawer access works in all macro states.
      2. required channels emit expected data.
      3. sensitive-material redaction checks pass.
   6. Quality gate readiness:
      1. lint passes.
      2. type-check passes.
      3. build passes.
      4. host static-smoke checks pass.
      5. targeted browser flow tests pass.
   7. UX readiness:
      1. discovery/session surfaces show friendly names and thumbnails.
      2. raw IDs are absent from primary end-user UI.
      3. mobile proof-of-concept orientation outcome (portrait viable or retired) is documented.
   8. Entry rule:
      1. Phase 4 rendering work remains blocked until all criteria above are green and recorded.
19. Lock 19 (2026-08-26): Lock v1 browser framework and toolchain baseline.
   1. Locked v1 stack:
      1. React for UI composition.
      2. TypeScript for typed contracts and maintainability.
      3. Vite for development/build pipeline.
   2. Routing alignment:
      1. Use hash-based routing in v1.
      2. This aligns with no-SPA-fallback, dumb static serving behavior in GameHost.
   3. Scope discipline:
      1. Keep framework extensions minimal until pre-render plumbing gate passes.
      2. Defer SSR/hybrid/meta-framework complexity from v1 scope.
20. Lock 20 (2026-08-26): Lock TypeScript strictness baseline with controlled exception policy.
   1. Required v1 strictness baseline:
      1. `strict: true`
      2. `noImplicitAny: true`
      3. `strictNullChecks: true`
      4. `noUncheckedIndexedAccess: true`
      5. `noFallthroughCasesInSwitch: true`
   2. Temporary relaxation policy:
      1. Allowed only for narrow third-party typing gaps or short-lived migration seams.
      2. Must be localized (line/file scope), never global configuration disablement.
      3. Must include tracked cleanup metadata (owner and target removal milestone).
   3. Disallowed relaxations:
      1. Turning strict mode off globally.
      2. Disabling `noImplicitAny` globally.
      3. Untracked blanket ignore patterns.
21. Lock 21 (2026-08-26): Lock CI guardrails with config validation as mandatory baseline.
   1. Mandatory config guardrails and validation gates:
      1. orchestration JSON schema validation.
      2. theme JSON schema validation.
      3. cross-file reference validation (state->scene, state->layoutProfile, scene->sectionVariant, section key existence).
      4. deterministic transition validation (duplicate or ambiguous handlers fail).
      5. unreachable/orphan macro-state validation.
   2. Mandatory non-config baseline gates:
      1. TypeScript type-check.
      2. browser client production build.
      3. GameHost static-mount smoke check.
   3. Recommended but non-blocking in initial rollout:
      1. lint gate.
      2. targeted browser flow tests.
   4. Merge policy:
      1. Any mandatory gate failure blocks merge.
22. Lock 22 (2026-08-26): Lock v1 browser project naming and placement.
   1. Project name is locked to `Storyboard.WebPortal`.
   2. Project location is locked to top-level sibling placement in the solution root (`Storyboard.WebPortal/`).
   3. Project ownership includes:
      1. browser shell source
      2. external orchestration and theme config files
      3. build pipeline artifacts consumed by GameHost static mounts

## Non-Goals for Initial Delivery
1. Multiplayer-specific UX flows beyond placeholder extension points.
2. Advanced remote debugger parity from simulator technical tooling.
3. Arbitrary user-authored scripting inside orchestration configs.
