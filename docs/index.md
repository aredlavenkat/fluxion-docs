# SrotaX Pipeline Engine

SrotaX lets you execute Mongo-style pipelines directly in your JVM services. It preserves the familiar stage and operator vocabulary while focusing on single-document and request/response scenarios today (bulk aggregation support will arrive later).

## Key Capabilities

- End-to-end pipeline execution via `PipelineExecutor`
- 200+ documented operators and 40+ stages with Mongo semantics
- System variables such as `$$ROOT`, `$$CURRENT`, `$$NOW`, `$$REMOVE`
- JSON-first workflow with helpers for parsing pipelines and documents
- Extension hooks for custom operators or stages

## Quick Start

```java
List<Document> input = DocumentParser.getDocumentsFromJsonArray("""
  [
    { "device": "sensor-1", "status": "active", "temperature": 18.6 },
    { "device": "sensor-1", "status": "active", "temperature": 19.1 }
  ]
""");

List<Stage> pipeline = DocumentParser.getStagesFromJsonArray("""
  [
    { "$match": { "status": "active" } },
    { "$addFields": { "fahrenheit": { "$add": [{ "$multiply": ["$temperature", 1.8] }, 32] } } }
  ]
""");

PipelineExecutor executor = new PipelineExecutor();
List<Document> transformed = executor.run(input, pipeline, Map.of());
```

Check the [Usage Guide](usage.md) for a step-by-step walkthrough and additional examples.

## Navigate the Docs

- 📘 [Usage Guide](usage.md) – load documents, run pipelines, inspect results
- 🧰 [Integration Developer Guide](core/integration-developer-guide.md) – API details and extension patterns
- 📊 [Stages Reference](stages/index.md) – payload shapes and examples
- 🧮 [Operators Reference](operators/index.md) – syntax and behaviour
- 🧠 [Glossary](glossary.md) – terminology used across the project
- 🗺️ [Roadmap](roadmap.md) – upcoming features and milestones

SrotaX is designed to embed cleanly into your applications. Start experimenting with the quick start above, then dive into the references when you need exact syntax. Continuous feedback is welcome!
