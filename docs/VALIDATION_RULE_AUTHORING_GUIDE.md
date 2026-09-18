# Validation Rule Authoring Guide

This guide explains how to add a new validation rule in StoryboardDesigner.App.

## 1. Create A Rule Class

Place the new rule in one of these folders:

1. StoryboardDesigner.App/Validation/Rules/Project/
2. StoryboardDesigner.App/Validation/Rules/Objects/
3. StoryboardDesigner.App/Validation/Rules/Actions/
4. StoryboardDesigner.App/Validation/Rules/Scripting/

Implement IValidationRule with:

1. Stable RuleId in metadata (format: AAA-###)
2. Default severity and category
3. SupportedCandidateScopeTypes pre-filter
4. Evaluate logic that emits deterministic ValidationIssue values

Example skeleton:

```csharp
public sealed class ExampleRule : IValidationRule
{
    public IReadOnlySet<ScopeType> SupportedCandidateScopeTypes { get; } =
        new HashSet<ScopeType> { ScopeType.Area };

    public ValidationRuleMetadata Metadata { get; } = new(
        RuleId: "EXM-001",
        Title: "Example Rule",
        DefaultSeverity: ValidationSeverity.Error,
        Category: "Example");

    public IEnumerable<ValidationIssue> Evaluate(ValidationRuleContext context)
    {
        // emit issues
        yield break;
    }
}
```

## 2. Register The Rule

Register it in validation composition:

1. Main validation registry in StoryboardDesigner.App/ViewModels/MainWindowViewModel.FileCommands.cs
2. Any other focused composition paths (if added later)

Rules are executed in deterministic registration order.

## 3. Add Tests

Add tests in StoryboardDesigner.App.Tests:

1. A dedicated rule behavior test file (positive and negative cases)
2. A registration/execution smoke assertion in ValidationEngineRegistrationTests

Minimum expected assertions:

1. RuleId is emitted as expected
2. Severity is correct
3. Path and description are stable and meaningful

## 4. Authoring Constraints

1. Keep Evaluate pure and deterministic.
2. Do not mutate project model state inside rules.
3. Use ValidationRuleContext.Lookup for cross-node relationships.
4. Prefer small focused rules instead of wide multi-purpose checks.

## 5. RuleId Policy

1. Use pattern ^[A-Z]{3}-\d{3}$.
2. RuleId is a stable contract identifier.
3. Do not reuse retired IDs.

## 6. Suggested Workflow

1. Create rule class.
2. Register rule.
3. Add focused tests.
4. Run: dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "<NewRuleTests>|ValidationEngineRegistrationTests"
5. Run full suite before merge.
