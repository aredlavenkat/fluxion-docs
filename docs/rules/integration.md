# Integrating the Rule Engine

Fluxion's rule engine is lightweight enough to embed directly into microservices, stream processors, or serverless functions. This guide outlines common integration patterns.

## Runtime deployment workflow

1. **Authoring** – rules are created in JSON (or through the builders) and stored in version control or a rule registry.
2. **Validation** – before deployment, run `RuleDslParser.parseWithLints` to surface issues, then persist only lint-free rule sets.
3. **Distribution** – ship rule sets alongside your application binaries, fetch them from an API, or load them from a configuration store at runtime.
4. **Execution** – instantiate `RuleEngine` once per service instance and reuse it. Rule definitions and sets are immutable, so you can safely cache them.
5. **Monitoring** – leverage hook outputs, action side effects, and debug traces to collect metrics and structured logs.

## Embedding in a service

```java
public final class RuleService {
    private final RuleEngine ruleEngine = new RuleEngine();
    private volatile RuleSet activeRuleSet;

    public void loadRules(String json) {
        RuleDslParser parser = new RuleDslParser();
        RuleParseResult result = parser.parseWithLints(json);
        if (result.hasLints()) {
            throw new IllegalArgumentException("Invalid rules: " + result.lints());
        }
        activeRuleSet = result.ruleSet();
    }

    public List<RuleEvaluationResult> evaluate(List<Document> documents, boolean executeActions, boolean debug) {
        RuleSet ruleSet = activeRuleSet;
        if (ruleSet == null) {
            throw new IllegalStateException("No rules loaded");
        }
        if (executeActions) {
            return ruleEngine.execute(documents, ruleSet, debug);
        }
        return ruleEngine.evaluate(documents, ruleSet, debug);
    }
}
```

## Hot reloading

Because `RuleSet` instances are immutable, you can safely swap them atomically (e.g., using `volatile` or `AtomicReference`). Make sure to validate and lint new rule sets before replacing the active one. Keep previous versions available for rollback.

## Scaling considerations

- **Thread safety**: `RuleEngine` is stateless; you can share one instance across threads. Contexts and matches are local to each evaluation call.
- **Stage registry extensions**: if you rely on custom stage handlers, ensure their modules are on the service classpath before the engine initialises.
- **Action side effects**: actions execute synchronously during `execute`. Use asynchronous adapters or queueing if actions perform network or I/O work.
- **Debug mode**: enable debug traces sparingly in production—they snapshot documents and can increase memory usage.

## Service configuration template

```yaml
fluxion:
  rules:
    source: s3://config/rules/orders.json
    refreshInterval: 5m
    debugEnabled: false
    actionRegistry:
      modules:
        - ai.fluxion.rules.actions.BuiltinActions
        - com.example.rules.actions.CustomContributor
```

This example configuration declares where rules are fetched, how often they refresh, and which modules expose actions/hooks. Adapt it to your configuration system (Spring Boot, Micronaut, Quarkus, etc.).

## Integration with connectors & enrichers

The rule engine operates solely on `Document` objects. Use Fluxion Connectors to ingest data (Kafka, HTTP, etc.), Fluxion Enrich to add pre-processing operators, then hand the prepared documents to the rule engine. Many teams schedule rule evaluation as the final stage after enrichment and transformation.

## Observability

- Share structured context data via action attributes or hooks.
- Export `RuleDebugStageTrace` entries to your logging pipeline when troubleshooting.
- Collect counts of matches per rule to monitor drift or unexpected behaviour.

With these patterns in place, the rule engine can power decisioning workloads ranging from fraud detection to entitlement checks with minimal operational friction.
