# Testing & Debugging Rules

Reliable rules need automated tests and diagnostics. Fluxion offers several tools to make this straightforward.

## Unit testing with RuleEngine

Embed the rule engine in your test suite and supply documents that exercise success and failure paths.

```java
RuleDefinition rule = RuleDefinition.builder("High value")
    .condition(RuleCondition.pipeline(List.of(
        new Stage(Map.of("$match", Map.of("status", "active"))),
        new Stage(Map.of("$match", Map.of("total", Map.of("$gte", 500))))
    )))
    .build();

RuleSet ruleSet = RuleSet.builder().addRule(rule).build();

RuleEngine engine = new RuleEngine();
List<RuleEvaluationResult> results = engine.execute(List.of(new Document(Map.of(
    "status", "active",
    "total", 750
))), ruleSet);

assertFalse(results.getFirst().passes().isEmpty());
```

Use dedicated tests to verify negative cases (documents that should not pass) and to assert that actions and hooks mutate context as expected.

## Debug traces in tests

Enable debug mode to capture stage-level traces:

```java
RuleEvaluationResult result = engine.evaluate(List.of(document), ruleSet, true).getFirst();
RuleExecutionContext context = result.ruleContexts().getFirst();
for (RuleDebugStageTrace trace : context.debugTrace()) {
    // inspect trace.operator(), trace.inputs(), trace.outputs()
}
```

Capturing traces is helpful when rules become complex—store the traces as golden files or log them when a test fails.

## Static lint assertions

When testing DSL ingestion, call `RuleDslParser.parseWithLints` and assert that no lints are returned. This mimics the behaviour of authoring tools and guarantees that the definitions are valid before you deploy them.

```java
RuleParseResult result = new RuleDslParser().parseWithLints(jsonPayload);
assertFalse(result.hasLints(), () -> "Unexpected lint: " + result.lints());
```

## CI recommendations

- Run `mvn test` (or your build equivalent) on each change; the module-level tests cover the rule engine logic.
- Add dedicated suites for your domain-specific rules using the strategies above.
- Surface lint messages as part of pull-request feedback to catch configuration issues early.

## Optional: golden dataset strategy

For complex rule sets, you can maintain a golden dataset (JSON documents + expected outcomes):

1. Store documents in a version-controlled directory (`rulesets/orders/examples/*.json`).
2. Write a test harness that loads each document, runs the rule engine, and compares passes against expected values recorded in YAML/JSON.
3. When rules change, update the golden files as part of the review so diffs are easy to inspect.

## Optional: property-based testing

Use property-based frameworks (jqwik / QuickTheories) to generate documents and assert invariants:

- **Idempotence** – executing the same rule twice with the same document should produce identical passes.
- **Boundary checks** – ensure salience boundaries behave as expected (e.g. no rule below salience X passes when an above-threshold rule should own the decision).
- **Composability** – shared attributes should remain consistent regardless of rule execution order (the engine enforces salience ordering, but invariants still help).

## Optional: observability-driven QA

- Export metrics for pass counts per rule and watch for sudden spikes/drops after deployments.
- Capture samples of `RuleExecutionContext.attributes()` for audit trails.
- Integrate with chaos tooling: mutate documents to ensure the rule engine fails fast when faced with unexpected fields/types.
