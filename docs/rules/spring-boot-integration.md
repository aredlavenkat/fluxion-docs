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

### Property reference

All starter settings share the `fluxion.rules.` prefix:

| Property | Description | Default |
| --- | --- | --- |
| `directory` | Legacy filesystem directory scanned when `locations` is empty. Relative paths resolve against the application working directory. | `rules` |
| `locations` | List of Spring `Resource` patterns (e.g. `classpath*:rules/*.json`, `file:/etc/fluxion/*.json`). When empty the starter falls back to `directory`. | `[]` |
| `fail-on-error` | Fail startup if discovery fails or JSON cannot be parsed. When `false`, errors are logged and surfaced via Actuator. | `true` |
| `require-explicit-id` | Enforce rule-set `id` values in JSON. When `false`, ids fall back to the file name. | `true` |
| `registry-enabled` | Publish the auto-configured `RuleSetRegistry` bean. Disable when you provide your own registry implementation. | `true` |
| `evaluation-service-enabled` | Publish the convenience `RuleEvaluationService`; requires the registry bean. | `true` |

Refer to the starter configuration table whenever you need to tweak behaviour per environment.

## 4. React to rule discovery

Use `RuleSetsLoadedEvent` to register custom actions/hooks or kick off downstream processes once rule sets are loaded.

```java
@Component
class RuleBootstrapListener {

    @EventListener
    void onRuleSetsLoaded(RuleSetsLoadedEvent event) {
        // Register custom rule actions on first load
        RuleActionRegistry.register("audit-event", ctx ->
                log.info("Audit rule {}", ctx.rule().name()));

        // Wire rule-set hooks dynamically if desired
        RuleHookRegistry.registerRuleSetHook("capture-metrics", new MetricsCapturingHook());

        // Inspect loaded descriptors for logging or auditing
        event.getLoadResult().registry().descriptors()
                .forEach(descriptor -> log.info("Loaded rule set {} from {}", descriptor.id(), descriptor.source()));
    }
}
```

Hook/action registration can also rely on ServiceLoader SPIs; the event is an optional integration point for dynamic wiring.

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

## 6. Observe health, metadata, and metrics

With `spring-boot-starter-actuator` on the classpath:

- `/actuator/health/fluxionRules` includes `ruleSetCount`, `lastLoaded`, and a list of non-fatal discovery errors.
- `/actuator/fluxionRules` returns rule-set descriptors (`id`, `name`, `version`, `ruleCount`, `metadata`, `source`) and the same error payload.
- Micrometer gauges are auto-bound: 
  - `fluxion.rules.rule_sets` – number of loaded rule sets
  - `fluxion.rules.load_errors` – non-fatal error count (only increments when `fail-on-error=false`)
  - `fluxion.rules.last_loaded_epoch_millis` – epoch timestamp of the last successful load

Example `/actuator/fluxionRules` response:

```json
{
  "ruleSets": [
    {
      "id": "orders",
      "name": "Order Rules",
      "version": "1.0.0",
      "ruleCount": 4,
      "metadata": {
        "owner": "payments-risk"
      },
      "source": "classpath:rules/orders.json"
    }
  ],
  "errors": [],
  "loadedAt": "2024-10-10T18:23:11.123Z"
}
```

Errors appear only when `fail-on-error=false` and the starter skips malformed files.

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
