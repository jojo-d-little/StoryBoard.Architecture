# Validation Rules Management Plan

Status: Completed
Owner: StoryboardDesigner.App authoring workflows
Last updated: 2026-07-03

## Completion Decision

As of 2026-07-03, this plan is considered complete for its implemented scope.

Closeout notes:

1. Validation execution architecture and rule extraction are in place and test-covered.
2. Node-level ignore behavior is now node-only, while project-level ignores remain global.
3. Traversal validation visibility and linked-instance action-sharing fixes are implemented and validated.
4. Any optional inherited-ignore semantics are explicitly deferred as a future enhancement.

## 1. Purpose

Establish a maintainable validation architecture where each logical validation rule is implemented as a dedicated class with stable identity, predictable severity behavior, and clear scope.

This plan focuses on rule management and execution architecture, not on changing rule intent unless explicitly called out.

## 2. Why This Plan Exists

Validation has become a core product feature for authoring quality and export/runtime safety.

Current validation behavior is functionally strong but implementation is distributed across mixed patterns:

1. Legacy string-based project validation in MainWindowViewModel partial.
2. Typed traversal validation issues in TraversalValidationService.
3. Linked-action graph validation as a separate static validator.
4. Script diagnostics validation in a separate analyzer pipeline.

As the rule count grows, this pattern increases refactor cost, testing complexity, and risk of inconsistent behavior.

## 3. Goals

1. One logical rule = one rule class.
2. Shared, typed validation result contract across pipelines.
3. Stable RuleId for every rule (for reporting, suppression, telemetry, tests).
4. Deterministic ordering and execution.
5. Explicit scope targeting (project/planet/country/area/room/object/action/traversal leg/etc.).
6. Keep existing user-visible behavior compatible during migration.
7. Improve testability with focused rule-level tests.
8. Keep validation implementation 100% in StoryboardDesigner.App as a producer/design-time responsibility.

## 4. Non-Goals

1. No broad rule semantics rewrite in initial migration.
2. No UI redesign beyond what is needed to display richer rule metadata.
3. No cross-host coupling that violates existing architecture boundaries.
4. No removal of current guardrails during transition.
5. No validation engine, contracts, rule classes, or execution plumbing in Storyboard.Shared.
6. No validation execution dependencies in Storyboard.Simulator.

## 4.1 Boundary Lock (Designer-Only)

This validation initiative is explicitly design-time and producer-centric.

Hard boundaries:

1. All validation rule management code lives in StoryboardDesigner.App.
2. Storyboard.Shared remains free of designer validation engine abstractions.
3. Storyboard.Simulator remains free of designer validation dependencies.
4. Runtime/shared contracts may be validated by designer rules, but rule execution ownership remains in designer app.

## 5. Current Validation Inventory (Logical Rule Families)

Current families identified for migration:

1. Project structure presence and starting-scope validity.
2. Duplicate naming checks across multiple scopes.
3. Producer-notes required checks.
4. Command must have linked action checks.
5. Linked-action graph integrity rules.
6. Traversal topology rules.
7. Traversal leg isPassable contract rules.
8. Door-traversal linkage integrity (IsDoorLinkedToTraversal).
9. Save/export traversal hard-gate behavior.
10. Action script diagnostics rules.

Note: this inventory is the baseline set for extraction and formal RuleId assignment.

## 6. Proposed Target Architecture

## 6.1 Core Contracts

Introduce typed contracts in StoryboardDesigner.App validation layer:

1. ValidationRuleContext
2. ValidationIssue
3. ValidationSeverity
4. ValidationScope
5. ValidationRuleMetadata (RuleId, title, default severity, category)

### 6.1.1 ValidationRuleContext (Locked v1)

ValidationRuleContext is the runtime data payload passed to Evaluate after eligibility succeeds.

Locked shape:

```csharp
public sealed record ValidationRuleContext(
   ProjectModel Project,
   ScopeNodeBase CandidateNode, // Current node being evaluated in this invocation.
   ScopeNodeBase RequestRoot, // User-selected root that defines validation run scope.
   ValidationExecutionRequest Request,
   ValidationProfile ActiveProfile,
   bool IncludeDescendants,
   bool IsRecursivePass,
   bool IsFullProject,
   IValidationLookupService Lookup,
   IValidationSharedState SharedState,
   CancellationToken CancellationToken = default);

public interface IValidationSharedState
{
   bool TryGet<T>(string key, out T? value);
   T GetOrAdd<T>(string key, Func<T> factory);
}

public interface IValidationLookupService
{
   bool TryGetSharedPropertyRelationship(Guid sourceVariableId, out ValidationSharedPropertyRelationship relationship);
   bool TryGetPropertyByVariableId(Guid variableId, out ValidationGamePropertyDescriptor property);
   IReadOnlyList<ValidationSharedPropertyRelationship> GetSharedPropertyRelationshipsForVariable(Guid variableId);
}

public sealed record ValidationSharedPropertyRelationship(
   Guid SourceVariableId,
   Guid TargetVariableId,
   string RelationshipKind,
   string SourceScopePath,
   string TargetScopePath);

public sealed record ValidationGamePropertyDescriptor(
   Guid VariableId,
   string VariableName,
   string ScopePath,
   string ValueRestriction,
   bool IsShared,
   string? SemanticHint);
```

Field intent:

1. Project: full authored model graph.
2. CandidateNode: node currently being evaluated by the rule.
3. RequestRoot: user-selected root scope for this run.
4. Request: original execution request values.
5. ActiveProfile: resolved profile after catalog lookup.
6. IncludeDescendants/IsRecursivePass/IsFullProject: normalized execution flags.
7. Lookup: engine-provided read-only lookup/query surface for cross-node integrity checks.
8. SharedState: optional engine-provided scratch/cache for cross-rule reuse.
9. CancellationToken: cooperative cancellation support.

Read-only guarantees:

1. Rules should treat ValidationRuleContext as read-only.
2. Rules should not mutate Project or CandidateNode.
3. Rules should not mutate Request, ActiveProfile, or any object reachable from them.
4. SharedState is for performance caches only, not behavioral side effects.
5. If a rule needs expensive graph lookups repeatedly, cache by deterministic key in SharedState.
6. Evaluate must not trigger UI updates, persistence writes, or external service calls.
7. Cross-node integrity lookups must go through IValidationLookupService, not direct mutation-capable model APIs.
8. IValidationLookupService outputs are immutable descriptors/snapshots, not live mutable model objects.

SharedState usage rules:

1. Keys are deterministic and namespaced by RuleId (example: "TRV-001:RoomLookup").
2. Cached values are derived from current request/project inputs only.
3. Rules must not use SharedState to communicate behavioral flags between rules.

Lookup service rules:

1. Lookup service lifetime is one validation run (constructed by engine/orchestrator per Execute call).
2. Lookup data is read-only and deterministic for the same request + project snapshot.
3. Rules may compose lookup results but must not retain references across runs.
4. Expensive lookup indexes may be internally precomputed once per run and reused by rules.

Separation note:

1. ValidationEligibilityContext answers "should this rule run here?"
2. ValidationRuleContext supports "run now and emit issues".

Proposed minimum rule interface:

```csharp
public interface IValidationRule
{
    ValidationRuleMetadata Metadata { get; }
   IReadOnlySet<ScopeType> SupportedCandidateScopeTypes { get; }
   ValidationEligibilityResult CanEvaluate(ValidationEligibilityContext eligibility);
    IEnumerable<ValidationIssue> Evaluate(ValidationRuleContext context);
}
```

## 6.2 Rule Organization

Proposed folder structure:

