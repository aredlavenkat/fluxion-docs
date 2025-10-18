# Rule Engine Examples

This page collects end-to-end scenarios that demonstrate how to describe, validate, and execute rules. Each example builds on the core concepts covered in the other guides.

## Example 1 — Basic match with DSL and Java

### DSL definition

```json
{
  "id": "orders",
  "rules": [
    {
      "id": "active-order",
      "name": "Active order",
      "salience": 10,
      "stages": [
        { "$match": { "status": "active" } }
      ]
    }
  ]
}
```

### Parsing & validation

```java
String json = Files.readString(Path.of("orders-rule.json"));
RuleDslParser parser = new RuleDslParser();
RuleParseResult result = parser.parseWithLints(json);
if (result.hasLints()) {
    throw new IllegalStateException("Rule issues: " + result.lints());
}
RuleSet ruleSet = result.ruleSet();
```

### Runtime evaluation

```java
RuleEngine engine = new RuleEngine();
Document doc = new Document(Map.of("status", "active"));
List<RuleEvaluationResult> results = engine.evaluate(List.of(doc), ruleSet);
boolean matched = !results.getFirst().matches().isEmpty();
```

---

## Example 2 — Actions and shared attributes

This scenario flags suspicious orders and records the decision in the shared context.

```java
RuleActionRegistry.register("flag-order", context -> {
    context.putAttribute("decision", "flagged");
    context.putSharedAttribute("latestDecision", context.rule().name());
});

RuleDefinition suspicious = RuleDefinition.builder("Suspicious order")
    .salience(100)
    .condition(RuleCondition.pipeline(List.of(
        new Stage(Map.of("$match", Map.of("status", "pending"))),
        new Stage(Map.of("$match", Map.of("total", Map.of("$gte", 1000))))
    )))
    .addAction(RuleActionRegistry.resolve("flag-order").orElseThrow())
    .build();

RuleSet ruleSet = RuleSet.builder().addRule(suspicious).build();

List<RuleEvaluationResult> results = new RuleEngine().execute(
    List.of(new Document(Map.of("status", "pending", "total", 1500))),
    ruleSet
);
RuleExecutionContext ctx = results.getFirst().matches().getFirst().context();
assert "flagged".equals(ctx.attributes().get("decision"));
assert "Suspicious order".equals(results.getFirst().sharedAttributes().get("latestDecision"));
```

When this runs in production, both shared and per-rule attributes can be logged or forwarded downstream.

---

## Example 3 — Hooks for auditing

Register hook providers via the SPI (see [Extensions & SPIs](extensions.md)), then reference them by name:

```java
RuleDefinition rule = RuleDefinition.builder("Audited rule")
    .condition(RuleCondition.pipeline(List.of(
        new Stage(Map.of("$match", Map.of("status", "active")))
    )))
    .addHookByName("audit-before")
    .build();

RuleSet ruleSet = RuleSet.builder()
    .addHookByName("audit-ruleset")
    .addRule(rule)
    .build();
```

The contributed hooks can log the evaluation timeline, enrich shared attributes, or emit metrics.

---

## Example 4 — Debugging unexpected drops

Enable debug mode to discover which stage filtered a document out.

```java
Document doc = new Document(Map.of("status", "inactive"));
RuleEvaluationResult result = new RuleEngine()
    .evaluate(List.of(doc), ruleSet, true)
    .getFirst();

RuleExecutionContext context = result.ruleContexts().getFirst();
RuleDebugStageTrace trace = context.debugTrace().getFirst();
System.out.printf("Stage %s filtered document (outputs=%d)\n",
    trace.operator(), trace.outputs().size());
```

Each `RuleDebugStageTrace` contains snapshots of input/out documents, so you can attach them to support tickets or dashboards.

---

## Example 5 — Lint feedback for authoring tools

Provide rich feedback to rule authors before deployment:

```java
RuleParseResult result = new RuleDslParser().parseWithLints(json);
if (result.hasLints()) {
    for (RuleLint lint : result.lints()) {
        ui.notify(lint.type(), lint.message(), lint.context());
    }
}
```

Typical lint payload:

```json
{
  "type": "UNSUPPORTED_OPERATOR",
  "ruleName": "Inactive rule",
  "message": "Rule 'Inactive rule' uses unsupported operator '$foo' at position 1",
  "context": {
    "stageIndex": 0,
    "operator": "$foo"
  }
}
```

Helping authors correct rules early prevents surprises during runtime deployment.

---

## Example 6 — Combining multiple rules with salience bands

```java
RuleDefinition allowLoyalty = RuleDefinition.builder("Allow loyalty member")
    .salience(200)
    .condition(RuleCondition.pipeline(List.of(
        new Stage(Map.of("$match", Map.of("loyalty", true)))
    )))
    .build();

RuleDefinition escalateFraud = RuleDefinition.builder("Escalate fraud")
    .salience(900)
    .condition(RuleCondition.pipeline(List.of(
        new Stage(Map.of("$match", Map.of("flags", Map.of("$in", List.of("fraud")))))
    )))
    .build();

RuleSet ruleSet = RuleSet.builder()
    .addRule(allowLoyalty)
    .addRule(escalateFraud)
    .build();
```

Because salience values differ, the `escalateFraud` rule executes first. When both match, two `RuleMatch` objects are returned, in salience order.

---

## Example 7 — Rule-set hooks for cross-document context

```java
RuleSetHook auditHook = new RuleSetHook() {
    @Override
    public void beforeRules(Document doc, Map<String, Object> shared) {
        shared.put("auditStart", Instant.now());
    }

    @Override
    public void afterRules(Document doc, Map<String, Object> shared, List<RuleMatch> matches) {
        Duration elapsed = Duration.between((Instant) shared.get("auditStart"), Instant.now());
        metrics.record("rules.elapsed", elapsed.toMillis());
    }
};

RuleSet ruleSet = RuleSet.builder()
    .addHook(auditHook)
    .addRule(rule)
    .build();
```

Hook output (e.g. metrics recording) applies once per document regardless of how many rules matched.
