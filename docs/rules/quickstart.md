# Quick Start

This walkthrough shows how to go from an empty project to a running rule engine evaluation. It covers four steps:

1. **Declare dependencies**
2. **Write a rule set (DSL + Java)**
3. **Validate and load the rule**
4. **Evaluate documents and inspect passes**

## 1. Dependencies

Add the SrotaX artifacts to your build. With Maven:

```xml
<dependency>
  <groupId>ai.fluxion</groupId>
  <artifactId>fluxion-core</artifactId>
  <version>${fluxion.version}</version>
</dependency>
<dependency>
  <groupId>ai.fluxion</groupId>
  <artifactId>fluxion-rules</artifactId>
  <version>${fluxion.version}</version>
</dependency>
```

Gradle (Kotlin DSL):

```kotlin
dependencies {
    implementation("ai.fluxion:fluxion-core:$fluxionVersion")
    implementation("ai.fluxion:fluxion-rules:$fluxionVersion")
}
```

Ensure any custom actions/stages you need are on the classpath before the engine initialises.

## 2. Write a rule set

Create a JSON file (`rules.json`) describing the conditions and actions you want:

```json
{
  "id": "orders",
  "name": "Order Decisions",
  "rules": [
    {
      "id": "high-value",
      "name": "High value order",
      "salience": 100,
      "stages": [
        { "$match": { "status": "active" } },
        { "$match": { "total": { "$gte": 500 } } }
      ],
      "actions": ["flag-order"]
    }
  ]
}
```

Register an action somewhere during startup:

```java
RuleActionRegistry.register("flag-order", context -> {
    context.putAttribute("decision", "flagged");
});
```

## 3. Validate and load the rule

```java
RuleDslParser parser = new RuleDslParser();
RuleParseResult result = parser.parseWithLints(Files.readString(Path.of("rules.json")));
if (result.hasLints()) {
    result.lints().forEach(lint ->
        System.err.printf("[%s] %s%n", lint.type(), lint.message())
    );
    throw new IllegalStateException("Fix lints before continuing");
}
RuleSet ruleSet = result.ruleSet();
```

Alternatively, build directly in Java:

```java
RuleDefinition rule = RuleDefinition.builder("High value")
    .salience(100)
    .condition(RuleCondition.pipeline(List.of(
        new Stage(Map.of("$match", Map.of("status", "active"))),
        new Stage(Map.of("$match", Map.of("total", Map.of("$gte", 500))))
    )))
    .addAction(RuleActionRegistry.resolve("flag-order").orElseThrow())
    .build();

RuleSet ruleSet = RuleSet.builder().addRule(rule).build();
```

## 4. Evaluate documents

```java
RuleEngine engine = new RuleEngine();
List<Document> documents = List.of(
    new Document(Map.of("status", "active", "total", 750)),
    new Document(Map.of("status", "inactive", "total", 1200))
);

List<RuleEvaluationResult> results = engine.execute(documents, ruleSet, true);

for (RuleEvaluationResult result : results) {
    System.out.printf("Document %s -> passes=%d shared=%s%n",
        result.document().toMap(),
        result.passes().size(),
        result.sharedAttributes()
    );

    for (RulePass rulePass : result.passes()) {
        System.out.printf("  Rule %s actions=%s debug=%s%n",
            rulePass.rule().name(),
            rulePass.context().attributes(),
            rulePass.context().debugTrace()
        );
    }
}
```

`RulePass` exposes the rule, context, and queued actions for any rule whose condition pipeline passed. Debug mode (`true`) collects `RuleDebugStageTrace` entries so you can inspect how each stage impacted the document.

## Next steps

- Learn the [DSL schema](dsl-reference.md) to build validation tooling.
- Understand [runtime execution](runtime.md) and how hooks share context.
- Explore [testing strategies](testing.md) and the [extension SPIs](extensions.md).