1. StoryboardDesigner.App/Validation/
2. StoryboardDesigner.App/Validation/Contracts/
3. StoryboardDesigner.App/Validation/Rules/Project/
4. StoryboardDesigner.App/Validation/Rules/Traversal/
5. StoryboardDesigner.App/Validation/Rules/Actions/
6. StoryboardDesigner.App/Validation/Rules/Scripting/
7. StoryboardDesigner.App/Validation/Execution/

## 6.3 Execution Model

1. ValidationRuleRegistry provides ordered rule set.
2. ValidationEngine runs rules deterministically and aggregates issues.
3. Pipelines (save, explicit validate command, export preconditions, editor diagnostics) choose rule sets by profile.
4. Profiles support targeted execution (for example, traversal-only gate for export).

## 6.4 Rule Identity

Each rule gets stable RuleId format:

1. Canonical pattern: `^[A-Z]{3}-\\d{3}$`.
2. Prefix is exactly 3 uppercase letters identifying the rule family (examples: PROJ, NAME, OBJ, CMD, ACT, TRV, SCR).
3. Numeric segment is 001-999, zero-padded to 3 digits.
4. RuleId appears in report output, logs, and tests.
5. RuleId remains stable across internal refactors.

Ownership and allocation policy (v1):

1. Ownership: StoryboardDesigner.App validation maintainers own RuleId assignment.
2. Source of truth: one central registry map in validation composition code (RuleId -> rule type).
3. Allocation: next available number within the family prefix; never reuse retired IDs.
4. Rename behavior: class/type names may change, RuleId must not change unless a deliberate contract break is approved.
5. Merge conflict rule: if duplicate candidate IDs are proposed, keep the earlier merged ID and renumber the newer addition before merge.

## 6.5 Scope-Tree-First Rule Classification

Validation rule classification is anchored to the authored scope tree model.

### 6.5.1 Single Node Validation

Definition:

1. Rule evaluates one scope node in isolation.
2. Rule does not require traversing children or global state.

Examples:

1. Required properties on a node (Name, Id, mandatory flags).
2. Node-local variable shape checks.

Execution model:

1. Engine iterates eligible nodes.
2. Rule runs once per eligible node.
3. Issue path points to evaluated node.

### 6.5.2 Recursive Validation

Definition:

1. Rule starts from root/global scope and recursively traverses the full project tree.
2. Rule requires cross-node aggregation or correlation.

Examples:

1. Duplicate ObjectId detection across entire project.
2. Global uniqueness constraints spanning many scopes.

Execution model:

1. Rule receives root context and traversal utility.
2. Rule may keep internal lookup state (for example HashSet or Dictionary).
3. Issues can target specific offending nodes and optionally a project-level summary issue.

### 6.5.3 Node Scope Applicability (Orthogonal To ExecutionKind)

Definition:

1. Rule may apply to all node types, or only to a constrained set of node types.
2. This is separate from whether rule execution shape is SingleNode or Recursive.

Examples:

1. Traversal leg door-link shared-variable integrity checks.
2. Action graph rules for action nodes.
3. Duplicate-name recursive checks that can still be constrained to selected scope families.

Execution model:

1. Rule exposes supported candidate node scope types as a list/set capability.
2. Engine uses capability as a fast pre-filter.
3. Rule CanEvaluate remains the final runtime authority.

### 6.5.4 Classification Notes

1. Classification describes execution shape, not severity.
2. A rule has one primary classification for ownership and testing.
3. Node scope applicability is orthogonal metadata and can coexist with either SingleNode or Recursive execution shape.

### 6.5.5 Invocation Scope (Derived From Request + TraversalRequirement)

Invocation scope is derived from request shape and traversal requirement, rather than a separate metadata axis.

Derived modes:

1. FullProject:
   - Global root + IncludeDescendants = true.
2. RootOnly:
   - Any root + IncludeDescendants = false.
3. Subtree:
   - Non-global root + IncludeDescendants = true.

Notes:

1. This removes redundant overlap between ProjectOnly/SubtreeCapable and traversal requirement flags.
2. Full-project-only behavior is expressed via TraversalRequirement = FullProjectTraversalRequired.
3. Validation UI/commands should surface when a result is partial due to subtree scope.

## 6.6 Engine Model For Classified Rules

To support classification cleanly, rules should declare execution intent in metadata.

Proposed additions:

1. ValidationExecutionKind enum:
   - SingleNode
   - Recursive
2. Node scope applicability capability is exposed by each rule and evaluated at runtime.
3. Deterministic scheduler groups rules by execution kind to avoid accidental repeated full-tree traversals.
4. Traversal requirement metadata determines invocation-depth constraints and full-project-only behavior.

Additional enum:

1. ValidationTraversalRequirement:
   - NodeOnly
   - DescendantsRequired
   - FullProjectTraversalRequired

Possible contract sketch:

```csharp
public enum ValidationExecutionKind
{
    SingleNode,
   Recursive
}

public enum ValidationTraversalRequirement
{
   NodeOnly,
   DescendantsRequired,
   FullProjectTraversalRequired
}

public sealed record ValidationRuleMetadata(
    string RuleId,
    string Title,
    ValidationSeverity DefaultSeverity,
    string Category,
    ValidationExecutionKind ExecutionKind,
   ValidationTraversalRequirement TraversalRequirement);
```

Execution request model addition:

```csharp
public sealed record ValidationExecutionRequest(
   ValidationProfile Profile,
   ScopeNodeBase RootScope,
   bool IncludeDescendants = true);
```

Behavior:

1. Full project execution is explicit: pass Global root scope with IncludeDescendants = true.
2. Root-only execution is explicit: pass any root scope with IncludeDescendants = false.
3. Subtree execution is explicit: pass Area/Room/other scope root with IncludeDescendants = true.
4. Engine runs applicable rules for the requested scope selection.
5. Rules with TraversalRequirement = FullProjectTraversalRequired are skipped unless request is full-project (Global root + IncludeDescendants = true), and are reported as not applicable in diagnostics metadata.
6. Applicability is evaluated with two independent gates:
   - scope gate (RootScope type plus rule scope applicability capability)
   - traversal gate (IncludeDescendants and TraversalRequirement)
7. A rule that supports Global scope can still be skipped when IncludeDescendants = false if TraversalRequirement is DescendantsRequired or FullProjectTraversalRequired.

## 6.7 Current Rules Mapped To Classification

Initial mapping for migration planning:

1. SingleNode:
   - Required producer notes on object node.
   - Node-local mandatory property checks.
2. Recursive:
   - Duplicate naming families that require grouped scans.
   - Inventoriable global uniqueness checks.
   - Future duplicate ObjectId checks.
   - Some future recursive rules may support subtree execution for Area/Room quality checks.
3. Node applicability constrained examples (orthogonal):
   - Traversal leg isPassable contract checks.
   - IsDoorLinkedToTraversal checks.
   - Linked-action graph integrity.
   - Script diagnostics rules.

## 6.8 Concrete StoryboardDesigner.App Package And Namespace Layout

Initial layout for implementation (designer-only):

1. Folder: StoryboardDesigner.App/Validation/Contracts/
   - Namespace: StoryboardDesigner.App.Validation.Contracts
   - Contains: ValidationIssue, ValidationSeverity, ValidationRuleContext, ValidationRuleMetadata, IValidationRule
2. Folder: StoryboardDesigner.App/Validation/Execution/
   - Namespace: StoryboardDesigner.App.Validation.Execution
   - Contains: ValidationEngine, ValidationRuleRegistry, ValidationProfile, scheduler/dispatch utilities
3. Folder: StoryboardDesigner.App/Validation/Traversal/
   - Namespace: StoryboardDesigner.App.Validation.Traversal
   - Contains: traversal-focused rules and adapters from existing traversal validator path
4. Folder: StoryboardDesigner.App/Validation/Rules/Project/
   - Namespace: StoryboardDesigner.App.Validation.Rules.Project
   - Contains: structural and scope hierarchy rules (starting scope, duplicate names, required fields)
