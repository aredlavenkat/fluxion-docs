# Platform Architecture Overview

High-level map of Fluxion modules, runtimes, and extension points. Use this to
select the right entry point and understand how pieces compose.

![Fluxion platform architecture](../assets/platform-architecture.png)

---

## 1. Layered architecture

```text
Connect  →  Streaming Engine  →  Rule Engine
                  │                │
          └──────►│                │
                  ▼                │
              Fluxion Core  ◄──────┘
                    │
                  Enrich
```

| Layer | Purpose |
| --- | --- |
| Fluxion Core | Deterministic aggregation engine (stages/operators), expression evaluator, metrics, registries. |
| Fluxion Connect | Source/sink connectors (Kafka, Event Hubs, MongoDB, custom SPI). |
| Fluxion Enrich | Operators that call external systems (`$httpCall`, `$sqlQuery`). |
| Streaming Engine | Always-on pipelines (fetch → transform → deliver → checkpoint). |
| Rule Engine | JSON/DSL-defined rule sets for decisioning and governance workflows. |

---

## 2. Module checklist

| Feature | Module | Reference |
| --- | --- | --- |
| Aggregation runtime | `fluxion-core` | [Core Overview](../core/index.md) |
| Connectors | `fluxion-connect` | [Connect Overview](../connect/index.md) |
| Enrichment operators | `fluxion-enrich` | [Enrich Overview](../enrich/index.md) |
| Rule engine | `fluxion-rules` | [Rule Engine Overview](../rules/index.md) |
| Streaming engine | (contained in `fluxion-core` module today) | [Streaming Overview](../streaming/index.md) |

---

## 3. Selecting an entry point

| Need | Start here |
| --- | --- |
| Run aggregation pipelines in-process | `fluxion-core` (see [Usage Guide](../usage.md)) |
| Ingest from Kafka/Event Hubs/Mongo | `fluxion-connect` + Streaming Engine |
| Enrich with HTTP/SQL lookups | `fluxion-enrich` operators |
| Declarative rule authoring/governance | `fluxion-rules` (Rule Engine) |
| Long-running stream processing | Streaming Engine (with Connect + Enrich) |

---

## 4. Extending Fluxion

1. **Add capabilities at the lowest layer.**
   - New operator/stage → contribute to Fluxion Core.
   - New connector → implement SPI in Fluxion Connect.
   - New enrichment behaviour → extend Fluxion Enrich.
2. **Expose through runtimes.** Surface new operators/connectors to rule/streaming
   engines so users can adopt them immediately.
3. **Document + test.** Update docs/tests alongside the new contribution to keep
   assistants and developers aligned.

---

## 5. Integration patterns

| Scenario | Modules involved | Notes |
| --- | --- | --- |
| Request/response service with pipeline execution | Core | Cache parsed stages, use `PipelineExecutor`. |
| Manual approval workflow (Temporal) | Core + Rules + Workflow integration | See [Temporal Bridge](../workflow/temporal.md). |
| Real-time enrichment from Kafka to HTTP | Core + Connect + Enrich + Streaming | Use streaming pipeline with Kafka source and HTTP sink/operators. |
| Analytics batch job with JDBC source | Core + custom source | Extend streaming SPI or use executor in batch mode. |

---

## 6. References & next steps

- [Core Overview](../core/index.md) – runtime APIs and extension points.
- [Connect Overview](../connect/index.md) – built-in connectors and SPI.
- [Enrich Overview](../enrich/index.md) – HTTP/SQL operators.
- [Streaming Overview](../streaming/index.md) – pipeline lifecycle, error policies.
- [Rule Engine Overview](../rules/index.md) – declarative rule runtime.
- [Usage Guide](../usage.md) – end-to-end pipeline execution tutorial.
- [Examples Gallery](../examples/exampleSet1.md) – ready-made pipelines to copy.

This overview keeps the module map clear; dive into the linked guides for deeper,
LLM-ready documentation.
