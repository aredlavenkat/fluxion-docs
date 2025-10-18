# Tooling & IDE Support

This page explains how to build productive tooling around the rule engine, including VS Code extensions, language servers, and CI automation. The goal is to make authoring rules as ergonomic as editing source code.

## JSON schema & completion

- Use the [DSL Reference](dsl-reference.md) schema to power inline validation.
- Provide completion for common stage operators and actions. Fetch action names by interrogating your deployment registry or by reading `RuleActionRegistry.ruleActions()` at runtime.
- Suggest salience values based on best-practice ranges (see [Best Practices](best-practices.md)).

### VS Code snippet example

```json
{
  "Add Rule": {
    "scope": "json",
    "prefix": "rule",
    "body": [
      "{",
      "  \"name\": \"${1:Rule name}\",",
      "  \"salience\": ${2:100},",
      "  \"stages\": [",
      "    { \"$match\": { \"${3:field}\": \"${4:value}\" } }",
      "  ],",
      "  \"actions\": [\"${5:action-name}\"]",
      "}",
      "$0"
    ]
  }
}
```

## Language server tips

- Run `RuleDslParser.parseWithLints` in the background to convert lint results into diagnostics.
- Use the `context` data from `RuleLint` (e.g. `stageIndex`) to place squiggles on the correct stage.
- Provide code actions that insert missing stages or fix operator casing.

## Java editor assistance

For teams writing rules in code:

- Publish live templates that scaffold the builder pattern (`RuleDefinition.builder(…)`).
- Offer inspections that warn when `RuleDefinition.builder` is created without calling `.condition(...)`.
- Use static analysis to ensure `RuleSet.Builder.addRule` is followed by `.build()` before evaluation.

## CLI and CI integrations

- Add a `rules lint` CLI command that wraps `parseWithLints` and prints lints in SARIF or GitHub annotation format.
- Integrate linting and unit tests into pull-request workflows; fail the build when lints are present.
- Generate summary reports listing salience, actions, and metadata for every rule.

## Debug trace visualisation

- Build a viewer that formats `RuleDebugStageTrace` entries as a table. Include stage index, operator, input size, output size, and diff summary.
- For VS Code, render the trace in a WebView when the user runs a “Test rule” command.

## Example VS Code extension workflow

1. **Activation** – extension activates on `rules.json` files.
2. **Schema association** – registers the JSON schema and snippets above.
3. **Lint task** – exposes a command `fluxion.rules.lint` that invokes a local CLI.
4. **Test command** – prompts for a sample JSON document, sends it to a locally running rule service, then shows matches and debug trace in the UI.
5. **Action registry sync** – optionally polls a dev server for available action names and hot reloads completion lists.

A clear tooling story makes the rule engine approachable for both humans and AI assistants. Use these building blocks to craft bespoke authoring experiences for your teams.
