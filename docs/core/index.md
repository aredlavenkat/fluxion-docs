# Core Platform Overview

Fluxion Core provides the building blocks that power pipeline execution across every module. This page highlights the main entry points for application developers.

## Pipeline Runtime

- **`PipelineExecutor`** – primary API for running pipelines in-process. Each document is evaluated independently against the stage list.
- **`DocumentParser`** – parses JSON into `Document` and `Stage` objects so you can load definitions from config files.
- **System variables** – `$$ROOT`, `$$CURRENT`, `$$NOW`, `$$REMOVE`, and friends are populated automatically while a pipeline runs.
- **Stage & operator registries** – `StageRegistry` and `OperatorRegistry` expose 40+ stages and 200+ operators with MongoDB semantics.

## Extension Points

- Implement the `Operator` interface and register via `OperatorContributor` to add custom expressions.
- Implement `StageHandler` and register via `StageHandlerContributor` to introduce domain-specific stages.
- Use ServiceLoader descriptors (`META-INF/services/...`) so contributions are discovered automatically at runtime.

## Observability & Tooling

- `StageMetrics` captures per-stage counts and timings, with optional OTEL export via `StageMetricsOtelBridge`.
- The core module ships with comprehensive JUnit coverage to protect behaviour across refactors.
- Helper utilities in `ai.fluxion.core.util` make it easy to seed documents, compare output, and build fixtures for tests.

> Streaming connectors and orchestrators are moving to a separate module. Documentation for those components will return once the new packaging is settled.

Use the navigation to dive deeper:

- [Usage Guide](../usage.md) for an end-to-end walkthrough.
- [Integration Developer Guide](integration-developer-guide.md) for API details and extension patterns.
- [Stages](../stages/) and [Operators](../operators/) for reference material.
