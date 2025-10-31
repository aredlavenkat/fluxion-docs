# Platform Architecture

Fluxion packages data orchestration into a set of composable layers. Understanding
how these layers line up makes it easier to choose the right entry point for a
project, and to know which modules to extend when new requirements appear.

## Layered view

```text
Connect  →  Streaming Engine  →  Rule Engine
                   │                │
           └──────►│                │
                   ▼                │
               Fluxion Core  ◄──────┘
                     │
                   Enrich
```

- **Fluxion Core** sits at the base. It is the deterministic execution engine
  that powers aggregation pipelines, stateful windowing, and expression
  evaluation. Every other component delegates back to Core for predictable
  compute.
- **Fluxion Connect** provides connectors for sources and sinks (Kafka, HTTP,
  SQL, MongoDB, …). Connect is how data enters and exits the platform,
  independent of which runtime consumes it.
- **Fluxion Enrich** adds optional operators that reach out to external systems
  – `$httpCall`, `$sqlQuery`, and more – so enrichment logic stays inside a
  declarative pipeline instead of bespoke glue code.
- **Execution Runtimes** package the lower layers into opinionated experiences.
  - The **Rule Engine** focuses on JSON/DSL-defined rule sets suitable for
    decisioning, enrichment, and governance workloads.
  - The **Streaming Engine** orchestrates long-running event pipelines,
    combining Connect streams with Core stages and Enrich operators.

## Choosing the right entry point

| If you need…                                     | Start with…                                            |
| ------------------------------------------------ | ------------------------------------------------------ |
| Deterministic operators, aggregation semantics   | Fluxion Core reference (Stages and Operators sections) |
| New source/sink integration                      | Fluxion Connect                                        |
| External lookups or side effects inside a stage  | Fluxion Enrich                                         |
| Declarative rule authoring & governance controls | Rule Engine runtime                                    |
| Continuous stream ingestion and delivery         | Streaming Engine runtime                               |

## Extending the platform

When extending Fluxion:

1. **Add capabilities at the lowest layer possible.** A new operator or stage
   goes into Core; a new data source should live in Connect; new enrichment
   behaviours belong in Enrich.
2. **Surface the capability through the runtimes.** Once a primitive exists,
   expose it to Rule and/or Streaming engines so users immediately benefit.
3. **Document the extension path.** Both engines link back to the module
   references so contributors can trace behaviour from runtime to primitive.

This separation keeps the execution engine lean while still letting teams tailor
Fluxion to their data landscape.
