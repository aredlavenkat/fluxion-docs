# Core Platform Overview

Fluxion Core delivers the aggregation engine, streaming runtime, and shared services that power every module in the platform.

## Aggregation Engine

- MongoDB-compatible pipeline language with 200+ operators and 40+ stages.
- Expression evaluation reuses the same engine across batch and streaming flows.
- System variables (`$$ROOT`, `$$CURRENT`, `$$NOW`, `$$REMOVE`, etc.) are fully supported.

## Streaming Runtime

- `StreamingPipelineOrchestrator` wires sources, stages, and sinks using `StreamingPipelineDefinition`.
- `StreamingRuntimeConfig` exposes knobs for queue capacity, worker pool size, and backpressure.
- Configurable error policies (`StreamingErrorPolicy`, skip/dead-letter routing) keep pipelines resilient.
- Sources implement lifecycle hooks (pause, resume, cancel, checkpoint) through `StreamingSource`.

## Observability & Tooling

- Built-in metrics reporters and health probes for runtime insight.
- Connector registry handles discovery and validation of streaming connectors.
- Extensive JUnit coverage under `fluxion-core` ensures behaviour stays stable across refactors.

Use the navigation to explore:

- [Usage Guide](../usage.md) for end-to-end setup.
- [Stages](../stages/) for pipeline building blocks.
- [Operators](../operators/) for expression language details.
