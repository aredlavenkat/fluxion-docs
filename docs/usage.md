# Usage Guide

Step-by-step walkthrough for running Fluxion pipelines inside a Java service. This
focuses on single-document/request-response scenarios; streaming pipelines have
separate guides.

---

## 1. Prerequisites

| Requirement | Notes |
| --- | --- |
| Fluxion module | Add `ai.fluxion:fluxion-core` to your build. |
| Runtime | Java 21+ |
| Pipeline definition | JSON file or programmatic builder. |
| Optional helpers | Caching library (e.g., Caffeine) if you want to reuse parsed stages. |

---

## 2. Describe the pipeline

Pipelines reuse MongoDB’s stage syntax. Store them in JSON or construct them in
code. Example: enrich a device reading with derived fields.

```json
[
  { "$match": { "status": "active" } },
  { "$addFields": {
      "isHot": { "$gt": ["$temperatureC", 30] },
      "temperatureF": { "$add": [{ "$multiply": ["$temperatureC", 1.8] }, 32] }
    }
  }
]
```

---

## 3. Parse documents and stages

Use `DocumentParser` to load JSON into Fluxion types (or build them manually).

```java
List<Document> input = DocumentParser.getDocumentsFromJsonArray("""
  [
    { "device": "sensor-1", "status": "active", "temperatureC": 18.6 },
    { "device": "sensor-2", "status": "offline", "temperatureC": 31.2 }
  ]
""");

List<Stage> pipeline = DocumentParser.getStagesFromJsonArray(
    Files.readString(Path.of("pipelines/temperature.json"))
);
```

---

## 4. Execute with `PipelineExecutor`

`PipelineExecutor` runs each document independently against the stage list.
Optionally pass globals.

```java
PipelineExecutor executor = new PipelineExecutor();
Map<String, Object> globals = Map.of("tenantId", "acme");
List<Document> transformed = executor.run(input, pipeline, globals);
```

---

## 5. Inspect results

Documents are mutable JSON wrappers. Log them or extract fields for downstream
processing.

```java
transformed.forEach(doc -> {
    System.out.println(doc.toJson());
    boolean isHot = (boolean) doc.get("isHot");
    alertService.recordTemperature(doc.getString("device"), isHot);
});
```

System variables (`$$ROOT`, `$$CURRENT`, `$$NOW`, etc.) are available during execution.

---

## 6. Cache parsed pipelines

Parsing JSON on every request is wasteful. Cache the `List<Stage>` once per
pipeline/tenant and reuse it.

```java
private final LoadingCache<String, List<Stage>> pipelineCache =
    Caffeine.newBuilder()
            .maximumSize(128)
            .build(name -> DocumentParser.getStagesFromJsonArray(loadPipelineJson(name)));

public List<Document> execute(String pipelineName, List<Document> input) {
    List<Stage> stages = pipelineCache.get(pipelineName);
    return executor.run(input, stages, Map.of());
}
```

Invalidate the cache entry whenever the pipeline definition changes.

---

## 7. Error-handling cheat sheet

| Exception | When it appears | Suggested response |
| --- | --- | --- |
| `IllegalArgumentException` | Stage/operator payload malformed (missing fields, wrong types). | Return a 400-style error with the message for debugging. |
| `UnsupportedOperationException` | Pipeline contains an unimplemented stage (`$merge`, `$out`, `$search`, `$vectorSearch`). | Remove/replace the stage; check the roadmap. |
| Custom `RuntimeException` | User-defined operator/stage threw an error. | Validate inputs and wrap exceptions with context. |

Wrap executor calls in try/catch blocks to translate these into your API’s error model.

---

## 8. Testing & validation

1. Write regression tests using the examples from `docs/examples/` or your own fixtures.
2. Run the core module tests to ensure behaviour matches expectations:
   ```bash
   mvn -pl fluxion-core test
   ```
3. Use debug tracing (`PipelineDebugStageTrace`) when troubleshooting stage behaviour.

---

## 9. Going further

- [Integration Developer Guide](core/integration-developer-guide.md) – custom
  operators/stages, SPI registration.
- [Stages](stages/index.md) & [Operators](operators/index.md) – reference material.
- [Examples Gallery](examples/exampleSet1.md) – advanced pipelines to copy and adapt.
- [Workflow → Temporal Bridge](workflow/temporal.md) – orchestrate rules inside Temporal workflows.

This pattern scales from simple request/response services to complex rule
engines. Iterate on the pipeline JSON, rerun the executor, and deploy when the
results look right.
