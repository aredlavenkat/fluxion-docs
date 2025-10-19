# Authoring Rule Sets

Rules are described with a simple JSON DSL that mirrors MongoDB-style aggregation pipelines. A **rule set** groups related rules and provides shared metadata and hooks.

```json
{
  "id": "orders",
  "name": "Order Rules",
  "version": "1.0.0",
  "metadata": {
    "team": "risk"
  },
  "rules": [
    {
      "id": "rule-1",
      "name": "Active high value order",
      "description": "Flag active orders above $500",
      "salience": 50,
      "stages": [
        { "$match": { "status": "active" } },
        { "$match": { "total": { "$gte": 500 } } }
      ],
      "actions": ["mark-active"]
    }
  ]
}
```

## Top-level structure

| Field | Purpose |
| --- | --- |
| `id`, `name`, `version` | Optional identifiers used for reporting and lifecycle tooling. |
| `metadata` | Arbitrary key/value pairs carried into evaluation results. |
| `rules` | Ordered array of rule definitions. The engine sorts by `salience` (highest first) at build time. |

## Rule definition fields

| Field | Purpose |
| --- | --- |
| `id`, `name`, `description` | Human readable identifiers surfaced in traces and passes. |
| `salience` | Priority value (default `0`). Higher values execute sooner. |
| `stages` | Required array describing the aggregation pipeline. Empty pipelines are rejected. |
| `actions` | Optional list of action names. Actions must be registered in `RuleActionRegistry`. |
| `metadata` | Rule-specific metadata copied into pass contexts. |

## Building rule sets programmatically

Use the fluent builders when constructing rules directly in Java:

```java
RuleDefinition rule = RuleDefinition.builder("High value")
    .id("rule-1")
    .salience(100)
    .condition(RuleCondition.pipeline(List.of(
        new Stage(Map.of("$match", Map.of("status", "active"))),
        new Stage(Map.of("$match", Map.of("total", Map.of("$gte", 500))))
    )))
    .addAction(RuleActionRegistry.resolve("mark-active").orElseThrow())
    .build();

RuleSet ruleSet = RuleSet.builder()
    .id("orders")
    .version("1.0.0")
    .addRule(rule)
    .build();
```

DSL payloads can be parsed using `RuleDslParser`. Call `parseWithLints` to surface validation warnings without throwing exceptions, or use `parse` for strict behaviour:

```java
RuleDslParser parser = new RuleDslParser();
RuleParseResult result = parser.parseWithLints(jsonPayload);
if (result.hasLints()) {
    // surface lint messages to authors
}
RuleSet ruleSet = result.ruleSet();
```

## Referencing actions and hooks

- Actions are referenced by name in the DSL. They must exist in `RuleActionRegistry` (either registered manually or discovered via the SPI). If the name is missing, parsing fails.
- When using the builder API you can call `addAction(RuleAction)` directly or resolve by name: `RuleActionRegistry.resolve("mark-active").orElseThrow()`.
- Hooks are typically attached in Java using `addHookByName(String)`. DSL support for named hooks can be layered on top of this API by your tooling.

## Sorting and conflicts

Rules are automatically sorted by salience (descending). Duplicate salience values generate a validation error for the conflicting rules. Missing stages or unsupported operators are also flagged during validation—see the [Validation & Linting](validation.md) guide for details.

## Authoring checklist

1. Choose a descriptive rule name and populate `description`.
2. Assign a salience value from your team's range (see [Best Practices](best-practices.md)).
3. Define a minimal pipeline—avoid heavy logic in a single stage.
4. Attach actions by name or via the builder. Ensure actions are registered in the application.
5. Add metadata fields such as `owner`, `runbookUrl`, and `version`.
6. Run linting via `parseWithLints` before committing.
