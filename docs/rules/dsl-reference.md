# DSL Reference

The rule DSL is JSON-based and intentionally mirrors MongoDB aggregation syntax. This reference documents every field recognised by `RuleDslParser` so that tooling, linters, and schema validators can target the correct structure.

## Rule set schema

```json
{
  "id": "string?",
  "name": "string?",
  "version": "string?",
  "metadata": { "string": "any" }?,
  "hooks": ["string"]?,        // optional when using hook registry shortcuts (future)
  "rules": [ RuleDefinition ]
}
```

### Rule definition

```json
{
  "id": "string?",
  "name": "string" (default: "rule"),
  "description": "string?",
  "salience": "int" (default: 0),
  "stages": [ StageObject ],
  "actions": [ "string" ]?,
  "metadata": { "string": "any" }?
}
```

### Stage object

A stage must be a single-key object whose key is a stage operator (e.g. `$match`). The value is the operator specification.

```json
{ "$match": { "status": "active" } }
```

If you need to reuse pipelines, construct them in Java and attach them via the builder APIs.

### JSON schema (draft 7)

If you want strict schema validation, adapt the following snippet:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["rules"],
  "properties": {
    "id": {"type": "string"},
    "name": {"type": "string"},
    "version": {"type": "string"},
    "metadata": {"type": "object"},
    "rules": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["stages"],
        "properties": {
          "id": {"type": "string"},
          "name": {"type": "string"},
          "description": {"type": "string"},
          "salience": {"type": "integer"},
          "stages": {
            "type": "array",
            "items": {
              "type": "object",
              "minProperties": 1,
              "maxProperties": 1
            }
          },
          "actions": {
            "type": "array",
            "items": {"type": "string"}
          },
          "metadata": {"type": "object"}
        }
      }
    }
  }
}
```

## Programmatic builder equivalence

| DSL field | Builder call |
| --- | --- |
| `id` | `RuleDefinition.Builder.id(String)` |
| `name` | supplied via `RuleDefinition.builder(name)` |
| `description` | `RuleDefinition.Builder.description(String)` |
| `salience` | `RuleDefinition.Builder.salience(int)` |
| `stages` | `RuleCondition.pipeline(List<Stage>)` |
| `actions` | `RuleDefinition.Builder.addAction(RuleAction)` / `addActions(List)` |
| `metadata` | `RuleDefinition.Builder.metadata(Map)` |

Rule-set level metadata and hooks follow the same pattern: `RuleSet.Builder.metadata(...)`, `addHook`, and `addHookByName`.

## Validation rules

Refer to [Validation & Linting](validation.md) for the precise rules enforced by `RuleValidator` and the lint collector. In summary:

- `stages` must be present and non-empty.
- Each stage must contain exactly one operator.
- Operators must exist in `StageRegistry` (built-ins or contributed via SPI).
- Salience values must be unique within a rule set.

## Tooling tips

- Provide auto-complete for common stage operators (e.g. `$match`, `$project`), leveraging the same catalogue used for aggregator docs.
- Offer snippets that expand to `"actions": [ "<action-name>" ]` and `"metadata": {}`.
- Surface lint messages returned by `parseWithLints` inline; they contain `context` keys such as `stageIndex` and `operator` which can map to editor diagnostics.