5. Folder: StoryboardDesigner.App/Validation/Rules/Objects/
   - Namespace: StoryboardDesigner.App.Validation.Rules.Objects
   - Contains: object-focused rules (producer notes, inventoriable uniqueness, object id checks)
6. Folder: StoryboardDesigner.App/Validation/Rules/Actions/
   - Namespace: StoryboardDesigner.App.Validation.Rules.Actions
   - Contains: action graph rules and command/action linkage rules
7. Folder: StoryboardDesigner.App/Validation/Rules/Scripting/
   - Namespace: StoryboardDesigner.App.Validation.Rules.Scripting
   - Contains: action script diagnostics and syntax/reference rule wrappers
8. Folder: StoryboardDesigner.App/Validation/Adapters/
   - Namespace: StoryboardDesigner.App.Validation.Adapters
   - Contains: legacy compatibility shims for current save/report/export call sites during migration

Companion tests layout:

1. Folder: StoryboardDesigner.App.Tests/Validation/Rules/
   - one test file per rule class.
2. Folder: StoryboardDesigner.App.Tests/Validation/Execution/
   - registry/profile/scheduler tests.
3. Folder: StoryboardDesigner.App.Tests/Validation/Parity/
   - before/after migration parity fixtures.

## 6.9 Dependency Direction Rules (Enforced By Structure)

Allowed dependencies:

1. Rules -> Contracts only.
2. Engine/Execution -> Contracts and Rules.
3. Adapters -> Engine and legacy call sites.
4. UI/ViewModel call sites -> Adapters/Engine public entry points.

Disallowed dependencies:

1. Contracts depending on WPF/UI/ViewModel types.
2. Rules depending directly on dialogs, windows, or filesystem UI surfaces.
3. Any validation namespace depending on Storyboard.Simulator.
4. Any validation namespace being moved into Storyboard.Shared.
5. Rule implementations depending on mutation-capable services for project editing, persistence, or UI workflows.

Dependency direction summary:

1. Contracts is the innermost layer.
2. Rules are pure evaluation units.
3. Execution orchestrates rules and provides read-only lookup snapshots.
4. Adapters bridge migration-era callers.
5. UI/ViewModel consumes only orchestration boundaries.

## 6.10 Rule Registration Model

Registration goals:

1. Deterministic: same rule set and order for same profile every run.
2. Explicit: no hidden reflection scan required in v1.
3. Testable: profile composition is unit-testable without UI.

### 6.10.1 Registration Components

1. ValidationRuleDescriptor
   - v1 shape is intentionally minimal: wraps rule instance only.
   - registration-time metadata overrides are out of scope in v1.
2. ValidationRuleRegistry
   - single source of truth for registered rules.
   - validates uniqueness of RuleId at startup.
3. ValidationProfileCatalog
   - maps profile names to include/exclude rule sets.

### 6.10.2 Registration Style (v1)

Use explicit code-based registration in a single composition location.

Example:

```csharp
registry.Register(new DuplicateNameInScopeRule());
registry.Register(new MissingProducerNotesRule());
registry.Register(new DuplicateObjectIdRecursiveRule());
```

Rules:

1. Duplicate RuleId registration fails fast.
2. Registry preserves insertion order.
3. Optional explicit Order value may refine order inside same category/execution kind.
4. Rule metadata source of truth is IValidationRule.Metadata; registry does not shadow these fields.

### 6.10.3 Profile Composition

Profiles should be declarative and stable:

1. SaveProfile
   - broad authoring validation set for standard save workflow.
2. ExportGateProfile
   - strict subset used for export-blocking checks.
3. TraversalFocusedProfile
   - traversal-only diagnostics and repair-related checks.
4. ScriptEditorProfile
   - scripting diagnostics subset for action editor usage.
5. ScopedAuthoringProfile
   - subtree-capable rules for room/area "validate here" workflows.

Profile rule selection controls:

1. IncludeByRuleId
2. IncludeByCategory
3. ExcludeByRuleId
4. MinSeverity (optional)

### 6.10.4 Registration Versus Runtime Eligibility

Registration should stay mostly blind/minimal and capture stable capabilities.

Locked v1 split:

1. Registration-time (static):
   - Rule instance reference
   - Deterministic order (insertion order or explicit order)
   - Rule metadata read from IValidationRule.Metadata (no registry override layer)
2. Runtime (dynamic):
   - request scope root
   - IncludeDescendants
   - active profile filters
   - current node under evaluation
   - derived run flags (isFullProject, isRecursivePass)
   - scope-type applicability decision (via rule CanEvaluate)

Design rule:

1. Registry does not need every execution detail up front.
2. Engine asks each candidate rule if it is eligible in current runtime context.
3. Rules can return NotApplicable with reason for observability.
4. Scope type support is decided at runtime by the rule, not declared as required static registration metadata.
5. SupportedCandidateScopeTypes remains rule-owned capability metadata, not a registry field.

## 6.11.5 Eligibility Predicate (Does It Make Sense To Run Here)

The engine should evaluate eligibility through a dedicated predicate instead of hard-coding all routing in one place.

Locked v1 shape:

```csharp
public sealed record ValidationEligibilityContext(
   ValidationExecutionRequest Request,
   ScopeNodeBase CandidateNode,
   ScopeNodeBase RequestRoot,
   bool IsRecursivePass,
   bool IsFullProject,
   ValidationProfile ActiveProfile,
   IReadOnlySet<string> IncludedRuleIds,
   IReadOnlySet<string> ExcludedRuleIds,
   ValidationNodeFacetSet CandidateFacets);

public readonly record struct ValidationNodeFacetSet(
   bool HasActions,
   bool HasTraversalLegs,
   bool HasScriptContent);

public interface IValidationRule
{
    ValidationRuleMetadata Metadata { get; }
   IReadOnlySet<ScopeType> SupportedCandidateScopeTypes { get; }
    ValidationEligibilityResult CanEvaluate(ValidationEligibilityContext eligibility);
    IEnumerable<ValidationIssue> Evaluate(ValidationRuleContext context);
}
```

Capability convention:

1. Empty SupportedCandidateScopeTypes means no scope-type pre-filter (rule can consider any node type).
2. Non-empty set allows fast executor pre-filter before CanEvaluate.
3. CanEvaluate remains authoritative and may still return NotApplicable.

Eligibility inputs that matter in v1:

1. Candidate node scope type.
2. Request root scope type.
3. IncludeDescendants value.
4. IsRecursivePass (derived from IncludeDescendants and scheduler behavior).
5. IsFullProject (derived from request root is Global and IncludeDescendants = true).
6. Rule ExecutionKind.
7. Rule TraversalRequirement.
8. Rule SupportedCandidateScopeTypes capability (fast pre-filter).
9. Profile include or exclude filters (rule ids and categories).
10. Optional: node feature facets (has actions, has traversal legs, has scripts).

Eligibility output (locked taxonomy):

1. Eligible = true or false.
2. SkipReason when false:
   - UnsupportedScopeType
   - NotInProfile
   - ExcludedByRequest
   - DescendantsRequired
   - FullProjectTraversalRequired
   - MissingRequiredFacet

Proposed result contract:

```csharp
public enum ValidationSkipReason
{
   UnsupportedScopeType,
   NotInProfile,
   ExcludedByRequest,
   DescendantsRequired,
   FullProjectTraversalRequired,
   MissingRequiredFacet
}

public sealed record ValidationEligibilityResult(
   bool IsEligible,
   ValidationSkipReason? SkipReason = null,
   string? SkipDetail = null);
```

Notes:

1. Node type + IncludeDescendants + IsFullProject are necessary but not sufficient.
2. Profile filters and required feature facets prevent noisy or wasted rule invocations.
3. Keeping eligibility explicit makes diagnostics easier and reduces hidden coupling.

