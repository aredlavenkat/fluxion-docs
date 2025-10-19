# Usage Guide

This guide walks through the core workflow for running Fluxion pipelines inside a Java service. The current focus is on single-document and request/response processing. (Future releases will add dedicated aggregation helpers.)

---

## 1. Describe the Pipeline

Pipelines reuse MongoDB’s stage syntax. Store them as JSON or build them programmatically. This example enriches a device reading with derived fields.

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

## 2. Load Documents and Stages

The helper `DocumentParser` turns JSON arrays into Fluxion `Document` and `Stage` instances. You can also build them manually if you prefer.

```java
List<Document> input = DocumentParser.getDocumentsFromJsonArray("""
  [
    { "device": "sensor-1", "status": "active", "temperatureC": 18.6 },
    { "device": "sensor-2", "status": "offline", "temperatureC": 31.2 }
  ]
""");

List<Stage> pipeline = DocumentParser.getStagesFromJsonArray(Files.readString(
    Path.of("pipelines/temperature.json")
));
```

---

## 3. Execute with `PipelineExecutor`

`PipelineExecutor` is the primary entry point for integrators. It accepts the documents, stage list, and optional globals. Each input document is evaluated independently.

```java
PipelineExecutor executor = new PipelineExecutor();
Map<String, Object> globals = Map.of("tenantId", "acme");

List<Document> transformed = executor.run(input, pipeline, globals);
```

---

## 4. Inspect Results

Documents are mutable JSON wrappers. Use `toJson()` during development or extract individual fields for downstream processing.

```java
transformed.forEach(doc -> {
    System.out.println(doc.toJson());
    boolean isHot = (boolean) doc.get("isHot");
    // continue with your business logic…
});
```

System variables such as `$$ROOT`, `$$CURRENT`, and `$$NOW` are automatically populated while the pipeline runs.

---

## 5. Going Further

- Need to extend the engine? See the [Integration Developer Guide](core/integration-developer-guide.md) for custom operators and stages.
- Looking for stage/operator syntax? Head to the [Stages](stages/index.md) and [Operators](operators/index.md) references.
- Want ready-made scenarios? Explore the [Examples Gallery](examples/exampleSet1.md).

You now have everything required to execute pipelines inside your own service. Iterate on the pipeline JSON, re-run the executor, and ship when the results match your expectations.
