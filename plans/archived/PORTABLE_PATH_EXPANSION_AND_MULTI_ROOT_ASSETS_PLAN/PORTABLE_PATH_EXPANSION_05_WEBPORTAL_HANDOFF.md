# Portable Path Expansion — Stage 05 WebPortal Verification Handoff

Status: Complete — verification-only; no WebPortal implementation change required  
Stage: 05 of 07  
Date: 2026-09-18  
Owner Session: Codex / StoryBoard.WebPortal

## Opening Prompt (Use To Start This Stage)

Execute only Stage 05 of `plans/active/PORTABLE_PATH_EXPANSION_AND_MULTI_ROOT_ASSETS_PLAN.md`, starting from the completed Stage 04 Simulator verification. Verify WebPortal remains a runtime consumer: its static files are mounted by GameHost and its game assets are host/runtime-relative URLs. Make no code change unless verification finds a concrete consumer defect; record evidence either way.

## Stage Boundary Allowlist Snapshot

Allowed read: Stages 03-04 handoffs, `StoryBoard.WebPortal/**`, Host asset/transport contracts.  
Allowed edit: `StoryBoard.WebPortal/**` only for a proven defect; otherwise this handoff only.

## Scope Completed

Verified that WebPortal remains a runtime consumer and does not resolve authoring source roots.

1. The WebPortal package contains compiled static content only and copies it to the consuming Host's `wwwroot` output/publish directories.
2. Vite uses a mount-agnostic relative base (`./`); no source-root or physical filesystem dependency is present in the browser bundle configuration.
3. Game image, sound, and catalog requests use host APIs with runtime-relative locators such as `assets/images/...`, `assets/sounds/...`, and `assets/PresentationCues/...`.
4. Host scene mapping carries runtime image locators into the renderer; the session workflow resolves them through the Host asset API into browser-loadable data URLs before Pixi rendering.
5. No WebPortal code references `ASSETROOT:/`, `STORYBOARD_ASSET_SOURCE_ROOT`, environment expansion, or authoring physical paths.
6. No WebPortal change is required for Portable Path Expansion. Source-root expressions remain a Host/Engine concern and must not be introduced into browser code.

## Files Changed

Only this handoff was updated. No files under `Storyboard.WebPortal`, `Storyboard.WebPortal.Package`, or `Storyboard.WebPortal.Tests` were modified.

## Contract/Interface Impact

Expected: none.

## Validation Commands Executed

1. `npm run build` — passed; production Vite bundle generated successfully.
2. `npm test` — passed; 21 test files, 179 tests.
3. Focused image/runtime suite — passed; 3 files, 26 tests covering host scene mapping, runtime image hydration, and Pixi rendering behavior.
4. `npm run test:visual` — passed; 4 browser smoke tests, including room-object and directional-image screenshots.
5. `npm run verify:contracts` — passed; 3 host contract files verified.
6. `dotnet test Storyboard.WebPortal.Tests/Storyboard.WebPortal.Tests.csproj --configuration Release` — test assembly built, but 13 repository-level contract tests failed because this checkout does not contain the expected `StoryboardDesigner.slnx`; this is a test-harness path assumption unrelated to Portable Path Expansion or WebPortal runtime asset handling.

## Test Results

Host-mounted/static bundle and runtime asset URL verification passed. Browser image smoke coverage passed.

## Behavioral Notes

No authoring-time asset root should reach browser code.

## Known Issues/Risks

No WebPortal filesystem assumption was found. Any future discovered filesystem assumption belongs to the owner that introduced it; do not introduce source-root resolution in WebPortal by default.

## Boundary Compliance Report

Out-of-scope reads: None.  
Out-of-scope edits: None.  
Exceptions: The repository-level .NET contract tests cannot locate the external/shared `StoryboardDesigner.slnx` from this checkout; this predates and is independent of Stage 05.

## Explicit Next-Stage Start Checklist

1. Provide WebPortal verification/no-change evidence to the migration or closeout stage.
2. Preserve runtime-relative asset URL invariants.
