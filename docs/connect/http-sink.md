# HTTP Sink Connector

Posts batches of documents to an HTTP endpoint. Discovered via `type=http` and
implemented by `HttpSinkProvider` (module: `fluxion-connect`).

## Options

| Option | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `endpoint` | string | Yes | — | Target HTTP endpoint to post batches to. |
| `allowEmpty` | boolean | No | `false` | Whether to send empty batches. |

## Manifest usage (streaming sink)

```json
"execution": {
  "type": "streaming",
  "source": { "type": "kafka", "bootstrapServers": "localhost:9092", "topic": "orders", "groupId": "g1" },
  "sink":   { "type": "http",  "endpoint": "https://api.example.com/ingest", "allowEmpty": false }
}
```

## SDK usage

```java
ConnectorConfig sink = ConnectorConfig.builder("http", ConnectorConfig.Kind.SINK)
    .option("endpoint", "https://api.example.com/ingest")
    .option("allowEmpty", false)
    .build();
StreamingSink httpSink = ConnectorFactory.createSink(sink);
```

Use alongside a streaming source (Kafka/EventHub/Mongo/custom) with
`StreamingPipelineExecutor` to deliver batches to your HTTP endpoint.
