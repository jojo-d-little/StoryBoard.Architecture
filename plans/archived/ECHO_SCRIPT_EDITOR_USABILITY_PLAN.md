# Echo Script Editor Usability Plan

Status: In progress (majority implemented; metadata extension pending)
Owner: Pending
Last updated: 2026-06-30

## 1) Objective
Improve authoring usability in the echo message script editor by restoring and formalizing `self.` variable access, improving token discoverability, and adding clear guidance/diagnostics without breaking existing content.

## 2) Problem Statement
1. `self.`-prefixed variable discovery regressed and is no longer visible in suggestions.
2. Authors lack immediate clarity on what variables are available in each action context.
3. Token usage relies on memory/trial-and-error instead of guided discovery.

## 3) Scope
In scope:
1. Restore `self.` completion and token resolution behavior.
2. Add structured variable metadata for better suggestions/guidance.
3. Improve completion ordering and contextual discoverability.
4. Add editor warnings for unknown/out-of-context tokens (non-blocking).
5. Add regression tests covering `self.` and token guidance behavior.
6. Update script-editor help/tooltips to document expected usage.

Out of scope (this pass):
1. Breaking runtime token contract changes.
2. Hard-fail validation that blocks save for unknown tokens.
3. Major redesign of editor layout beyond additive guidance affordances.

## 4) `self.` Contract (Agreed)
1. `self.name`
- In-game object name (`NameInGame` when set, otherwise object name).

2. `self.objectName`
- Raw object definition name (`Name`).

3. `self.<gameProperty>`
- Any game property defined on the action-attached object.
- Includes built-in and custom properties.

4. Matching and safety behavior
- Case-insensitive matching.
- Unknown `self.` tokens resolve to empty string at runtime (non-breaking).
- Editor surfaces warning diagnostics for unknown tokens.

## 5) Proposed Changes
### 5.1 Completion + Ordering
1. Ensure typing `self.` reliably opens completion list.
2. Surface `self.*` entries at top when prefix is `self.`.
3. Include all object properties for the current action scope.
4. Keep the quick `{}` list intentionally focused on `self.*` and `actionProperty.*`; use the chooser for broader scoped variables.

### 5.2 Variable Metadata Model
Add token metadata abstraction used by completion and guidance panels:
1. Token name
2. Description
3. Category (Self, Action, Composite, Scope, List/Count)
4. Context availability rules (action types/panels)
5. Example value template
6. Alias/synonym mapping (if any)

### 5.3 Editor Guidance UX
1. Add richer completion detail text (description + sample).
2. Add an "Available Variables" grouped view in script editor context.
3. Promote context-relevant tokens first for current action type.
4. Add quick-insert patterns for list/index token forms.

### 5.4 Quick Brace Picker + Full Variable Chooser Integration
1. Keep fast `{}` completion as the default quick-insert path.
2. Limit the `{}` quick list to:
- `self.*` tokens
- `actionProperty.*` tokens
3. Always pin `Choose variable...` as the first item in the `{}` suggestion popup.
4. Selecting `Choose variable...` opens the shared Variable Chooser dialog already used by Set Game Property workflows.
5. Variable Chooser remains the path for broader scope variables outside `self.*` and `actionProperty.*`.
6. Insertion behavior remains unchanged after selection (selected token inserted into current `{...}` expression).

### 5.5 Validation/Diagnostics
1. Warn for unknown tokens.
2. Warn for out-of-context tokens.
3. Warn for malformed indexed/list references.
4. Keep warnings non-blocking by default.

### 5.6 Docs/Tooltips
1. Update inline script-editor help to explain `self.`.
2. Provide examples for common authoring cases.
3. Align wording across action dialogs and script editor.

## 6) Test Plan
1. Regression: `self.` suggestions appear in completion.
2. Regression: `self.name` resolves correctly.
3. Regression: `self.objectName` resolves correctly.
4. Regression: custom object properties appear as `self.<property>`.
5. Regression: unknown `self.` tokens produce warning diagnostics and do not crash.
6. Regression: existing composite/action tokens remain available and unchanged.

## 7) Implementation Phases
### Phase U1: Restore `self.` baseline
1. Reintroduce `self.` completion entries.
2. Restore token resolution for `self.name`, `self.objectName`, and object properties.
3. Add minimum regression tests for restored behavior.

Exit criteria:
1. `self.` options visible in editor suggestions.
2. Runtime token resolution verified by tests.

Status:
1. Complete.

### Phase U2: Guidance metadata + completion polish
1. Implement token metadata model.
2. Upgrade completion entries with descriptions/examples.
3. Context filtering and ordering improvements.
4. Add `Choose variable...` launch entry at top of brace completion.
5. Restrict quick brace suggestions to `self.*` and `actionProperty.*`; rely on chooser for all other variables.

