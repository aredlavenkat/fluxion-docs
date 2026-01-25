# Spring Boot Starter Integration

This guide shows how to embed the SrotaX rule engine inside a Spring Boot application using the `fluxion-rules-spring-boot-starter`. It covers dependency wiring, rule-set discovery, REST evaluation, and operational telemetry through Micrometer and Actuator.

> **Requirements**: Spring Boot 3.2+, Java 21, SrotaX `0.0.1-SNAPSHOT` (or later).

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

The starter pulls in Spring Boot autoconfiguration, the SrotaX rule engine, and optional Actuator/Micrometer integrations.

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

SrotaX:
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
        include: health,info,SrotaXRules
```

Key points:

- `locations` accepts any Spring `Resource` pattern (`classpath*:rules/*.json`, `file:/etc/SrotaX/*.json`, etc.).
- `fail-on-error=false` lets the app boot even when some JSON is invalid; errors are reported via Actuator.
- `registry-enabled=false` skips the in-memory `RuleSetRegistry` if you provide your own.
- `evaluation-service-enabled=false` disables the convenience `RuleEvaluationService` bean.

### Property reference

All starter settings share the `fluxion.rules.` prefix:

| Property | Description | Default |
| --- | --- | --- |
| `directory` | Legacy filesystem directory scanned when `locations` is empty. Relative paths resolve against the application working directory. | `rules` |
| `locations` | List of Spring `Resource` patterns (e.g. `classpath*:rules/*.json`, `file:/etc/SrotaX/*.json`). When empty the starter falls back to `directory`. | `[]` |
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

- `/actuator/health/SrotaXRules` includes `ruleSetCount`, `lastLoaded`, and a list of non-fatal discovery errors.
- `/actuator/SrotaXRules` returns rule-set descriptors (`id`, `name`, `version`, `ruleCount`, `metadata`, `source`) and the same error payload.
- Micrometer gauges are auto-bound: 
  - `fluxion.rules.rule_sets` – number of loaded rule sets
  - `fluxion.rules.load_errors` – non-fatal error count (only increments when `fail-on-error=false`)
  - `fluxion.rules.last_loaded_epoch_millis` – epoch timestamp of the last successful load

Example `/actuator/SrotaXRules` response:

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
        .withConfiguration(AutoConfigurations.of(SrotaXRulesAutoConfiguration.class))
        .withPropertyValues("fluxion.rules.locations=classpath:rules/*.json")
        .run(context -> {
            assertThat(context).hasSingleBean(RuleSetRegistry.class);
            assertThat(context.getBean(RuleSetRegistry.class).find("orders")).isPresent();
        });
```

For behavioural tests, call `RuleEvaluationService` with sample documents and assert on `RulePass` outputs.

## 8. Sample application

A runnable Spring Boot sample demonstrating YAML config, REST endpoints, and Actuator exposure lives in the `samples/fluxion-rules-starter-sample/` directory of the core repository. A dedicated samples repository will be linked here once published.

## 9. Extend rules with custom actions/hooks via the SPIs (see Extensions & SPIs)

The starter honours the rule-engine Service Provider Interfaces (SPIs), so any custom actions or hooks you publish are discovered automatically at startup. This keeps behaviour reusable and infrastructure code minimal.

### When to use the SPIs

- Share a library of actions/hooks across multiple services.
- Encapsulate side-effects (notifications, persistence, metrics) in actions rather than controllers.
- Attach instrumentation or safety guards (timeouts, auditing) through hooks.

### Implementation steps

1. **Create a contributor class.**
   - Actions: implement `ai.fluxion.rules.spi.RuleActionContributor` and return a `Map<String, RuleAction>`.
   - Hooks: implement `ai.fluxion.rules.spi.RuleHookContributor` and return maps for `RuleHook` and/or `RuleSetHook`.
   - Both delegate to the shared `ai.fluxion.core.pipeline` infrastructure, so any action/hook you publish can be reused by other SrotaX engines (e.g., streaming) without additional wiring.
2. **Register with ServiceLoader.** Add a descriptor file under `META-INF/services/` containing the fully-qualified class name.
3. **Package on the classpath.** Include the contributor jar in your application; the starter’s registries load it via `ServiceLoader`.
4. **Reference by name in DSL/Java.** Use the action/hook names in rule JSON or builder APIs (`builder.addHookByName("audit-before")`).

### Example: Action contributor

```java
package com.example.rules.actions;

import ai.fluxion.rules.domain.RuleAction;
import ai.fluxion.rules.spi.RuleActionContributor;

import java.util.Map;

public final class NotificationActionContributor implements RuleActionContributor {

    @Override
    public Map<String, RuleAction> ruleActions() {
        return Map.of(
                "notify-upgrade", context -> notificationClient().sendUpgrade(context.document()),
                "flag-order", context -> context.putAttribute("flagged", true)
        );
    }

    private NotificationClient notificationClient() {
        return NotificationClientHolder.INSTANCE;
    }
}
```

`META-INF/services/ai.fluxion.rules.spi.RuleActionContributor`:

```
com.example.rules.actions.NotificationActionContributor
```

### Example: Hook contributor

```java
package com.example.rules.hooks;

import ai.fluxion.rules.engine.RuleHook;
import ai.fluxion.rules.engine.RuleSetHook;
import ai.fluxion.rules.spi.RuleHookContributor;

import java.util.Map;

public final class AuditingHookContributor implements RuleHookContributor {

    @Override
    public Map<String, RuleHook> ruleHooks() {
        return Map.of("audit-before", context -> auditService().logStart(context));
    }

    @Override
    public Map<String, RuleSetHook> ruleSetHooks() {
        return Map.of("capture-metrics", new MetricsCapturingHook());
    }
}
```

`META-INF/services/ai.fluxion.rules.spi.RuleHookContributor`:

```
com.example.rules.hooks.AuditingHookContributor
```

### Testing contributors

```java
@Test
void actionContributorRegistersNames() {
    NotificationActionContributor contributor = new NotificationActionContributor();
    assertThat(contributor.ruleActions()).containsKey("notify-upgrade");
}
```

You can also bootstrap a Spring context (with the starter on the classpath) and assert that `RuleActionRegistry.resolve("notify-upgrade")` returns the expected implementation.

### Dynamic overrides via events

For experiments or environment-specific tweaks, register actions/hooks imperatively inside a `RuleSetsLoadedEvent` listener:

```java
@EventListener
void onRulesLoaded(RuleSetsLoadedEvent event) {
    RuleActionRegistry.register("debug-action", ctx -> log.debug("rule {} fired", ctx.rule().name()));
}
```

These manual registrations live alongside ServiceLoader contributions and are handy for toggling behaviour based on feature flags or configuration services.

> Tip: keep action/hook names lowercase-with-dashes to avoid collisions and keep DSL payloads readable.
