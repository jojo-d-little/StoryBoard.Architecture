# Asset Prediscovery and Download Cache Plan

Status: Closed (archived 2026-08-14)

## 1. Purpose

Define a host-facing, cross-asset discovery interface that allows hosts to prefetch and locally cache likely-near-future runtime assets.

This plan intentionally covers both image and audio assets.

## 2. Why This Is Separate

1. Sound cue playback payload and asset prefetch payload are different concerns.
2. Sound phase-1 work remains immediate command-response playback intent.
3. Prediscovery and cache policy can be introduced later without changing immediate playback semantics.

## 3. Scope

In scope:

1. Runtime-to-host discovery contract design.
2. Cross-asset candidate model (images and audio first).
3. Priority and budgeting hints for host prefetch decisions.
4. Browser and remote-runtime safety constraints.

Out of scope:

1. Host-specific downloader implementations.
2. Shared-runtime file/network transport code.
3. Cache eviction policy standardization across hosts.
4. Event-driven orchestration model beyond first discovery surface.

## 4. Principles

1. Shared runtime computes discovery hints; host performs downloads.
2. Discovery payload is source-agnostic and asset-type-neutral.
3. Deterministic ordering is required for identical runtime state.
4. Payload must be bounded by host-supplied limits.
5. Runtime behavior remains correct even if host ignores discovery hints.

## 5. Proposed Interface Direction

Recommended direction:

1. Introduce a dedicated host discovery interface.
2. Do not embed discovery lists into ordinary command responses.
3. Keep current-presentation hydration and discovery as separate APIs.

Candidate seam (draft):

1. Host requests asset discovery snapshot for a runtime context.
2. Runtime returns prioritized candidates with reason tags and cache hints.

## 6. Candidate Payload Shape (Draft)

1. Snapshot metadata:
1. Runtime state anchor (for example room id plus scope context identifiers).
2. Generated timestamp.
3. Optional sequence token for incremental follow-up.
4. Request scope anchor is current-room context; host does not need global world-asset awareness.
5. Truncation indicator when host budget caps were hit.
6. Optional continuation token so host can request additional candidates for the same scoped snapshot.

2. Asset candidate descriptors:
1. AssetType (Image, Audio, extensible).
2. RuntimeAssetRef (portable runtime export reference).
3. StableAssetId (optional when available).
4. ContentHash or ETag hint (optional when available).
5. ByteSizeHint (optional).

3. Prefetch hints:
1. PriorityTier (CurrentRoom, AdjacentRoom, AncestorActionLikely, GlobalHotset).
2. PriorityScore within tier.
3. ReasonTags for diagnostics.
4. Optional preload deadline hint.

## 7. Candidate Priority Inputs

1. Current room background and renderable object images.
2. Current room likely sound effects from applicable action outcomes.
3. Adjacent room render assets.
4. Ancestor-scope action sound assets applicable in current room context.
5. Optional bounded look-ahead for immediate traversal neighbors.

## 8. Budget and Safety Constraints

1. Discovery interface should support max-item and max-byte limits.
2. Runtime should return deterministic top-N ordering.
3. Host may select policy profile (conservative, balanced, aggressive).
4. Missing assets should be discoverable via diagnostics rather than silently hidden.
5. Contract must be safe for browser-hosted deployment and web-backed asset origins.

## 9. Lock-Off Decisions (Plain Text)

Status key:

1. Open means the team still needs a final decision.
2. Locked means approved and implementation-ready.

Current lock state:

1. ADP-01 is Locked. Use a dedicated host discovery interface instead of embedding discovery lists in command responses.
2. ADP-02 is Locked. Discovery requests are session-scoped, with runtime resolving current-room context internally and optionally expanding to adjacent and ancestor-applicable candidates based on policy profile and server limits.
3. ADP-03 is Locked. Priority is represented as tier plus score plus reason tags.
4. ADP-04 is Locked. Snapshot mode starts with full room-scoped snapshots; incremental diff mode is a later additive enhancement, and invalid or missing diff tokens fall back to full snapshot responses.
5. ADP-05 is Locked. Host cache identity uses relative locator/path token as primary key for this phase, with optional hash/etag metadata for revalidation.
6. ADP-06 is Locked. Runtime owns fine-grained candidate caps and deterministic ordering; host selects from supported policy profiles. When capped, response includes truncation and optional continuation token so host can request more data.
7. ADP-07 is Locked. Missing assets are included with warning diagnostics rather than omitted silently.
8. ADP-08 is Locked. Browser and remote-host safety is required in the first contract draft. Partial relative locator hints are allowed in discovery and download requests, but physical path resolution remains server/engine-only.
9. ADP-09 is Locked. Use one unified asset accessor interface for image and audio retrieval. Split into media-specific accessors only if proven divergence appears in authorization domains, delivery mechanics, processing pipeline requirements, infrastructure ownership, or operational limits.
10. ADP-10 is Locked. Retrieval identity uses a server-resolved relative locator/path token as the primary identifier. New runtime asset-ref IDs are not required for this phase. Physical-path dereference remains server/engine-only.
11. ADP-11 is Locked. Retrieval payload contract uses bytes plus metadata as the baseline shape. In-process streaming helpers may be added later as implementation optimizations without changing the transport-neutral contract.
12. ADP-12 is Locked. Cache validation uses hash/etag revalidation with explicit NotModified response support. Host clients must also support a local force-clear cache control; for current phase this is a simulator-host responsibility and does not require server-side cache state.
13. ADP-13 is Locked. Use explicit enum-based result codes for retrieval outcomes (for example not found, forbidden, invalid reference, not modified, source unavailable). Freeform messages are supplemental diagnostics only.
14. ADP-14 is Locked. Asset retrieval requires session/principal context and visibility/authorization checks; unrestricted retrieval by known reference is not allowed.
15. ADP-15 is Locked. After runtime/project load, all asset retrieval paths (including debugger-driven inspection/playback) use the unified asset accessor seam. Direct disk asset reads are not part of normal or debugger asset retrieval flows.

