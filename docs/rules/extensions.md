# Extending the Rule Engine

Fluxion exposes ServiceLoader-based service provider interfaces (SPIs) for adding behaviours without modifying core modules.

## Custom aggregation stages

Register additional aggregation stages by implementing `ai.fluxion.core.engine.spi.StageHandlerContributor` in any module on the classpath.

```java
public final class CustomStageContributor implements StageHandlerContributor {
    @Override
    public Map<String, StageHandler> stageHandlers() {
        return Map.of("$geoFence", new GeoFenceStageHandler());
    }
}
```

Add the fully qualified class name to `META-INF/services/ai.fluxion.core.engine.spi.StageHandlerContributor`. `StageRegistry` automatically loads contributors at startup, making the new stage available to both pipelines and rules.

## Manual action/hook registration

You can register actions and hooks imperatively during application start-up. In a Spring Boot service, run the registration inside a `@PostConstruct` method:

```java
@Configuration
class RuleEngineConfig {

    @PostConstruct
    void registerActionsAndHooks() {
        RuleActionRegistry.register("flag-order", ctx -> ctx.putAttribute("decision", "flagged"));
        RuleHookRegistry.registerRuleHook("audit-before", new AuditRuleHook());
        RuleHookRegistry.registerRuleSetHook("audit-ruleset", new AuditRuleSetHook());
    }
}
```

Outside of Spring, execute the same registration logic when your application boots (e.g., in your dependency injection initializer or main method). Manual registration works well for small deployments or when actions are tightly coupled to a single service.

## Rule actions

Implement `ai.fluxion.rules.spi.RuleActionContributor` to publish reusable actions.

```java
public final class CustomerActions implements RuleActionContributor {
    @Override
    public Map<String, RuleAction> ruleActions() {
        return Map.of(
            "flag-customer", context -> context.putSharedAttribute("customerFlagged", true)
        );
    }
}
```

Declare the contributor in `META-INF/services/ai.fluxion.rules.spi.RuleActionContributor`. `RuleActionRegistry` discovers actions automatically; resolve them by name or call `RuleActionRegistry.reload()` in tests to trigger discovery.

### ServiceLoader checklist

1. Implement the contributor interface (`RuleActionContributor`, `RuleHookContributor`, or `StageHandlerContributor`).
2. Create the matching service descriptor file under `META-INF/services/` with the fully qualified class name (one per line if multiple).
3. Package the implementation and descriptor in the same jar placed on the application classpath.
4. Restart the application (or call the `reload()` helper in tests) to pick up new contributors.
5. Use consistent naming (`team.feature.action-name`) to avoid collisions—when duplicate names exist, the last one registered wins.

ServiceLoader-based contributors shine when you maintain a catalogue of reusable actions/hooks across multiple services.

## Rule and rule-set hooks

Use `ai.fluxion.rules.spi.RuleHookContributor` to add hooks that can be referenced by name when building rules or rule sets.

```java
public final class AuditHooks implements RuleHookContributor {
    @Override
    public Map<String, RuleHook> ruleHooks() {
        return Map.of("audit-before", new AuditRuleHook());
    }

    @Override
    public Map<String, RuleSetHook> ruleSetHooks() {
        return Map.of("audit-ruleset", new AuditRuleSetHook());
    }
}
```

With the contributor registered, builders can call:

```java
RuleDefinition.builder("My Rule")
    .addHookByName("audit-before")
    ...
RuleSet.builder()
    .addHookByName("audit-ruleset")
    ...
```

## Testing SPIs

When writing unit tests for SPI providers, remember to place the service descriptor under `src/test/resources/META-INF/services/` so the test classpath discovers it. The project’s `RuleActionContributorTest`, `RuleHookContributorTest`, and `StageRegistryContributorTest` illustrate the pattern.

## Distribution considerations

- Package contributors in the same jar as their service descriptor. Missing descriptors result in silent no-ops.
- If multiple modules register the same action/hook name, the last one wins. Adopt naming conventions (e.g. prefix with organisation/team).
- Reload behaviour: `RuleActionRegistry.reload()` and `RuleHookRegistry.reload()` rebuild the registry by re-running ServiceLoader. Use these only in tests or controlled admin flows to avoid race conditions.

## Combining SPIs

You can combine custom stages and actions for vertical features. Example:

1. Create a `$httpEnrich` stage via `StageHandlerContributor` that fetches external data.
2. Publish an action `record-http-enrich` that stores enrichment metadata.
3. Build rules that reference both constructs, allowing the engine to fetch data and tag outcomes in one pass.
