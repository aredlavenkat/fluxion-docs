# Integration Developer Guide

SrotaX targets engineers who need MongoDB-style analytics and enrichment inside their own services without operating a database engine. This guide shows how to assemble aggregation pipelines, execute them in-process, and extend SrotaX with custom operators or stages.

## Audience Checklist

- You embed SrotaX into a JVM service (Spring Boot, Quarkus, Micronaut, etc.).
- You want the pipeline DSL (Mongo-style stages and expressions) for batch or request/response workloads.
- You may need to add bespoke stages or operators for domain-specific logic.

## Dependencies

Add the modules that match the features you plan to use. At minimum you will need `fluxion-core`. Add `fluxion-enrich` if you rely on `$httpCall`, `$sqlQuery`, or other service-aware operators.

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>ai.fluxion</groupId>
      <artifactId>fluxion-bom</artifactId>
      <version>${fluxion.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<dependencies>
  <dependency>
    <groupId>ai.fluxion</groupId>
    <artifactId>fluxion-core</artifactId>
  </dependency>
  <!-- Optional enrichment operators -->
  <dependency>
    <groupId>ai.fluxion</groupId>
    <artifactId>fluxion-enrich</artifactId>
  </dependency>
</dependencies>
```

## Pipeline Anatomy

SrotaX reuses MongoDB aggregation constructs. A pipeline is composed of:

- **Stage list**: ordered MongoDB-style stages (`$match`, `$project`, `$group`, `$setWindowFields`, etc.).
- **Expressions**: stage payloads that use operators (`$add`, `$map`, `$dateDiff`, etc.).
- **Documents**: input data represented as `ai.fluxion.core.model.Document`.
- **Globals**: optional immutable values available to all expressions (`$$GLOBALS`).

The same stage list can be executed repeatedly on new data, making it easy to store pipeline definitions in configuration or generate them dynamically.

## Example Pipeline JSON

The example below scores incoming telemetry, buckets devices by health, and emits a condensed document.

```json
[
  { "$match": { "status": { "$ne": "offline" } } },
  { "$addFields": {
      "score": {
        "$sum": [
          { "$multiply": ["$metrics.uptimeHours", 0.1] },
          { "$multiply": ["$metrics.signalQuality", 0.4] },
          { "$multiply": ["$metrics.packetSuccess", 0.5] }
        ]
      }
    }
  },
  { "$bucket": {
      "groupBy": "$score",
      "boundaries": [0, 40, 70, 85, 100],
      "default": "investigate",
      "output": {
        "devices": { "$push": "$deviceId" },
        "averageTemp": { "$avg": "$metrics.temperatureC" }
      }
    }
  },
  { "$project": {
      "_id": 0,
      "healthBand": "$_id",
      "devices": 1,
      "averageTemp": 1,
      "processedAt": { "$literal": { "$date": "$$NOW" } }
    }
  }
]
```

Save the JSON to a config store, bundle it with your service, or generate it dynamically.

## Running the Pipeline

You can construct stages programmatically or parse JSON using `DocumentParser`. The snippet below shows batch execution with `PipelineExecutor`.

```java
import ai.fluxion.core.engine.PipelineExecutor;
import ai.fluxion.core.model.Document;
import ai.fluxion.core.model.Stage;
import ai.fluxion.core.util.DocumentParser;

import java.nio.file.Files;
import java.nio.file.Path;
import java.util.List;
import java.util.Map;

List<Document> input = DocumentParser.getDocumentsFromJsonArray(
    Files.readString(Path.of("data/telemetry-sample.json"))
);

List<Stage> stages = DocumentParser.getStagesFromJsonArray(
    Files.readString(Path.of("pipelines/telemetry.json"))
);

PipelineExecutor executor = new PipelineExecutor();
List<Document> results = executor.run(input, stages, Map.of("tenantId", "acme"));

results.forEach(doc -> System.out.println(doc.toJson()));
```

Key points:

- `DocumentParser` converts JSON arrays into SrotaX `Document` and `Stage` instances.
- `PipelineExecutor#run` accepts the input documents, stage list, and optional globals.
- Pipelines are pure functions: they return a new list of documents without mutating the input.

## Managing Pipeline Definitions

- **Storage options**: keep pipeline JSON in source control, a config server (Spring Cloud Config, Consul), or a database table keyed by tenant. The executor is agnostic as long as you can load the JSON string.
- **Cache parsed stages**: converting JSON DSL into `List<Stage>` objects allocates heavily. Cache by pipeline id + version (etag, checksum, `lastModified`) and evict when your configuration changes.

  ```java
  record CachedPipeline(List<Stage> stages, String version, Instant loadedAt) {}

  final class PipelineRepository {
      private final ConcurrentMap<String, CachedPipeline> cache = new ConcurrentHashMap<>();

      CachedPipeline resolve(String name) {
          return cache.compute(name, (key, existing) -> {
              LoadedPipeline loaded = configService.fetch(name);
              if (existing != null && existing.version().equals(loaded.version())) {
                  return existing;
              }
              List<Stage> stages = DocumentParser.getStagesFromJsonArray(loaded.json());
              return new CachedPipeline(stages, loaded.version(), Instant.now());
          });
      }

      void invalidate(String name) {
          cache.remove(name);
      }
  }
  ```

- **Globals**: supply immutable values (tenant, feature flags) through the `globals` map instead of hard-coding them in the pipeline JSON.
- **Error handling**: catch `IllegalArgumentException` and `UnsupportedOperationException` to surface actionable diagnostics back to the client.

## Stage Syntax Cheatsheet

- Every stage is a single-key object: `{ "$stageName": <payload> }`.
- Payload shapes:
  - **Pass-through** (`$match`, `$project`, `$addFields`): Mongo query fragments and expressions.
  - **Accumulators** (`$group`): Use `$sum`, `$avg`, `$max`, etc. inside `output` fields.
  - **Window functions** (`$setWindowFields`): Provide `partitionBy`, `sortBy`, and `output` with window specs.
- Arrays of stages execute in order; stage output feeds into the next stage.
- System variables available everywhere: `$$ROOT`, `$$CURRENT`, `$$NOW`, `$$REMOVE`, `$$PRUNE`, `$$KEEP`.

See the [Stages reference](../stages/index.md) for stage-by-stage payload details.

## Expression Operators Cheatsheet

- Operators mirror MongoDB semantics; they are case-sensitive and always start with `$`.
- Operands accept literals, field paths (prefixed with `$`), sub-expressions, or nested documents.
- Common families:
  - **Comparison**: `$eq`, `$gt`, `$gte`, `$lt`, `$lte`, `$ne`, `$in`, `$nin`.
  - **Math**: `$add`, `$subtract`, `$multiply`, `$divide`, `$sqrt`, `$pow`.
  - **Array**: `$map`, `$filter`, `$reduce`, `$size`, `$concatArrays`, `$zip`.
  - **Date**: `$dateDiff`, `$dateAdd`, `$dateTrunc`, `$isoWeek`, `$week`.
  - **Control flow**: `$cond`, `$switch`, `$ifNull`, `$coalesce`, `$let`.
- Operators can be nested arbitrarily: `{"$map": {"input": "$events", "as": "event", "in": {"$toUpper": "$$event.type"}}}`.

See the [Operators reference](../operators/index.md) for exhaustive syntax, edge cases, and performance notes.

## Adding a Custom Operator

1. Implement `Operator` and resolve operands with `ExpressionEvaluator`.
2. Register the operator through an `OperatorContributor`.
3. Advertise the contributor using Java `ServiceLoader`.

```java
package com.acme.telemetry.ops;

import ai.fluxion.core.expression.ExpressionEvaluator;
import ai.fluxion.core.expression.Operator;
import ai.fluxion.core.expression.OperatorContributor;
import ai.fluxion.core.expression.OperatorRegistry;
import ai.fluxion.core.model.Document;

import java.util.List;
import java.util.Map;

public final class WeightedPercentileOperator implements Operator {
    @Override
    public Object apply(Object expr, Document doc, Map<String, Object> vars, ExpressionEvaluator evaluator) {
        Map<String, Object> spec = (Map<String, Object>) expr;
        List<Map<String, Object>> buckets = (List<Map<String, Object>>) evaluator.resolve(spec.get("buckets"), doc, vars);
        double percentile = ((Number) spec.getOrDefault("percentile", 0.5d)).doubleValue();
        return PercentileMath.weighted(buckets, percentile);
    }

    public static final class Contributor implements OperatorContributor {
        @Override
        public void contribute(OperatorRegistry registry) {
            registry.register("$weightedPercentile", new WeightedPercentileOperator());
        }
    }
}
```

`src/main/resources/META-INF/services/ai.fluxion.core.expression.OperatorContributor`:

```
com.acme.telemetry.ops.WeightedPercentileOperator$Contributor
```

- Register imperatively in tests: `OperatorRegistry.getInstance().register("$weightedPercentile", new WeightedPercentileOperator());`.
- Operators run on every document; ensure they are pure functions and validate inputs defensively.

## Adding a Custom Stage

1. Implement `StageHandler` and encapsulate your transformation inside `apply`.
2. Return the input list (possibly mutated) or allocate a new list of `Document` objects.
3. Register the handler through `StageHandlerContributor` and `ServiceLoader`.

```java
package com.acme.telemetry.stages;

import ai.fluxion.core.aggregation.stages.StageHandler;
import ai.fluxion.core.engine.spi.StageHandlerContributor;
import ai.fluxion.core.model.Document;

import java.util.List;
import java.util.Map;

public final class MaskFieldsStageHandler implements StageHandler {
    @Override
    public List<Document> apply(List<Document> input, Object spec, Map<String, Object> vars) {
        Map<String, Object> config = (Map<String, Object>) spec;
        List<String> fields = (List<String>) config.get("fields");
        Object replacement = config.getOrDefault("replacement", "***");

        for (Document document : input) {
            for (String field : fields) {
                if (document.containsKey(field)) {
                    document.put(field, replacement);
                }
            }
        }
        return input;
    }

    public static final class Contributor implements StageHandlerContributor {
        @Override
        public Map<String, StageHandler> stageHandlers() {
            return Map.of("$maskFields", new MaskFieldsStageHandler());
        }
    }
}
```

`src/main/resources/META-INF/services/ai.fluxion.core.engine.spi.StageHandlerContributor`:

```
com.acme.telemetry.stages.MaskFieldsStageHandler$Contributor
```

Tips:

- Stages receive `vars` populated with `$$ROOT`, `$$CURRENT`, and any globals supplied when executing the pipeline.
- Use `StageRegistry.getInstance().register(...)` during development or tests to inject stages without ServiceLoader wiring.
- `StageRegistry` and `OperatorRegistry` are thread-safe singletons; register contributions during application bootstrap to avoid races.

## Additional Resources

- [Stage reference](../stages/index.md) for payload schemas, aliases, and gotchas.
- [Operator reference](../operators/index.md) for complete expression coverage.
- [Enrichment operators](../enrich/index.md) for `$httpCall`, `$sqlQuery`, and service integrations.
- SrotaX Core repository (`fluxion-core-engine-java/docs/fluxion-core-developer-guide.md`) for engine internals and SPI details.
- [LLM assistant notes](../shared/llm-assistant-notes.md) to keep generated answers aligned with current capabilities.