## 10. Lock Order

1. Lock ADP-02 through ADP-08 first so discovery contract shape and safety limits are stable.
2. Lock ADP-09 through ADP-12 next so accessor request/response and cache validation semantics are fixed.
3. Lock ADP-13 through ADP-15 last so errors, authorization, and migration enforcement are implementation-ready.

## 11. Exit Criteria

1. ADP-01 through ADP-15 are all Locked.
2. Contract touch-list is approved.
3. Shared/runtime and host implementation slices are scheduled.
4. Regression test matrix draft is approved.

## 12. Initial Implementation Slice Ideas (Deferred)

1. Add discovery contract types and host interface methods.
2. Add runtime discovery service for current-room context.
3. Add deterministic ranking and budget trimming.
4. Add simulator-side local prefetch prototype.
5. Add diagnostics and focused regression tests.

## 13. First Execution Slice (Path Decoupling Before Prediscovery)

This slice is the recommended first step and should complete before broad prefetch orchestration work.

Goal:

1. Remove host direct-to-disk dependency for runtime asset retrieval.
2. Route asset retrieval through a host-facing runtime interface using asset references and hints.
3. Keep backend physical path ownership internal to runtime-side implementations.

Proposed interface direction:

1. Add a host/runtime seam named `IHostAssetManagementClient`.
2. Keep transport-neutral method semantics so in-process C# calls can later map to remote HTTP without host call-site changes.
3. Retrieval requests should accept source-agnostic identifiers:
 - `RelativeLocator` or partial relative path token (primary key for this phase).
 - optional host cache metadata (etag/hash or last-seen fingerprint).
 - optional future extension for explicit runtime asset reference if later needed.
4. Hosts may pass back the locator hint they received, but hosts must not require knowledge of backend physical storage layout.

Retrieval response requirements:

1. Return asset payload stream/bytes plus metadata needed for cache handling.
2. Include cache freshness and validation hints (for example ETag/content-hash and optional expiration timestamp).
3. Include explicit not-modified outcome so host can retain local cache without re-downloading payload bytes.

Host caching expectations for this slice:

1. Host keeps a local asset cache keyed by relative locator/path token (with optional future support for explicit runtime asset references).
2. Host revalidates cache entries through the accessor seam before payload refresh.
3. Host UI/runtime call paths stop opening runtime asset files directly from disk.
4. Typical host loop is: on first visit to a room, request room-scoped discovery, check local cache for returned asset identifiers, then fetch missing entries through the accessor seam.
5. Host provides a force-clear local cache control for troubleshooting and deterministic test/reset flows.

Out of scope for this first slice:

1. Full pre-discovery scheduling.
2. Aggressive multi-hop look-ahead heuristics.
3. Global cache eviction policy standardization.

Completion criteria for slice 1:

1. Normal image/sound render/playback paths can fetch assets via host/runtime accessor seam.
2. Simulator host and debugger post-load workflows no longer require direct knowledge of backend physical asset locations for asset retrieval.
3. Local cache revalidation behavior is wired and validated with focused tests.
4. Prediscovery work can then build on this seam as a second layer.

## 14. Current Implementation Status (2026-08-14)

Completed in code:

1. Unified asset accessor seam is implemented and used for simulator asset retrieval paths via host-facing retrieval calls.
2. Simulator local disk cache is implemented with locator-based partitioning, ETag revalidation, and NotModified handling.
3. Discovery warm-up path is implemented and routes candidate hydration through the same cache/get-asset flow as on-demand retrieval.
4. Discovery pagination support is implemented with continuation token handling and bounded warm-up paging.
5. Discovery scope depth control is implemented:
 - `NearMe` for current room plus nearby scope.
 - `Ancestors` for broader first-pass warm-up coverage.
6. Simulator warm-up policy now performs one Ancestors warm-up per attached session, then uses NearMe for subsequent room-entry warm-ups.
7. Focused regression coverage is in place for runtime discovery depth behavior and simulator warm-up depth sequencing.
8. Client warm-up pagination hardening is implemented with explicit stop conditions and warnings for non-advancing/repeating continuation tokens, plus page-cap warning behavior.
9. Simulator host now exposes a force-clear local discovered-asset cache control (command + UI action) with focused regression coverage.

Closure notes:

1. Exit criteria are considered satisfied for this phase: lock decisions are complete and implementation slices are delivered with focused regression coverage.
2. Browser/remote transport conformance tests are intentionally deferred to the dedicated remote-host phase so they can validate the actual transport boundary rather than in-process host calls.
3. Force-clear cache UX parity for non-simulator host surfaces is deferred to each host integration track and is not a blocker for closing this simulator/runtime seam plan.
