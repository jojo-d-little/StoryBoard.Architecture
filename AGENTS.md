# Agent Authoring Rules

These rules apply to all agent-authored changes in this repository.

## XML Documentation Requirement

- For every new or modified C# contract/interface type, add XML `<summary>` comments on the type.
- For every public property on new or modified C# contract/interface types, add XML `<summary>` comments.
- For every public method on new or modified C# interfaces, add XML `<summary>`, `<param>`, and `<returns>` comments.
- Treat missing XML comments in these cases as a blocking issue and fix before concluding work.