## 6.11 Validation Run Request Model

Request goals:

1. A single engine API for all callers.
2. Caller controls scope root and profile, not individual rule internals.
3. Output explains what ran, what was skipped, and why.

### 6.11.1 Request Contract

```csharp
public enum ValidationCompletionMode
{
   FullReport,
   StopOnFirstBlocking
}
```

```csharp
public sealed record ValidationExecutionRequest(
    ValidationProfile Profile,
   ScopeNodeBase RootScope,
    bool IncludeDescendants = true,
    IReadOnlyCollection<string>? IncludeRuleIds = null,
    IReadOnlyCollection<string>? ExcludeRuleIds = null,
    ValidationSeverity? MinSeverity = null,
   ValidationCompletionMode? CompletionModeOverride = null,
    string? RequestedBy = null);
```

Meaning:

1. Profile is required baseline.
2. RootScope is required and execution intent is controlled by RootScope scope type plus IncludeDescendants.
3. IncludeRuleIds/ExcludeRuleIds allow targeted diagnostics without new profile creation.
4. CompletionModeOverride allows caller-controlled execution behavior per run:
   - FullReport: execute through full candidate set and return complete issue list.
   - StopOnFirstBlocking: halt as soon as first blocking issue is emitted.
5. RequestedBy captures caller identity (for example SaveProject, ExportCleanJson, ValidateAreaCommand).

### 6.11.2 Result Contract (Run Metadata)

Engine result should include operational metadata:

1. Issues
2. ExecutedRuleIds
3. SkippedRules:
   - reason examples: NotApplicableToSubtree, ExcludedByProfile, ExcludedByRequest, DescendantsRequired, FullProjectTraversalRequired
4. ScopeSummary:
   - root path
   - node count traversed
   - subtree/full-project indicator
5. Completion metadata:
   - EffectiveCompletionMode
   - StoppedEarly (true/false)
   - StopRuleId (when StoppedEarly = true)
   - StopSeverity (when StoppedEarly = true)
   - RemainingRuleCountAtStop (when StoppedEarly = true)

### 6.11.3 Caller Workflows

Save command:

1. Request uses SaveProfile + Global root scope + IncludeDescendants = true.
2. Full rule set runs.
3. Save flow receives typed issues and report payload.

Export gate:

1. Request uses ExportGateProfile + Global root scope + IncludeDescendants = true.
2. Blocking categories only.
3. Default completion mode is StopOnFirstBlocking.
4. Internal callers may override to FullReport for diagnostics/report workflows.

Validate Area/Room action:

1. Request uses ScopedAuthoringProfile + RootScope selected node.
2. Local rules and rules compatible with request traversal requirements execute.
3. FullProjectTraversalRequired rules are skipped with explicit metadata.

Script editor diagnostics:

1. Request uses ScriptEditorProfile + rule include filter for scripting IDs.
2. Returns diagnostics tailored for editor panel use.

### 6.11.3.1 Navigation Tree Invocation UX (Implemented v1)

Primary user invocation model:

1. User right-clicks a node in the navigation tree and selects Run Validation....
2. App opens a compact run-settings dialog with:
   - Scope: Selected node only, Selected node and descendants, Whole project
   - Completion mode: Full report, Stop on first blocking
3. App executes validation using the selected options and writes report output through the validation report window.

Top-level quick-run invocation:

1. Main menu includes Validate Project next to File for frequent full-project reruns.

UX behavior rules:

1. Menu should be available on most scope nodes (Global, Planet, Country, Area, Room, Object, Action) unless a node type is explicitly non-validatable.
2. If no rule is eligible for selected root/profile, show a non-error informational result with executed/skipped summary.
3. Completion mode choice should persist per-user for context-menu validate commands.
4. Validation result should clearly label invocation scope as NodeOnly, Subtree, or FullProject.

### 6.11.3.2 Navigation Tree Validation Indicators (Planned)

Goal:

1. Reuse most-recent validation result data on the model/viewmodel to surface visual alert state in the navigation tree.

Indicator model:

1. Each tree node exposes a validation status summary derived from the latest completed validation run.
2. Summary includes at minimum:
   - HasErrors
   - HasWarnings
   - HighestSeverity
   - IssueCountOnNode
   - IssueCountInDescendants
   - LastValidationRunId (or timestamp)
3. Status is reset/cleared when model changes invalidate prior validation context (or marked stale until rerun).

Tree rendering behavior:

1. Node with direct errors shows error indicator (for example red exclamation).
2. Node with only warnings shows warning indicator (for example amber marker).
3. Ancestor nodes bubble child status using descendant counts (for example subtle aggregate badge).
4. Node tooltip/hover can show concise summary: error count, warning count, last run scope/mode.

Update rules:

1. After each validation run, project issues to node summaries by resolved node path/id.
2. Projection should be deterministic and independent of UI control state.
3. Projection should respect invocation scope:
   - Nodes outside scoped run remain unchanged or marked stale based on selected policy.

Staleness policy (v1):

1. Any edit affecting a node should mark that node and ancestors as ValidationStatusStale.
2. Stale status is visually distinct from active violations.
3. Next validation run for overlapping scope refreshes stale markers with current status.

### 6.11.4 Initial API Surface

```csharp
public interface IValidationEngine
{
    ValidationExecutionResult Execute(ValidationExecutionRequest request, ValidationRuleContext context);
}
```

Adapter entry points (migration period):

1. ExecuteSaveValidation(project)
2. ExecuteExportGateValidation(project)
3. ExecuteScopedValidation(project, rootScope)
4. ExecuteScriptDiagnostics(project, editorContext)

## 6.12 v1 Profile Matrix (Draft)

Legend:

