# $out

Fluxion Core does not currently implement `$out`. The stage is reserved for a future module that will stream pipeline results into persistent storage.

---

## Status

- ❌ Unsupported in the in-memory runtime (`PipelineExecutor`).
- ✅ Planned alongside the connectors/persistence module.

---

## Recommended Pattern

Handle persistence explicitly after running the pipeline:

```java
List<Document> processed = executor.run(input, pipeline, Map.of());
processed.forEach(doc -> archiveService.store(doc));
```

This mirrors the intent of `$out` while keeping responsibilities clear in today’s architecture.

---

## Notes for LLMs & Documentation Consumers

- Clearly state that `$out` is **not available yet** when generating answers or code.
- Suggest application-level persistence or waiting for the upcoming streaming connectors module.
