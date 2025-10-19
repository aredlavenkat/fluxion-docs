# Rule Engine Examples

This page collects end-to-end scenarios that demonstrate how to describe, validate, and execute rules. Each example builds on the core concepts covered in the other guides.

## Example 1 — Basic pass with DSL and Java

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
boolean passed = !results.getFirst().passes().isEmpty();
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
RuleExecutionContext ctx = results.getFirst().passes().getFirst().context();
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

Because salience values differ, the `escalateFraud` rule executes first. When both pass, two `RulePass` objects are returned, in salience order.

---

## Example 7 — Rule-set hooks for cross-document context

```java
RuleSetHook auditHook = new RuleSetHook() {
    @Override
    public void beforeRules(Document doc, Map<String, Object> shared) {
        shared.put("auditStart", Instant.now());
    }

    @Override
    public void afterRules(Document doc, Map<String, Object> shared, List<RulePass> passes) {
        Duration elapsed = Duration.between((Instant) shared.get("auditStart"), Instant.now());
        metrics.record("rules.elapsed", elapsed.toMillis());
    }
};

RuleSet ruleSet = RuleSet.builder()
    .addHook(auditHook)
    .addRule(rule)
    .build();
```

Hook output (e.g. metrics recording) applies once per document regardless of how many rules passed.

## Example 8 — DSL rule with actions and hooks

You can declare actions and hooks directly in the DSL as long as they are registered at runtime. The `actions` array references action names, while the optional `hooks` arrays reference rule and rule-set hooks by name (resolved from `RuleHookRegistry`).

```json
{
  "id": "loyalty-offers",
  "name": "Loyalty Offers",
  "rules": [
    {
      "id": "gold-upgrade",
      "name": "Upgrade loyal customers",
      "description": "Grant gold tier when spend exceeds threshold",
      "salience": 500,
      "stages": [
        { "$match": { "loyaltyTier": "silver" } },
        { "$match": { "lifetimeSpend": { "$gte": 2500 } } }
      ],
      "actions": ["grant-gold-tier", "notify-upgrade"],
      "hooks": ["audit-loyalty"]
    }
  ],
  "hooks": ["audit-ruleset"]
}
```

Minimal bootstrap code:

```java
@Configuration
class LoyaltyRuleConfig {

    @PostConstruct
    void registerActsAndHooks() {
        RuleActionRegistry.register("grant-gold-tier", ctx -> ctx.putSharedAttribute("upgrade", true));
        RuleActionRegistry.register("notify-upgrade", ctx -> notificationClient.send(ctx.rule().name()));

        RuleHookRegistry.registerRuleHook("audit-loyalty", new LoyaltyAuditHook());
        RuleHookRegistry.registerRuleSetHook("audit-ruleset", new LoyaltyRuleSetHook());
    }
}
```

After registration, the DSL snippet above can reference those names. The parser leaves `actions` and `hooks` untouched; at runtime, the rule builder resolves them and attaches the implementations before evaluation.

## Example 9 — DSL with staged metadata and custom attributes

Metadata travels with the rule and is exposed via `RuleExecutionContext.metadata()` and `RuleDefinition.metadata()`. This is helpful for downstream logging, tracing, or business logic.

```json
{
  "id": "orders",
  "version": "1.4.0",
  "metadata": {
    "owner": "payments-risk",
    "runbook": "https://wiki/internal/runbooks/order-risk"
  },
  "rules": [
    {
      "id": "high-risk-order",
      "name": "High risk order",
      "salience": 800,
      "metadata": {
        "category": "fraud",
        "severity": "high"
      },
      "stages": [
        { "$match": { "status": "pending" } },
        { "$match": { "riskScore": { "$gte": 0.9 } } }
      ],
      "actions": ["flag-order"],
      "hooks": ["audit-order"]
    }
  ]
}
```

When evaluating a document:

```java
RuleEvaluationResult result = engine.execute(List.of(orderDoc), ruleSet).getFirst();
RulePass pass = result.passes().getFirst();
Map<String, Object> metadata = pass.rule().metadata();
log.info("Rule {} fired with severity {}", pass.rule().id(), metadata.get("severity"));
```

This keeps governance data alongside the rule definition and makes it available to actions, hooks, and observability pipelines.
