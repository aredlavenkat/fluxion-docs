# Connector Module Overview

Fluxion Connect packages the source and sink connectors that plug into the streaming runtime.

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
- Explore the [Roadmap](../roadmap.md) to track upcoming connector work.
- When adding a connector, document its option schema under this section so downstream teams can configure it quickly.
