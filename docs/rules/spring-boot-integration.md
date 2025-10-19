# Spring Boot Starter Integration

This guide shows how to embed the Fluxion rule engine inside a Spring Boot application using the `fluxion-rules-spring-boot-starter`. It covers dependency wiring, rule-set discovery, REST evaluation, and operational telemetry through Micrometer and Actuator.

> **Requirements**: Spring Boot 3.2+, Java 21, Fluxion `0.0.1-SNAPSHOT` (or later).

## 1. Add the starter dependency

=== "Maven"
    ```xml
    <dependency>
        <groupId>ai.fluxion</groupId>
        <artifactId>fluxion-rules-spring-boot-starter</artifactId>
        <version>${fluxion.version}</version>
    </dependency>
    ```

=== "Gradle"
    ```kotlin
    implementation("ai.fluxion:fluxion-rules-spring-boot-starter:$fluxionVersion")
    ```

The starter pulls in Spring Boot autoconfiguration, the Fluxion rule engine, and optional Actuator/Micrometer integrations.

## 2. Provide rule-set JSON

Create JSON rule sets using the standard DSL. Place them on the classpath or any location reachable via Spring `Resource` patterns.

```
src
 └── main
     └── resources
         └── rules
             ├── orders.json
             ├── customers.json
             └── returns.json
```

Example `orders.json`:

```json
{
  "id": "orders",
  "name": "Order Rules",
  "version": "1.0.0",
  "rules": [
    {
      "id": "high-value",
      "name": "High value order",
      "salience": 10,
      "stages": [
        { "$match": { "total": { "$gte": 1000 } } }
      ],
      "actions": ["flag-order"]
    }
  ]
}
```

## 3. Configure discovery and exposure

`application.yml` example:

```yaml
spring:
  application:
    name: fluxion-rules-service

fluxion:
  rules:
    locations:
      - classpath:rules/*.json
    fail-on-error: true
    require-explicit-id: true
    registry-enabled: true
    evaluation-service-enabled: true

management:
  endpoints:
    web:
      exposure:
        include: health,info,fluxionRules
```

Key points:

- `locations` accepts any Spring `Resource` pattern (`classpath*:rules/*.json`, `file:/etc/fluxion/*.json`, etc.).
- `fail-on-error=false` lets the app boot even when some JSON is invalid; errors are reported via Actuator.
- `registry-enabled=false` skips the in-memory `RuleSetRegistry` if you provide your own.
- `evaluation-service-enabled=false` disables the convenience `RuleEvaluationService` bean.

Refer to the [starter reference](../rules-spring-boot-starter.md) for the full property matrix, Micrometer gauges, and Actuator endpoint payloads.

## 4. React to rule discovery

Use `RuleSetsLoadedEvent` to register custom actions/hooks or kick off downstream processes once rule sets are loaded.

```java
@Component
class RuleBootstrapListener {

    @EventListener
    void onRuleSetsLoaded(RuleSetsLoadedEvent event) {
        RuleActionRegistry.register("flag-order", ctx -> ctx.putAttribute("flagged", true));

        event.getLoadResult().registry().descriptors()
                .forEach(descriptor -> log.info("Loaded rule set {} from {}", descriptor.id(), descriptor.source()));
    }
}
```

Hook registration can also rely on ServiceLoader SPIs; the event is convenient for dynamic wiring.

## 5. Evaluate requests via REST

Inject `RuleEvaluationService` to evaluate inbound JSON payloads:

```java
@RestController
@RequestMapping("/rules")
class RuleEvaluationController {

    private final RuleEvaluationService evaluationService;

    RuleEvaluationController(RuleEvaluationService evaluationService) {
        this.evaluationService = evaluationService;
    }

    @PostMapping("{ruleSetId}/evaluate")
    RuleEvaluationResult evaluate(@PathVariable String ruleSetId,
                                  @RequestBody Map<String, Object> payload) {
        return evaluationService.evaluate(new Document(payload), ruleSetId, true);
    }
}
```

When `RuleEvaluationService` is disabled, inject `RuleEngine` directly:

```java
RuleEvaluationResult evaluateManual(String ruleSetId, Map<String, Object> payload) {
    RuleSet ruleSet = registry.getRequired(ruleSetId);
    return ruleEngine.evaluate(new Document(payload), ruleSet);
}
```

## 6. Observe health and metrics

With `spring-boot-starter-actuator` on the classpath:

- `/actuator/health/fluxionRules` reports rule count, last load time, and non-fatal load errors.
- `/actuator/fluxionRules` lists rule-set metadata (id, name, version, source) plus error details.
- Micrometer gauges are auto-bound: `fluxion.rules.rule_sets`, `fluxion.rules.load_errors`, `fluxion.rules.last_loaded_epoch_millis`.

Expose the endpoints via `management.endpoints.web.exposure.include`. Feed metrics into Prometheus, Datadog, etc.

## 7. Testing patterns

Use `ApplicationContextRunner` to verify autoconfiguration:

```java
new ApplicationContextRunner()
        .withConfiguration(AutoConfigurations.of(FluxionRulesAutoConfiguration.class))
        .withPropertyValues("fluxion.rules.locations=classpath:rules/*.json")
        .run(context -> {
            assertThat(context).hasSingleBean(RuleSetRegistry.class);
            assertThat(context.getBean(RuleSetRegistry.class).find("orders")).isPresent();
        });
```

For behavioural tests, call `RuleEvaluationService` with sample documents and assert on `RulePass` outputs.

## 8. Sample application

A runnable Spring Boot sample demonstrating YAML config, REST endpoints, and Actuator exposure lives in the `samples/fluxion-rules-starter-sample/` directory of the core repository. A dedicated samples repository will be linked here once published.

## 9. Next steps

- Extend rules with custom actions/hooks via the SPIs (see [Extensions & SPIs](extensions.md)).
- Automate rule validation in your CI pipeline (linting, evaluation tests, JSON schema checks).
- Combine this starter with Fluxion connectors to build full ingestion + decision services.
