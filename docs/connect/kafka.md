# Kafka Source Connector

The Kafka connector demonstrates the full Fluxion connector SPI. It handles registration, option validation, and streaming lifecycle management.

## Configuration

```yaml
source:
  type: kafka
  options:
    bootstrapServers: "localhost:9092"
    topic: "orders"
    groupId: "enrich-pipeline"
```

| Option | Description |
| --- | --- |
| `bootstrapServers` | Comma-separated Kafka brokers. |
| `topic` | Topic to subscribe to. |
| `groupId` | Consumer group identifier. |

## Notes

- Uses `SourceConnectorProvider` to expose metadata and option definitions.
- Integrates with `StreamingPipelineOrchestrator` to honour pause/resume/cancel hooks.
- Add authentication, schema registry, and custom deserialisers as needed. Document new fields here when you extend the connector.
