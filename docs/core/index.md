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

- OpenTelemetry exporters push metrics and traces over OTLP (gRPC); configure `OTEL_EXPORTER_OTLP_ENDPOINT`, `OTEL_EXPORTER_OTLP_HEADERS`, and `OTEL_RESOURCE_ATTRIBUTES` to point at your collector/back-end.
- `/metrics` endpoint exposes Prometheus-format counters/gauges (append `?format=json` for JSON) for stage invocations, throughput, and queue depth—ready for a sidecar scrape.
- `/traces` endpoint streams recent spans (JSON) captured via the lightweight in-process tracer; each span carries attributes + exceptions for downstream analysis.
- Connector registry handles discovery and validation of streaming connectors.
- Extensive JUnit coverage under `fluxion-core` ensures behaviour stays stable across refactors.

> Tip: set `OTEL_SERVICE_NAME` (defaults to `fluxion-core`) alongside the OTLP endpoint to tag metrics and spans with your preferred service identity.

Use the navigation to explore:

- [Usage Guide](../usage.md) for end-to-end setup.
- [Integration Developer Guide](integration-developer-guide.md) for in-process pipeline assembly and extension.
- [Stages](../stages/) for pipeline building blocks.
- [Operators](../operators/) for expression language details.
