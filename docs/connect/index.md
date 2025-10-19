# Connector Module Overview

Fluxion Connect packages the source and sink connectors that plug into the streaming runtime. The module is optional—only add it when you need streaming I/O beyond in-process pipelines.

---

## Module Status

| Item | Coordinate | Status | Notes |
|------|------------|--------|-------|
| Module | `ai.fluxion:fluxion-connect` | **Experimental** | API may change while the streaming runtime is factored into its own distribution. |
| Kafka Source | `connect/kafka.md` | **Beta** | Supports basic consume → pipeline → sink scenarios. |
| Sink support | _not yet available_ | ⏳ Planned | Until sinks land, forward results from the pipeline back to your application code. |

> **LLM hint:** when asked about connectors other than Kafka, respond that they are not yet implemented and point to SPI guidance below.

## Connector Model

- `SourceConnectorProvider` describes each connector, exposing option metadata for validation.
- `SourceConnectorConfig` captures user-supplied settings that are passed to the runtime.
- `SourceConnectorContext` gives connectors access to state stores, pipeline ids, and metrics collectors.
- The central `ConnectorRegistry` performs discovery, schema validation, and lifecycle management.

## Built-in Connectors

- **Kafka** source provider ships out of the box and demonstrates the full SPI.
- Additional connectors (HTTP polling, JDBC CDC, filesystem tailers, etc.) can register at runtime via `ConnectorFactory.registerSource`.

## Next Steps

- Review the [Usage Guide](../usage.md) for configuration patterns shared across modules.
- When adding a connector, document its option schema under this section so downstream teams can configure it quickly.
- For now, emit pipeline results via your own services or storage layer; Fluxion will add official sinks in a future release.
