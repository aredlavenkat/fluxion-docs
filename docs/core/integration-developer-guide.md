# Integration Developer Guide

Fluxion targets engineers who need MongoDB-style analytics, enrichment, and streaming orchestration inside their own services. This guide shows how to assemble pipelines, wire the runtime, and extend Fluxion when a built-in operator or stage is missing.

## Audience Checklist

- You embed Fluxion into a JVM service (Spring Boot, Quarkus, Micronaut, etc.).
- You want the pipeline DSL (Mongo-style stages and expressions) without running MongoDB.
- You may need to add bespoke stages or operators for domain-specific logic.

## Dependencies

Add the modules that match the features you plan to use. At minimum you will need `fluxion-core`. Add `fluxion-connect` for connectors (Kafka source, sinks) and `fluxion-enrich` if you rely on `$httpCall`, `$sqlQuery`, or other service-aware operators.

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
  <dependency>
    <groupId>ai.fluxion</groupId>
    <artifactId>fluxion-connect</artifactId>
  </dependency>
  <dependency>
    <groupId>ai.fluxion</groupId>
    <artifactId>fluxion-enrich</artifactId>
  </dependency>
</dependencies>
```

> If you do not use enrichment operators or custom connectors you can omit the corresponding modules.

## Pipeline Anatomy

Fluxion reuses MongoDB aggregation constructs. A streaming pipeline is composed of:

- **Source connector**: Supplies documents (Kafka topic, HTTP poller, custom source).
- **Stage list**: Ordered MongoDB-style stages (`$match`, `$project`, `$group`, `$setWindowFields`, etc.).
- **Expressions**: Stage payloads that use operators (`$add`, `$map`, `$dateDiff`).
- **Sink**: Optional connector or custom sink `StreamingSink`; defaults to an in-memory collector.
- **Runtime configuration**: Batch size, queue capacities, worker pool sizing, error policies.

The same stage list can be executed in batch (`PipelineExecutor`) or streaming (`StreamingPipelineExecutor` via the orchestrator).

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

Construct stages programmatically or parse JSON using `DocumentParser`.

```java
import ai.fluxion.core.engine.streaming.orchestrator.StreamingPipelineDefinition;
import ai.fluxion.core.engine.streaming.orchestrator.StreamingPipelineOrchestrator;
import ai.fluxion.core.engine.streaming.orchestrator.StreamingPipelineHandle;
import ai.fluxion.core.engine.streaming.StreamingRuntimeConfig;
import ai.fluxion.core.engine.streaming.StreamingErrorPolicy;
import ai.fluxion.core.engine.connectors.SourceConnectorConfig;
import ai.fluxion.core.model.Stage;
import ai.fluxion.core.util.DocumentParser;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.List;

SourceConnectorConfig source = SourceConnectorConfig.builder("kafka")
    .topic("telemetry.events")
    .option("bootstrapServers", "localhost:9092")
    .option("groupId", "telemetry-pipeline")
    .build();

List<Stage> stages = DocumentParser.getStagesFromJsonArray(
    Files.readString(Path.of("pipelines/telemetry.json"))
);

StreamingRuntimeConfig runtime = StreamingRuntimeConfig.builder()
    .stageWorkerThreads(4)
    .sourceQueueCapacity(2048)
    .build();

StreamingPipelineDefinition definition = StreamingPipelineDefinition.builder(source)
    .stages(stages)
    .runtimeConfig(runtime)
    .errorPolicy(StreamingErrorPolicy.retrying(3))
    .pipelineId("telemetry-health-score")
    .build();

StreamingPipelineOrchestrator orchestrator = new StreamingPipelineOrchestrator();
StreamingPipelineHandle handle = orchestrator.run(definition);
```

Key points:

- `DocumentParser.getStagesFromJsonArray` loads JSON portable pipelines.
- `StreamingPipelineDefinition` owns the source, stage list, runtime config, optional sink, and failure policy.
- `StreamingPipelineOrchestrator` delegates to `StreamingPipelineExecutor` and returns a `StreamingPipelineHandle` for lifecycle hooks (pause, resume, metrics).

To run a batch pipeline (no streaming source), call `PipelineExecutor.run(List<Document> input, List<Stage> stages, Map<String, Object> globals)`.

## Stage Syntax Cheatsheet

- Every stage is a single-key object: `{ "$stageName": <payload> }`.
- Payload shapes:
  - **Pass-through** (`$match`, `$project`, `$addFields`): Mongo query fragments and expressions.
  - **Accumulators** (`$group`): Use `$sum`, `$avg`, `$max`, etc. inside `output` fields.
  - **Window functions** (`$setWindowFields`): Provide `partitionBy`, `sortBy`, and `output` with window specs.
- Arrays of stages execute in order; stage output feeds into the next stage.
- System variables available everywhere: `$$ROOT`, `$$CURRENT`, `$$NOW`, `$$REMOVE`, `$$PRUNE`, `$$KEEP`.

See the [Stages reference](../stages/) for stage-by-stage payload details.

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

See the [Operators reference](../operators/) for exhaustive syntax, edge cases, and performance notes.

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
2. Optionally override `process` to access `StageExecutionContext` (state store, metrics).
3. Return the input list (possibly mutated) or allocate a new list of `Document` objects.
4. Register the handler through `StageHandlerContributor` and `ServiceLoader`.

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

- Stages receive `vars` populated with `$$ROOT`, `$$CURRENT`, and any globals supplied via `StreamingPipelineDefinition.globalVariables(...)`.
- Use `StageExecutionContext` (override `process`) when you need metric recording or state storage.
- For one-off experiments you can register stages at runtime: `StageRegistry.getInstance().register("$maskFields", new MaskFieldsStageHandler());`.

## Operational Hooks

- **Metrics**: `StageMetrics.MetricSnapshot` captures per-stage throughput, latency, queue depth. Bridge to OpenTelemetry via `StageMetricsOtelBridge`.
- **Error handling**: `StreamingErrorPolicy.retrying(retries)` or `StreamingErrorPolicy.deadLetter(...)` controls how the runtime reacts to handler exceptions.
- **Global variables**: Supply immutable values (feature flags, static lookups) via `StreamingPipelineDefinition.globalVariables(Map.of("tenantId", "acme"))`. They surface as `$$GLOBALS.tenantId`.
- **State store**: Swap `StateStore` (default `InMemoryStateStore`) for a persistent implementation when stages must checkpoint progress.

## Additional Resources

- [Stage reference](../stages/) for payload schemas, aliases, and gotchas.
- [Operator reference](../operators/) for complete expression coverage.
- [Connector guides](../connect/) for Kafka and future connectors.
- [Enrichment operators](../enrich/) for `$httpCall`, `$sqlQuery`, and service integrations.
- Fluxion Core repository (`fluxion-core-engine-java/docs/fluxion-core-developer-guide.md`) for engine internals and SPI details.
