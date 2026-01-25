# Integration Guide

This recipe shows how to embed the rule engine directly inside a Java service. The example uses Spring Boot, but the same concepts apply to any DI framework.

## 1. Wire the engine and rule set

Create a configuration class that builds a singleton `RuleEngine` and a `RuleSet` parsed from JSON. The JSON can live on the classpath, a config service, or any storage of your choice.

```java
@Configuration
class RuleEngineConfig {

    private final ResourceLoader resourceLoader;

    RuleEngineConfig(ResourceLoader resourceLoader) {
        this.resourceLoader = resourceLoader;
    }

    @Bean
    RuleEngine ruleEngine() {
        return new RuleEngine();
    }

    @Bean
    RuleDslParser ruleDslParser() {
        return new RuleDslParser();
    }

    @Bean
    RuleSet ruleSet(RuleDslParser parser) throws IOException {
        Resource resource = resourceLoader.getResource("classpath:rules/orders.json");
        String json = resource.getContentAsString(StandardCharsets.UTF_8);

        RuleParseResult result = parser.parseWithLints(json);
        if (result.hasLints()) {
            throw new IllegalStateException("Rule errors: " + result.lints());
        }
        return result.ruleSet();
    }

    @PostConstruct
    void registerActionsAndHooks() {
        RuleActionRegistry.register("flag-order", ctx -> ctx.putAttribute("decision", "flagged"));
        // Register hooks similarly or rely on ServiceLoader contributors.
    }
}
```

**Thread-safety guidance**

- Share a single `RuleEngine` bean; it is stateless and safe across threads.
- Treat `RuleSet` as immutable; reload it atomically if you need hot updates (e.g., using `AtomicReference<RuleSet>`).
- Reuse `RuleDslParser` if you frequently reload rule sets.

## 2. Inject the engine where you need it

```java
@RestController
@RequestMapping("/rules")
class RuleController {

    private final RuleEngine ruleEngine;
    private final Supplier<RuleSet> ruleSetSupplier;

    RuleController(RuleEngine ruleEngine, RuleSet ruleSet) {
        this.ruleEngine = ruleEngine;
        this.ruleSetSupplier = () -> ruleSet; // Replace with AtomicReference for hot reloads.
    }

    @PostMapping("/evaluate")
    List<RuleEvaluationResult> evaluate(@RequestBody Map<String, Object> payload,
                                        @RequestParam(defaultValue = "false") boolean execute,
                                        @RequestParam(defaultValue = "false") boolean debug) {
        Document document = new Document(payload);
        RuleSet ruleSet = ruleSetSupplier.get();
        if (execute) {
            return ruleEngine.execute(List.of(document), ruleSet, debug);
        }
        return ruleEngine.evaluate(List.of(document), ruleSet, debug);
    }
}
```

You can adapt the same pattern for message listeners, schedulers, or event handlers; just inject the `RuleEngine` and `RuleSet` wherever you need rule evaluation.

## 3. Reloading rules

To support dynamic updates, wrap the `RuleSet` in an `AtomicReference` and schedule a task that pulls new JSON, validates it, and swaps the reference:

```java
@Component
class RuleReloader {

    private final AtomicReference<RuleSet> ruleSetRef = new AtomicReference<>();
    private final RuleDslParser parser;
    private final RuleEngine ruleEngine;

    RuleReloader(RuleDslParser parser, RuleEngine ruleEngine) {
        this.parser = parser;
        this.ruleEngine = ruleEngine;
    }

    @Scheduled(fixedDelayString = "PT5M")
    void refresh() throws IOException {
        String json = fetchFromS3("s3://config/rules/orders.json");
        RuleParseResult result = parser.parseWithLints(json);
        if (!result.hasLints()) {
            ruleSetRef.set(result.ruleSet());
        } else {
            log.warn("Skipping rule update due to lints: {}", result.lints());
        }
    }

    RuleSet currentRuleSet() {
        return ruleSetRef.get();
    }
}
```

Pass `ruleSetRef::get` into your controllers/services so they always read the most recent version.

## 4. Observability & debugging

- Enable debug mode (`evaluate(..., true)`) when diagnosing issues; inspect `context.debugTrace()` for per-stage data.
- Export metrics via the built-in OpenTelemetry bridge (configure `OTEL_*` environment variables).
- Log `RuleEvaluationResult.passes()` and shared attributes to trace decisions.

## 5. Actions and hooks

- Register actions manually (as shown above) or via ServiceLoader contributors.
- Use `RuleHookRegistry.addHookByName` / `RuleSet.Builder.addHookByName` when consuming contributors by name.
- Keep actions idempotent; they may run again if you call `execute` multiple times.

With these patterns, you can embed SrotaX’s rule engine directly into your service while retaining full control over configuration, deployment, and lifecycle.