Exit criteria:
1. Suggestions show meaningful guidance.
2. Context ordering behaves deterministically.
3. Brace popup always exposes `Choose variable...` first.
4. Non-`self.*`/non-`actionProperty.*` variables are discoverable through chooser rather than quick list.

Status:
1. Mostly complete.
2. Remaining gap: richer metadata dimensions (explicit context availability rules and alias/synonym mapping).

### Phase U3: Validation and docs alignment
1. Add non-blocking token diagnostics.
2. Update help text/tooltips/documentation.
3. Add regression tests for warning behaviors.

Exit criteria:
1. Authors receive actionable feedback for token mistakes.
2. No content-breaking regressions.

Status:
1. Complete for current diagnostics behavior and help text baseline.

## 8) Risks and Mitigations
1. Risk: Over-filtering hides tokens authors still need.
- Mitigation: Keep quick list intentionally focused and provide full chooser path for broad discovery.

2. Risk: Validation warnings become noisy.
- Mitigation: Keep warnings concise and context-aware; avoid blocking save.

3. Risk: Regression in existing token behavior.
- Mitigation: Add targeted regression tests for legacy token sets.

## 9) Acceptance Criteria
1. Authors can discover `self.` variables without memorization.
2. `self.name`, `self.objectName`, and `self.<gameProperty>` are available and reliable.
3. Script editor provides clear variable guidance by action context.
4. Unknown/out-of-context token usage is surfaced as non-blocking warnings.
5. Build and relevant tests pass with no runtime contract regressions.
6. Brace completion keeps fast insertion for `self.*` and `actionProperty.*` while providing `Choose variable...` as a consistent top entry.
7. Authors can still select any other variable via the shared chooser without expanding quick-list clutter.

## 10) Current Status Snapshot
1. Implemented:
- `self.` runtime aliasing and resolution reliability.
- Quick brace filtering to `self.*` + `actionProperty.*`.
- `Choose variable...` pinned as first quick entry.
- Available Variables grouped guidance panel.
- Non-blocking unknown-token and malformed-index warnings.
- Core regression tests for token filtering/metadata/diagnostics.

2. Remaining:
2.1 Keep quick brace policy as-is (restricted to `self.*` and `actionProperty.*` + chooser). Do not re-expand quick suggestions.
2.2 Decide whether to extend metadata model to include explicit action-context availability rules.
2.3 Decide whether to add alias/synonym metadata for token guidance (add only if a concrete authoring use-case is identified).

## 11) Metadata Extension Decision
Purpose: Resolve item 2.2 and item 2.3 with explicit go/defer gates.

### 11.1 Decision A: Context Availability Metadata (item 2.2)
Summary:
1. This metadata annotates each token with where it is valid (action types/editor surfaces), enabling context-aware ranking/filtering in guidance UI.

Go criteria:
1. There are repeated authoring complaints/tests showing out-of-context token confusion.
2. We can map token availability without changing runtime contracts.
3. Guidance can consume this metadata as additive behavior only (no save-blocking).
4. We can add deterministic tests for category/order/filter behavior.

Defer criteria:
1. Token context rules are too volatile during ongoing action-model refactors.
2. We cannot define stable availability semantics per action type yet.
3. The only expected benefit is cosmetic, with no measured discoverability gain.

Decision status:
1. Recommended: Go.
2. Target shape: add optional metadata fields for allowed action types/panels; use for ranking/grouping/warnings only.

### 11.2 Decision B: Alias/Synonym Metadata (item 2.3)
Summary:
1. This metadata maps alternate authoring terms to canonical tokens for guidance/discovery.

Go criteria:
1. At least one concrete use-case exists (legacy token rename, common authoring synonym, or migration support).
2. Canonical token remains clear and singular in UI output.
3. Alias behavior is guidance-only unless a separate runtime compatibility decision is approved.
4. Tests cover canonical display plus alias lookup behavior.

Defer criteria:
1. No concrete synonym/rename case exists.
2. Alias mapping risks adding ambiguous or duplicate-looking suggestions.
3. The change would increase cognitive load more than discoverability.

Decision status:
1. Recommended: Defer until a concrete use-case is documented.
2. Trigger to revisit: first approved migration/synonym scenario.

### 11.3 Follow-up Tasks
1. If Decision A is approved, create a small implementation note with schema fields and test updates before coding.
2. Keep item 2.1 unchanged: quick brace remains restricted to `self.*` and `actionProperty.*` + chooser.
3. Re-open Decision B only when a specific alias/synonym case is captured in plan notes.
