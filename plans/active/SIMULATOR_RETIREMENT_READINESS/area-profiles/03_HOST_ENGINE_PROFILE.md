# Host / Engine Area Profile

Purpose: boundary profile for authoritative runtime, session, host, persistence, recording, and flow-trace behavior.

## Default Scope

- Primary edit areas: `StoryBoard.GameEngine/Storyboard.GameEngine/**`, `StoryBoard.GameEngine/Storyboard.GameEngine.Tests/**`, `StoryBoard.GameEngine/Storyboard.Shared/**`, `StoryBoard.GameEngine/Storyboard.GameHost/**`, `StoryBoard.GameEngine/Storyboard.GameClient/**`, and associated focused tests.

## Boundary Rules

1. Preserve production isolation; development capabilities are absent or rejected outside approved development startup.
2. Keep authority in runtime/session services; browser and WPF dependencies do not enter runtime projects.
3. Use approved Contracts surface only. F01 uses the existing registration-file override and must prove discovery, session, and assets resolve the same identity.

## Required Pass Evidence

1. Focused runtime/host/socket/security-negative validation as applicable.
2. Authoritative state, lifecycle, error, and capability semantics for the next consumer pass.
3. Exact interface or configuration assumptions consumed by Designer, Portal, or integration.
