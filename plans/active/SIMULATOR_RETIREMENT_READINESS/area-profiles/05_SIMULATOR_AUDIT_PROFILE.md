# Simulator Audit Area Profile

Purpose: preserve useful Simulator behavior, fixtures, and tests as evidence while preventing additional WPF renderer investment.

## Default Scope

- Primary edit areas: `StoryBoard.GameEngine/Storyboard.Simulator/**`, `StoryBoard.GameEngine/Storyboard.Simulator.Tests/**`, and `StoryBoard.GameEngine/Storyboard.Simulator.SmokeTests/**`.

## Boundary Rules

1. No new WPF visual/parity feature work.
2. Audit a completed replacement feature only; record `Present`, `Replaced`, `Waived`, or `Blocking` status and durable fixture/test ownership.
3. Move semantic tests to their long-term owner before weakening simulator coverage.

## Required Pass Evidence

1. Capability disposition for the specific feature.
2. Tests/fixtures ported, retained temporarily, or deferred with owner.
3. Remaining P0 blockers and approved P1 follow-ups.
