# Validation & Linting

Robust validation prevents broken rules from reaching production. SrotaX provides two complementary mechanisms:

## Builder-time validation

Both `RuleDefinition.Builder.build()` and `RuleSet.Builder.build()` call `RuleValidator` which currently enforces:

- Each rule must define at least one pipeline stage.
- No stage may be empty (missing an operator).
- Every referenced operator must exist in `StageRegistry`.
- Rule sets may not contain duplicate salience values across rules.

Failures raise `RuleValidationException` with a human-readable message.

## Static lint collection

`RuleLintCollector` inspects either Java definitions or raw DSL JSON and emits structured `RuleLint` entries describing issues:

- `MISSING_STAGE` – rule contains no stages or an empty entry.
- `UNSUPPORTED_OPERATOR` – references an operator that is not registered.
- `DUPLICATED_SALIENCE` – multiple rules share the same salience.

Use `RuleDslParser.parseWithLints` to collect lints without immediately throwing exceptions. This is ideal for authoring tools that want to display warnings inline.

```java
RuleParseResult result = parser.parseWithLints(json);
if (result.hasLints()) {
    for (RuleLint lint : result.lints()) {
        // surface lint.type(), lint.message(), lint.context()
    }
}
```

The same collector backs the builder-time validation, ensuring consistent behaviour between authoring-time linting and runtime enforcement.

## Recommended workflow

1. Parse incoming rule definitions with `parseWithLints` and surface all lint messages to authors.
2. If no lints are returned, persist or deploy the rule set as needed.
3. At runtime, rely on the builders' strict validation to catch any regressions introduced through programmatic changes.

## Handling lint metadata

Each `RuleLint` carries a `context()` map that includes helpful pointers:

| Key | Meaning |
| --- | --- |
| `stageIndex` | Zero-based index of the stage that triggered the lint. |
| `operator` | Operator name (for `UNSUPPORTED_OPERATOR`). |
| `salience` | Conflicting salience value (for `DUPLICATED_SALIENCE`). |
| `conflictsWith` | Name of the existing rule with the same salience. |

Map these keys to editor ranges or structured logs so users can quickly remediate.

## Custom validation

If you need organisation-specific checks (e.g. required metadata fields):

1. Parse the rule set using `RuleDslParser`.
2. Run your own validator on the resulting `RuleSet` or `RuleDefinition` objects.
3. Wrap failures in a custom `RuleLint` extension or convert them to your own diagnostic format.

Because `RuleSet` is immutable you can safely cache validated instances across requests.
