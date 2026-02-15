# Core Platform Overview

SrotaX Core is the foundation for every module. It houses the pipeline executor,
JSON parsers, stage/operator registries, and extension points you use to embed
SrotaX in applications or build custom stages.

---

## 1. Prerequisites

| Requirement | Notes |
| --- | --- |
| SrotaX dependency | Add `ai.fluxion:fluxion-core` to your build. |
| Runtime | Java 21+ (same baseline as the rest of the platform). |
| JSON pipelines | Store pipeline definitions as JSON or build them programmatically. |
| Optional | `fluxion-connect`, `fluxion-rules` when needed. |

---

## 2. Runtime components

| Component | Purpose |
| --- | --- |
| `PipelineExecutor` | Evaluates documents against a list of stages. |
| `DocumentParser` | Parses JSON into `Document`/`Stage` objects. |
| `Document` | Mutable JSON wrapper representing a record. |
| `Stage` | Single pipeline stage (MongoDB syntax). |
| System variables | `$$ROOT`, `$$CURRENT`, `$$NOW`, `$$REMOVE`, etc. available during evaluation. |
| Registries | `StageRegistry`, `OperatorRegistry`, `ExpressionRegistry` expose built-in logic. |

### Sample usage

```java
List<Document> input = DocumentParser.getDocumentsFromJsonArray(jsonString);
List<Stage> stages = DocumentParser.getStagesFromJsonArray(pipelineJson);
PipelineExecutor executor = new PipelineExecutor();
List<Document> output = executor.run(input, stages, Map.of());
```

### Nested pipelines with `$subPipeline`

Use `$subPipeline` when you need to invoke child pipelines (registered or inline) from within a parent pipeline.

Sequential example:

```json
{
  "$subPipeline": [
    "cleanse@1",
    { "pipeline": [ { "$addFields": { "processed": true } } ] },
    "post-process"
  ]
}
```

Parallel example:

```json
{
  "$subPipeline": [
    {
      "parallel": [ "enrich-geo", "enrich-fraud" ],
      "merge": "facet"
    }
  ]
}
```

- Each entry in the array executes in order. A string references a registered pipeline (`name@version`), while `{ "pipeline": [ ... ] }` inlines stages.
- `parallel` lets you run multiple child pipelines concurrently; set `merge` to `concat`, `zip`, or `facet` to control how the branch outputs are combined.

---

## 3. Extension points

| Extension | How to | Notes |
| --- | --- | --- |
| Custom operator | Implement `Operator`, register via `OperatorContributor` (`META-INF/services`). | Used for domain-specific expressions. |
| Custom stage | Implement `StageHandler`, register via `StageHandlerContributor`. | Great for bespoke aggregation stages. |
| Custom pipeline action | Implement `PipelineAction` for reusable action logic. | Often used in rule engine integrations. |

For detailed examples see
[`core/integration-developer-guide.md`](integration-developer-guide.md).

---

## 4. Observability toolkit

| Tool | Description |
| --- | --- |
| `StageMetrics` | Captures per-stage counters and timings. |
| `StageMetricsOtelBridge` | Emits metrics to OpenTelemetry. |
| Debug tracing | Enable via `PipelineDebugStageTrace` for stage-by-stage inspection. |
| Test fixtures | Helpers in `ai.fluxion.core.util` generate documents and verify output. |

Run core tests to validate your extensions:

```bash
mvn -pl fluxion-core test
```

---

## 5. Reading guide

| Section | Use it when |
| --- | --- |
| [Usage Guide](../usage.md) | End-to-end tutorial with parsing + executor samples. |
| [Integration Developer Guide](integration-developer-guide.md) | Custom operators, stages, and SPI registration. |
| [Stages Reference](../stages/index.md) | Detailed stage semantics. |
| [Operators Reference](../operators/index.md) | Expression/operator catalogue. |
| [Examples Gallery](../examples/exampleSet1.md) | Ready-made pipelines to copy/adapt. |
| [Observability & Metrics](observability.md) | Stage metrics and OpenTelemetry bridge. |

---

## 6. Reference source files

| Path | Description |
| --- | --- |
| `fluxion-core/src/main/java/.../PipelineExecutor.java` | Core executor implementation. |
| `fluxion-core/src/main/java/.../DocumentParser.java` | JSON parsing utilities. |
| `fluxion-core/src/main/java/.../StageRegistry.java` | Stage discovery and registration. |
| `fluxion-core/src/main/java/.../OperatorRegistry.java` | Operator discovery and registration. |
| `fluxion-core/src/test/java/...` | Regression tests covering pipeline behaviour. |

Keep these references handy when extending SrotaX or integrating it into new services.
