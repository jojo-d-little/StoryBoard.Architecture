# Designer Area Profile

Purpose: boundary profile for Designer authoring, export, process orchestration, launch UX, and Designer-owned tests.

## Default Scope

- Primary edit areas: `StoryBoard.Designer/StoryboardDesigner.App/**`, `StoryBoard.Designer/StoryboardDesigner.App.Tests/**`, and `StoryBoard.Designer/StoryboardDesigner.App.SmokeTests/**`.
- Designer may save, validate, export, construct established launch configuration, start/manage external processes, open the browser, and report status.

## Boundary Rules

1. Designer does not implement GameHost session behavior, mutate the durable production catalog, or parse authoritative runtime state.
2. During F01-F07, a new GameHost/WebPortal action is added beside the existing WPF Simulator action; only F08 retires the legacy action.
3. Consume only approved shared contracts and established configuration formats.

## Required Pass Evidence

1. Focused Designer test/smoke results.
2. Clear launch, validation, readiness, retry, cleanup, and failure UX behavior where applicable.
3. Exact next consumer or integration pass.
