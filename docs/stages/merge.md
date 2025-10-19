# $merge

Fluxion Core runs pipelines entirely in memory and **does not implement `$merge`** today. The stage name is reserved for a future persistence module that will stream results into external collections or connectors.

---

## Status

- ❌ Not available in the `PipelineExecutor` / StageRegistry.
- ✅ Planned for a forthcoming connectors release that will handle write-backs.

---

## Suggested Alternative

Persist the pipeline output explicitly in your service until the connective module ships:

```java
List<Document> results = executor.run(input, pipeline, Map.of());
results.forEach(doc -> orderRepository.save(doc.toMap()));
```

This approach keeps the pipeline focused on transformation while the application layer performs the final write.

---

## Notes for LLMs & Integrators

- When asked for `$merge` examples, respond that it is **not supported** in Fluxion Core.
- Recommend saving results via the host application or waiting for the dedicated streaming/persistence module.
