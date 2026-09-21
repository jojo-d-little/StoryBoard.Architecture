# WebPortal Area Profile

Purpose: boundary profile for browser development UX, Pixi presentation controls, Portal diagnostics, and host-client consumption.

## Default Scope

- Primary edit areas: `StoryBoard.WebPortal/Storyboard.WebPortal/**` and `StoryBoard.WebPortal/Storyboard.WebPortal.Tests/**`.

## Boundary Rules

1. Pixi/rendering components do not call host APIs directly; Portal hooks/workflows own transport.
2. Portal consumes approved host contracts and established host behavior; it does not receive arbitrary host filesystem access.
3. Development controls are capability-gated and unavailable in production mode.
4. Portal diagnostics captures/exports client trace; it does not aggregate backend logs.

## Required Pass Evidence

1. Focused component/workflow/build/contract/visual validation as applicable.
2. User-visible behavior, capability gating, failure states, and explicit next integration evidence.