1. Run: rule executes by default in profile.
2. Skip: rule is excluded by default in profile.
3. Run (if C#): rule runs only when the referenced explicit condition key is true.

Profile columns:

1. Save: SaveProfile
2. Export: ExportGateProfile
3. Traversal: TraversalFocusedProfile
4. Script: ScriptEditorProfile
5. Scoped: ScopedAuthoringProfile

| Rule (Draft Id) | Save | Export | Traversal | Script | Scoped | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| PROJ-001 MissingPlanetRule | Run | Skip | Skip | Skip | Run (if C1) | Scoped run only when request root is Global. |
| PROJ-002 InvalidStartingPlanetRule | Run | Skip | Skip | Skip | Run (if C1) | Scoped run only when request root is Global. |
| PROJ-003 InvalidStartingCountryRule | Run | Skip | Skip | Skip | Run (if C1) | Scoped run only when request root is Global. |
| PROJ-004 InvalidStartingAreaRule | Run | Skip | Skip | Skip | Run (if C1) | Scoped run only when request root is Global. |
| PROJ-005 InvalidStartingRoomRule | Run | Skip | Skip | Skip | Run (if C2) | Scoped run when subtree contains the configured starting-room target. |
| NAME-001 DuplicateNameInScopeRule | Run | Skip | Skip | Skip | Run | Subtree-capable duplicate checks in selected scope branch. |
| OBJ-001 MissingProducerNotesRule | Run | Skip | Skip | Skip | Run | Node-level rule; useful at subtree scope. |
| CMD-001 CommandWithoutLinkedActionRule | Run | Skip | Skip | Skip | Run | Node-type-specific in room command context. |
| ACT-001 SynonymRequiresTargetRule | Run | Skip | Run (if C3) | Skip | Run | Traversal profile includes action integrity only when C3 is true. |
| ACT-002 SynonymSelfTargetRule | Run | Skip | Run (if C3) | Skip | Run | Same applicability gate as ACT-001. |
| ACT-003 SynonymMissingTargetRule | Run | Skip | Run (if C3) | Skip | Run | Same applicability gate as ACT-001. |
| ACT-004 LinkedActionSelfTargetRule | Run | Skip | Run (if C3) | Skip | Run | Same applicability gate as ACT-001. |
| ACT-005 LinkedActionMissingTargetRule | Run | Skip | Run (if C3) | Skip | Run | Same applicability gate as ACT-001. |
| ACT-006 LinkedFlowTargetsSynonymRule | Run | Skip | Run (if C3) | Skip | Run | Same applicability gate as ACT-001. |
| ACT-007 SynonymUsedInLinkedFlowRule | Run | Skip | Run (if C3) | Skip | Run | Same applicability gate as ACT-001. |
| ACT-008 LinkedActionCycleRule | Run | Skip | Run (if C3) | Skip | Run | Same applicability gate as ACT-001. |
| TRV-001 TraversalConnectionEmptyRoomIdRule | Run | Run | Run | Skip | Run | Traversal integrity; export-gate blocking. |
| TRV-002 TraversalConnectionSameRoomEndpointsRule | Run | Run | Run | Skip | Run | Traversal integrity; export-gate blocking. |
| TRV-003 TraversalConnectionRoomOutsideAreaRule | Run | Run | Run | Skip | Run | Traversal integrity; export-gate blocking. |
| TRV-004 DuplicateTraversalConnectionPairRule | Run | Run | Run | Skip | Run | Traversal integrity; export-gate blocking. |
| TRV-005 InvalidDiagonalForFourDirectionalRule | Run | Run | Run | Skip | Run | Traversal integrity; export-gate blocking. |
| TRV-006 MissingIsPassableRule | Run | Run | Run | Skip | Run | Traversal leg contract; export-gate blocking. |
| TRV-007 InvalidIsPassableRestrictionRule | Run | Run | Run | Skip | Run | Traversal leg contract; export-gate blocking. |
| TRV-008 InvalidIsPassableDefaultRule | Run | Run | Run | Skip | Run | Traversal leg contract; export-gate blocking. |
| TRV-009 DoorLinkMissingDoorRule | Run | Run | Run | Skip | Run | IsDoorLinkedToTraversal family. |
| TRV-010 DoorLinkMissingIsOpenRule | Run | Run | Run | Skip | Run | IsDoorLinkedToTraversal family. |
| TRV-011 DoorLinkSharedVariableMismatchRule | Run | Run | Run | Skip | Run | IsDoorLinkedToTraversal family. |
| OBJ-002 InventoriableGlobalUniquenessRule | Run | Skip | Skip | Skip | Skip | Full-project-only in v1 (TraversalRequirement = FullProjectTraversalRequired). |
| OBJ-003 DuplicateObjectIdRecursiveRule | Run | Skip | Skip | Skip | Skip | ProjectOnly recursive proof rule for v1 foundation. |
| SCR-001 ScriptUnknownReferenceRule | Skip | Skip | Skip | Run | Run (if C4) | Scoped includes script diagnostics only when C4 is true. |
| SCR-002 ScriptMalformedIndexedReferenceRule | Skip | Skip | Skip | Run | Run (if C4) | Scoped includes script diagnostics only when C4 is true. |
| SCR-003 ScriptIfElseBlockIntegrityRule | Skip | Skip | Skip | Run | Run (if C4) | Scoped includes script diagnostics only when C4 is true. |

Matrix notes:

1. Export profile intentionally focuses on traversal/export-blocking integrity in v1.
2. Save profile remains broad authoring quality validation.
3. Scoped profile prioritizes locally applicable rules and excludes full-project-only traversal requirements.
4. Script profile is editor-focused and intentionally decoupled from save/export in v1.
5. Draft rule IDs are placeholders until allocation pass is completed under the locked RuleId policy.
6. For all rows marked ProjectOnly intent, metadata should use TraversalRequirement = FullProjectTraversalRequired.
7. Condition keys for Run (if C#) cells are fixed for v1:
   - C1: Request.Profile = ScopedAuthoringProfile AND Request.RootScope.ScopeType = Global.
   - C2: Request.Profile = ScopedAuthoringProfile AND subtree traversal contains the configured starting-room target.
   - C3: Request.Profile = TraversalFocusedProfile AND profile option IncludeActionIntegrityRules = true.
   - C4: Request.Profile = ScopedAuthoringProfile AND (Request.IncludeRuleIds contains scripting RuleIds OR profile option IncludeScriptDiagnosticsInScoped = true).

## 6.13 Validation Rule Interface Contract (v1 Draft)

This section defines the internal interface developers implement for each rule.

### 6.13.1 Primary Interface Shape

```csharp
public interface IValidationRule
{
    ValidationRuleMetadata Metadata { get; }
    IReadOnlySet<ScopeType> SupportedCandidateScopeTypes { get; }
    ValidationEligibilityResult CanEvaluate(ValidationEligibilityContext eligibility);
    IEnumerable<ValidationIssue> Evaluate(ValidationRuleContext context);
}
```

Contract intent:

1. Metadata describes stable rule identity and execution semantics.
2. SupportedCandidateScopeTypes is a fast pre-filter only.
3. CanEvaluate decides runtime applicability for this request + node.
4. Evaluate performs validation and returns issues only (no UI actions, no side effects).

### 6.13.2 Method Responsibilities

Metadata:

1. Must be deterministic and constant for the rule class.
2. Must include stable RuleId and default severity.

SupportedCandidateScopeTypes:

1. Empty set means rule can consider any scope type.
2. Non-empty set should include only scope types that can ever be valid candidates.

CanEvaluate:

1. Check request-level eligibility (profile filters, traversal requirements, full-project constraints).
2. Check node-level eligibility (scope type, required node facets).
3. Return NotApplicable with a clear skip reason when false.

Evaluate:

1. Assume eligibility has passed.
2. Execute only rule logic, produce issues with stable paths/messages.
3. Avoid mutating project model state.

### 6.13.3 Developer Authoring Checklist

When implementing a new rule, developer should decide in this order:

1. Identity:
   - What is RuleId?
   - What category and default severity?
2. Execution shape:
   - SingleNode or Recursive?
3. Invocation span:
   - Is this rule full-project-only via FullProjectTraversalRequired, or can it run for root/subtree requests?
4. Traversal requirement:
   - NodeOnly, DescendantsRequired, or FullProjectTraversalRequired?
5. Applicability:
   - Which candidate scope types are worth pre-filtering?
   - What additional CanEvaluate conditions must be true?
6. Lookup needs:
   - Which read-only lookup queries are required for cross-node integrity checks?
   - Which derived indexes should be cached in SharedState for this run?
7. Rule output:
   - What exact issue paths/messages/hints are emitted?
   - Are outputs deterministic across repeated runs?

### 6.13.4 Developer Mental Model

Think of rule implementation as two stages:

1. Stage A: Should I run?
   - Implemented by SupportedCandidateScopeTypes and CanEvaluate.
2. Stage B: If I run, what issues do I emit?
   - Implemented by Evaluate.

Guideline:

1. Keep Stage A cheap and explicit.
2. Keep Stage B pure and deterministic.
3. For cross-node checks, use ValidationRuleContext.Lookup for reads and SharedState for run-scoped caches.
4. Prefer small focused rules over large multi-purpose rules.

### 6.13.5 Optional Base Class (If Useful)

If repeated boilerplate appears, introduce a small base class:

```csharp
public abstract class ValidationRuleBase : IValidationRule
{
    public abstract ValidationRuleMetadata Metadata { get; }
    public virtual IReadOnlySet<ScopeType> SupportedCandidateScopeTypes => EmptyScopeTypeSet.Instance;
    public virtual ValidationEligibilityResult CanEvaluate(ValidationEligibilityContext eligibility) => ValidationEligibilityResult.Eligible();
    public abstract IEnumerable<ValidationIssue> Evaluate(ValidationRuleContext context);
}
```

Constraint:

1. Base class should reduce boilerplate only, not hide rule-specific eligibility logic.

## 6.14 Severity Policy (Locked v1)

Severity goals:

1. Keep severity behavior predictable across workflows.
2. Keep per-rule defaults stable and explicit.
3. Allow narrow workflow-level filtering/escalation without mutating rule metadata.

Policy:

1. Source of truth baseline is `ValidationRuleMetadata.DefaultSeverity`.
2. Registry does not override severity metadata.
3. Request-level `MinSeverity` is a filter threshold, not a rule severity mutation.
4. Optional profile-level severity remap is allowed only through explicit policy config in `ValidationProfile` (not per-call ad hoc mutation).
5. Effective severity computation order:
   - Start with `DefaultSeverity`.
   - Apply profile severity remap if configured.
   - Apply `MinSeverity` filtering to include/exclude final issues.

Script diagnostics default policy:

1. `SCR-001 ScriptUnknownReferenceRule` default severity is Warning in v1.
2. `SCR-002 ScriptMalformedIndexedReferenceRule` default severity is Warning in v1.
3. `SCR-003 ScriptIfElseBlockIntegrityRule` default severity is Error in v1.
4. Save and Scoped workflows may surface these diagnostics; ExportGate excludes script profile rules in v1.

Constraints:

1. Effective severity must be deterministic for the same request + profile.
2. Any remap must be visible in run metadata for diagnostics transparency.
3. Severity overrides do not change RuleId identity or rule execution eligibility.

## 7. Migration Strategy

## Phase 0: Foundation

1. Add validation contracts and engine scaffolding.
2. Add registry and profile support.
3. Preserve current output format compatibility adapter.
4. Introduce classification metadata and execution scheduler.
5. Add execution request model supporting root-scoped runs (project root or selected subtree root).
6. Add explicit registry + profile catalog + request/response metadata plumbing.

Exit criteria:

1. Build green.
2. Existing validation UX still works with adapter path.
3. Engine can run one sample rule for each classification kind.
4. Engine can run both full-project-only and subtree-compatible example rules with correct applicability behavior.
5. Engine exposes executed vs skipped rule metadata for diagnostics/reporting.

## Phase 1: Extract Non-Traversal Legacy Rules

1. Move duplicate-name and starting-scope rules to dedicated classes.
2. Move producer-notes and command-link rules to dedicated classes.
3. Keep old methods as delegating wrappers, then remove once parity verified.
4. Ensure each extracted rule has explicit classification.

Exit criteria:

1. Rule-level tests added.
2. Previous behavior parity validated by targeted tests.

## Phase 2: Align Traversal Rules

1. Convert traversal rules to shared issue contract.
2. Keep TraversalValidationService as facade or adapter during transition.
3. Preserve export hard-gate semantics.

Exit criteria:

1. Traversal-focused regression suite green.
2. No behavior regression in save/export gating.

## Phase 3: Integrate Linked-Action and Script Diagnostics

1. Wrap or extract LinkedActionGraphValidator rules into rule classes.
2. Integrate script diagnostics with rule metadata while preserving warning behavior.
3. Add profile-level routing for editor diagnostics vs save validation.

Exit criteria:

1. Existing editor diagnostics behavior preserved.
2. Unified report pipeline supports these issues.

## Phase 4: Cleanup and Enablement

1. Remove legacy string-only plumbing.
2. Standardize report generation on typed issues.
3. Add contributor docs for creating a new rule class.

Exit criteria:

1. New-rule authoring guide added.
2. No remaining direct legacy validation calls.

Authoring guide location:

1. VALIDATION_RULE_AUTHORING_GUIDE.md

## 8. Testing Strategy

1. Rule unit tests: one test file per rule class.
2. Profile tests: verify which rules run in each workflow.
3. Parity tests: before/after migration snapshots for representative projects.
4. Regression tests for save flow and export hard gates.
5. Guardrail tests for deterministic ordering and stable RuleIds.

## 9. Risks and Mitigations

1. Risk: behavior drift during extraction.
   - Mitigation: parity fixtures and staged migration with adapters.
2. Risk: rule duplication during transition period.
   - Mitigation: profile registry prevents running both legacy and extracted versions simultaneously.
3. Risk: severity inconsistency.
   - Mitigation: Metadata default severity plus centralized mapping policy.
4. Risk: over-coupling UI and validation engine.
   - Mitigation: keep engine UI-agnostic, map to UI contracts at boundaries.

## 10. Decision Log (Needs Alignment)

1. Resolved: RuleId policy uses `^[A-Z]{3}-\\d{3}$`; assignment owned by StoryboardDesigner.App validation maintainers via central registry map.
2. Resolved: validation contracts and rules remain in StoryboardDesigner.App (designer-only boundary).
3. Resolved: severity baseline is DefaultSeverity; script unknown-reference is Warning by default in v1 with optional profile-level remap policy.
4. Whether suppression/waiver support is in scope now or deferred.
5. Whether rule metadata should include auto-fix capability flags.
6. ExecutionKind is single-choice (SingleNode or Recursive), while node scope applicability is multi-value via SupportedCandidateScopeTypes.
7. Scope traversal API shape (shared walker service vs helper extensions).
8. Which existing recursive rules are full-project-only (FullProjectTraversalRequired) versus subtree-compatible in v1.
9. Resolved: v1 supports completion mode by profile default with per-run override; ExportGate defaults to StopOnFirstBlocking, others default to FullReport.

## 10.1 Lock-Off Questions Tracker

Purpose:

1. Persist long-running architecture questions and decisions across sessions.
2. Track status and chosen direction for each lock-off topic.

Status legend:

1. Open: not yet discussed.
2. In Discussion: active decision thread.
3. Locked: decision accepted and reflected in plan/contracts.

| Topic | Status | Decision / Current Direction | Notes |
| --- | --- | --- | --- |
| 1. Overlap: EvaluationSpan vs TraversalRequirement | Locked | Collapse to TraversalRequirement only; derive invocation scope from request (RootScope + IncludeDescendants). | Full-project-only expressed by FullProjectTraversalRequired. |
| 2. Final IValidationRule interface surface | Locked | Metadata + SupportedCandidateScopeTypes + CanEvaluate + Evaluate. | Keep pre-filter cheap; CanEvaluate authoritative; Evaluate pure. |
| 3. ValidationEligibilityContext final fields + skip reason taxonomy | Locked | Context includes request/root/candidate/profile/filters/derived flags/facets; skip reasons standardized. | Supports deterministic skip reporting. |
| 4. ValidationRuleContext final fields and read-only guarantees | Locked | Project/candidate/request/profile/flags/shared-state/cancellation finalized; shared state constrained to deterministic caching only. | Enforces pure rule evaluation and side-effect boundaries. |
| 5. Registration metadata minimum set | Locked | Registry stores rule instance + deterministic order only; rule identity/execution metadata comes from IValidationRule.Metadata. | No registration-time metadata override layer in v1. |
| 6. Profile matrix Conditional cells -> explicit criteria | Locked | Replaced Conditional cells with Run (if C#) and fixed C1-C4 criteria in matrix notes; OBJ-002 Scoped changed to Skip for v1 consistency. | Matrix behavior is now deterministic and testable. |
| 7. RuleId policy (format, allocation ownership) | Locked | Pattern `^[A-Z]{3}-\\d{3}$`; family prefixes fixed; allocation owned by StoryboardDesigner.App validation maintainers; no ID reuse. | RuleId is treated as external contract identifier. |
| 8. Severity policy and overrides (especially script diagnostics) | Locked | DefaultSeverity is baseline; MinSeverity filters; optional profile remap policy; script unknown-reference defaults to Warning in v1. | Preserves consistency while allowing explicit workflow policy. |
| 9. Fail-fast policy by profile | Locked | Completion mode supports FullReport and StopOnFirstBlocking; profile default applies unless request override is provided. | ExportGate default StopOnFirstBlocking; Save/Scoped/Script/Traversal default FullReport. |
| 10. Phase 0 first extraction slice definition | Locked | Slice A selected: DuplicateNameInScopeRule, InvalidStartingScopeRule, MissingProducerNotesRule, CommandWithoutLinkedActionRule under SaveProfile adapter path. | Excludes traversal/script/linked-graph extraction in first slice to minimize risk. |

## 10.2 Lock-Off Workflow (One-By-One)

For each topic:

1. Clarify intent and boundary.
2. List options.
3. Pick recommended option.
4. Lock decision and update affected sections.
5. Add follow-up implementation tasks.

## 10.3 Topic 2 Lock-Off: Final IValidationRule Interface Surface

Status: Locked

Decision:

1. Final v1 interface shape is:

```csharp
public interface IValidationRule
{
   ValidationRuleMetadata Metadata { get; }
   IReadOnlySet<ScopeType> SupportedCandidateScopeTypes { get; }
   ValidationEligibilityResult CanEvaluate(ValidationEligibilityContext eligibility);
   IEnumerable<ValidationIssue> Evaluate(ValidationRuleContext context);
}
```

2. SupportedCandidateScopeTypes is a fast pre-filter only.
3. CanEvaluate is the authoritative runtime eligibility decision point.
4. Evaluate runs only after eligibility and must remain pure/deterministic.

Follow-up tasks:

1. Ensure all plan snippets reference this exact interface.
2. Add base-class/template scaffolding that preserves this contract.

## 10.4 Topic 3 Lock-Off: ValidationEligibilityContext + Skip Reasons

Status: Locked

Decision:

1. ValidationEligibilityContext includes:
   - Request
   - CandidateNode
   - RequestRoot
   - IsRecursivePass
   - IsFullProject
   - ActiveProfile
   - IncludedRuleIds
   - ExcludedRuleIds
   - CandidateFacets
2. CandidateFacets is a compact, precomputed node capability snapshot:
   - HasActions
   - HasTraversalLegs
   - HasScriptContent
3. Skip reason taxonomy is fixed for v1:
   - UnsupportedScopeType
   - NotInProfile
   - ExcludedByRequest
   - DescendantsRequired
   - FullProjectTraversalRequired
   - MissingRequiredFacet
4. CanEvaluate returns ValidationEligibilityResult with optional SkipDetail for diagnostics.

Follow-up tasks:

1. Mirror skip-reason enum in report metadata adapters.
2. Add execution tests that assert deterministic SkipReason values.

## 10.5 Topic 4 Lock-Off: ValidationRuleContext + Read-Only Guarantees

Status: Locked

Decision:

1. ValidationRuleContext includes:
   - Project
   - CandidateNode
   - RequestRoot
   - Request
   - ActiveProfile
   - IncludeDescendants
   - IsRecursivePass
   - IsFullProject
   - SharedState
   - CancellationToken
2. SharedState is typed via IValidationSharedState and supports deterministic cache retrieval only.
3. Rules treat ValidationRuleContext as read-only and must not mutate project/request/profile graphs.
4. Evaluate is side-effect-free: no UI actions, no persistence writes, no external calls.

Follow-up tasks:

1. Add rule-level tests asserting deterministic output for repeated Evaluate calls with identical context.
2. Add architecture guardrail tests that rule implementations do not call UI or persistence services.

## 10.6 Topic 5 Lock-Off: Registration Metadata Minimum Set

Status: Locked

Decision:

1. v1 registration is intentionally minimal:
   - Rule instance
   - Deterministic order (insertion or explicit order)
2. Rule identity/execution metadata is owned by IValidationRule.Metadata:
   - RuleId
   - Category
   - DefaultSeverity
   - ExecutionKind
   - TraversalRequirement
3. Registration-time metadata overrides are out of scope for v1.
4. Scope applicability capability remains rule-owned (SupportedCandidateScopeTypes), not registry-owned.

Follow-up tasks:

1. Keep ValidationRuleDescriptor slim and remove any planned override fields from v1 scaffolding.
2. Add registry tests asserting metadata is sourced from rule instances without shadow copies.

## 10.7 Topic 6 Lock-Off: Profile Matrix Conditional Cells

Status: Locked

Decision:

1. Matrix no longer uses ambiguous "Conditional" cells.
2. Conditional behavior is encoded as explicit "Run (if C#)" entries.
3. Condition keys are fixed in matrix notes (C1-C4) for deterministic execution behavior.
4. OBJ-002 in ScopedAuthoringProfile is explicitly Skip in v1 to align with full-project-only requirement.

Follow-up tasks:

1. Add profile/execution tests that assert C1-C4 behavior for representative requests.
2. Ensure profile options IncludeActionIntegrityRules and IncludeScriptDiagnosticsInScoped are represented in profile contract/options.

## 10.8 Topic 7 Lock-Off: RuleId Policy (Format + Ownership)

Status: Locked

Decision:

1. Canonical RuleId format is `^[A-Z]{3}-\d{3}$`.
2. Prefix family set in v1: PROJ, NAME, OBJ, CMD, ACT, TRV, SCR.
3. Allocation ownership is with StoryboardDesigner.App validation maintainers.
4. Allocation strategy: next available number in family; retired IDs are never reused.
5. RuleId is a contract identifier and remains stable across refactors/renames.

Follow-up tasks:

1. Add a registry test that all registered RuleIds match the canonical pattern.
2. Add a registry test that RuleIds are unique and non-reused within each family.

## 10.9 Topic 8 Lock-Off: Severity Policy + Overrides

Status: Locked

Decision:

1. Rule default severity source is `ValidationRuleMetadata.DefaultSeverity`.
2. Registry-level severity overrides are out of scope for v1.
3. `ValidationExecutionRequest.MinSeverity` is a filtering threshold only.
4. Optional severity remap is allowed only as explicit profile policy (deterministic and testable).
5. `SCR-001 ScriptUnknownReferenceRule` default severity is Warning in v1.

Follow-up tasks:

1. Add tests that effective severity is deterministic across Save/Scoped/Script profiles.
2. Add run-metadata fields that expose applied severity remap/filter decisions.

## 10.10 Topic 9 Lock-Off: Completion Mode (Fail-Fast By Profile)

Status: Locked

Decision:

1. v1 defines two completion modes:
   - FullReport
   - StopOnFirstBlocking
2. Completion mode is determined by profile default unless `ValidationExecutionRequest.CompletionModeOverride` is supplied.
3. Default profile policy in v1:
   - ExportGateProfile: StopOnFirstBlocking
   - SaveProfile: FullReport
   - ScopedAuthoringProfile: FullReport
   - ScriptEditorProfile: FullReport
   - TraversalFocusedProfile: FullReport
4. User-facing validation commands may expose mode selection for run-time control.
5. Engine must emit completion metadata indicating if/where an early stop occurred.

Follow-up tasks:

1. Add execution tests for both modes across at least Save and ExportGate profiles.
2. Add run metadata assertions for StoppedEarly, StopRuleId, and RemainingRuleCountAtStop.

## 10.11 Topic 10 Lock-Off: Phase 0 First Extraction Slice

Status: Locked

Decision:

1. First extraction slice is Slice A (non-traversal, high-value, low-coupling):
   - DuplicateNameInScopeRule
   - InvalidStartingScopeRule
   - MissingProducerNotesRule
   - CommandWithoutLinkedActionRule
2. Slice A runs through SaveProfile path first behind compatibility adapter wiring.
3. Slice A explicitly excludes traversal topology/leg rules, linked-action graph cycles, and script diagnostics extraction.
4. DuplicateObjectIdRecursiveRule remains in follow-on slice after Slice A parity is proven.
5. Immediate next slice after Slice A parity is Slice B (user-priority action/script validation):
   - Action integrity rules (ACT-001 through ACT-008)
   - Echo/script variable reference validity rules with SCR-001 as first priority
   - Related script structural checks (SCR-002, SCR-003)

Acceptance gates for Slice A:

1. Rule-level unit tests for all four extracted rules.
2. Save workflow parity tests for representative fixtures.
3. No regression in existing save validation UX/report structure.
4. Deterministic RuleId ordering verified for SaveProfile execution.

Follow-up tasks:

1. Implement Slice A rule classes + registry wiring + adapter delegation.
2. Add a focused parity test fixture pack for starting-scope, duplicates, producer-notes, and command-link checks.
3. Start Slice B immediately after Slice A parity gates are green, with SCR-001 extracted first.

## 11. Initial Implementation Backlog (Proposed)

1. Add Validation contracts + engine + registry skeleton.
2. Add ValidationExecutionKind and scheduler support.
3. Implement Slice A extracted rule classes (locked first extraction slice):
   - DuplicateNameInScopeRule
   - InvalidStartingScopeRule
   - MissingProducerNotesRule
   - CommandWithoutLinkedActionRule
4. Implement Slice B action/script extraction immediately after Slice A parity (user-priority):
   - ACT-001 through ACT-008
   - SCR-001 first (echo/script reference validity)
   - SCR-002 and SCR-003
5. Add first recursive proof rule after Slice B parity:
   - DuplicateObjectIdRecursiveRule (new)
6. Add first subtree-capable recursive proof rule:
   - SubtreeDuplicateNameRecursiveRule (Area/Room scoped applicability)
7. Add profile catalog (Save, ExportGate, TraversalFocused, ScriptEditor, ScopedAuthoring).
8. Wire save validation to engine profile behind compatibility adapter.
9. Add baseline rule authoring template and tests.
10. Implement navigation tree context-menu validation invocation:
   - Single Run Validation... entry
   - Run-settings dialog for scope and completion mode
   - Top-level Validate Project quick-run menu item
11. Persist per-user completion mode preference for context-menu validate commands.
12. Implement validation status projection to navigation tree nodes using most-recent validation results.
13. Add tree indicator UX states (error/warning/stale/bubbled descendant status) with deterministic update rules.
14. TBD (deferred): define a manual validation test plan for intentionally invalid states once Slice A/Slice B execution paths are available for reliable exercise.

## 12. Definition of Done for This Plan

This plan moves from Draft to Active when:

1. Decision Log items 1-3 are resolved.
2. Phase 0 scope is approved.
3. First extraction slice is selected.
4. Test strategy acceptance is confirmed.

## 13. Phased Implementation Plan (Execution)

This section is the implementation sequence to execute the locked architecture with explicit review gates.

### Phase 1: Foundation Scaffolding

Goal:

1. Stand up contracts, engine skeleton, and registry/profile composition without broad rule extraction.

Deliverables:

1. Validation contracts and execution scaffolding in StoryboardDesigner.App validation namespace.
2. ValidationRuleRegistry + ValidationProfileCatalog composition path.
3. ValidationExecutionRequest/Result plumbing with completion-mode metadata.
4. Adapter entry point wired for save validation path.

Exit criteria:

1. Solution builds cleanly.
2. Existing save flow remains behavior-compatible (adapter mode).
3. Engine can execute with empty or stub rule set without runtime errors.

### Phase 2: Vertical Slice Review (Single Rule End-To-End)

Goal:

1. Provide one clear, inspectable example of rule authoring, registration, and execution.

Selected demo rule for checkpoint:

1. MissingProducerNotesRule (single-node, low-coupling, easy to reason about).

What will be reviewable in code at this checkpoint:

1. Rule authoring:
   - a concrete rule class implementing IValidationRule.
   - metadata, eligibility, and Evaluate logic in one place.
2. Rule registration:
   - explicit registry registration line for MissingProducerNotesRule.
   - inclusion in SaveProfile rule selection.
3. Rule execution:
   - save adapter invokes ValidationEngine with SaveProfile request.
   - run result includes issue output + executed/skipped metadata.
4. Tests:
   - rule unit test for positive and negative cases.
   - one engine/profile integration test proving registered rule executes.

Review gate (explicit hold point for your review):

1. Stop after this phase and review:
   - authored rule class,
   - registry registration,
   - one runnable validation path showing emitted issue.

### Phase 3: Complete Slice A Extraction

Goal:

1. Finish remaining Slice A rules after checkpoint approval.

Scope:

1. DuplicateNameInScopeRule
2. InvalidStartingScopeRule
3. CommandWithoutLinkedActionRule

Exit criteria:

1. Slice A parity tests pass.
2. Deterministic RuleId ordering validated.
3. No save UX/report regressions.

### Phase 4: Slice B Priority Extraction (Actions + Echo Script References)

Goal:

1. Implement user-priority action and script validation slice immediately after Slice A parity.

Scope:

1. ACT-001 through ACT-008.
2. SCR-001 first (echo/script variable reference validity).
3. SCR-002 and SCR-003.

Exit criteria:

1. ScriptEditor and Scoped profile routing validated for SCR rules.
2. Action integrity rules execute with deterministic gating.
3. Severity policy behavior verified (SCR defaults warning unless explicit profile remap).

### Phase 5: Follow-On Recursive/Traversal Expansion

Goal:

1. Add recursive proof and subtree-capable follow-on rules after Slice B stability.

Scope:

1. DuplicateObjectIdRecursiveRule.
2. SubtreeDuplicateNameRecursiveRule.

Exit criteria:

1. Recursive execution path and traversal requirement behavior validated.
2. Export/save profile behavior remains consistent with locked completion mode policy.

### Phase 6: Deferred Manual Testing Plan Activation

Goal:

1. Activate manual test plan item once Slice A and Slice B are runnable in-app.

Activation conditions:

1. End-to-end validation command path is stable for Save + ScriptEditor profiles.
2. At least one deterministic invalid-state fixture exists per priority rule family.

Output:

1. Manual test matrix with expected RuleIds, severities, and completion-mode behavior.

## 14. Infrastructure Completion Checklist (No New Rule Families)

This checklist is the close-out sequence before adding net-new validation families.

1. Engine request and context contract completion:
   - Introduce root scope + include descendants + profile selection in engine request.
   - Migrate rule signature from Evaluate(ProjectModel) to Evaluate(ValidationRuleContext).
   - Include read-only validation lookup service on ValidationRuleContext for cross-node integrity reads.
2. Eligibility and skip metadata completion:
   - Add ValidationEligibilityContext and ValidationEligibilityResult in active execution path.
   - Emit deterministic skip reasons for ineligible rules.
3. Profile catalog completion:
   - Add explicit profile catalog composition (Save, ExportGate, TraversalFocused, ScriptEditor, ScopedAuthoring).
   - Route UI callers through profile-backed requests instead of ViewModel-side post-filtering.
4. Result metadata completion:
   - Expand execution result with skipped rules, scope summary, and completion metadata (StoppedEarly, StopRuleId, RemainingRuleCountAtStop).
5. ViewModel simplification completion:
   - Move scope filtering and completion truncation from MainWindowViewModel into engine execution orchestration.
6. Parity and guardrail completion:
   - Add parity tests for existing extracted rules under new request/profile model.
   - Keep architecture guardrail tests green and ensure deterministic RuleId order assertions remain stable.
   - Add guardrails proving rules consume read-only lookup projections and do not call mutation-capable services.
7. Plan close-out gate:
   - When items 1-6 are green in CI and manual smoke checks pass, mark plan status Complete and begin next validation family planning.
