# Azure Event Hubs Connectors

Fluxion Connect provides an Event Hubs source and sink so pipelines can ingest from and publish back to Azure Event Hubs without additional glue code. The connectors reuse the same SPI as Kafka/HTTP, so their configuration maps cleanly onto the existing runtime concepts (queueing, batching, error policies).

---

## Source (`type: eventhub`)

Use the source connector to subscribe to an event hub and push records into the streaming pipeline. Each Event Hubs message is converted into a Fluxion `Document`; metadata such as partition id, offset, and sequence number is carried under `_meta`.

```yaml
source:
  type: eventhub
  options:
    connectionString: "Endpoint=sb://your-namespace.servicebus.windows.net/;SharedAccessKeyName=..."
    eventHubName: orders
    consumerGroup: $Default         # optional, defaults to $Default
    queueCapacity: 64               # internal queue between receiver and stages
    maxBatchSize: 128               # documents per pipeline batch
    maxWaitTime: PT5S               # how long to wait before yielding an empty batch
    startPosition: earliest         # or `latest`
    prefetchCount: 300              # optional client prefetch size
```

| Option | Description | Default |
| --- | --- | --- |
| `connectionString` | Event Hubs connection string (may include the entity path). | **Required** |
| `eventHubName` | Event hub to read from (omit if present in the connection string). | `null` |
| `consumerGroup` | Consumer group used by the receiver. | `$Default` |
| `queueCapacity` | Internal queue capacity between Event Hubs and the pipeline. | `32` |
| `maxBatchSize` | Maximum number of events delivered per pipeline batch. | `128` |
| `maxWaitTime` | Maximum wait before emitting an empty batch. | `PT5S` |
| `startPosition` | Starting position (`earliest` or `latest`). | `earliest` |
| `prefetchCount` | Prefetch size requested from Event Hubs. | `null` |

---

## Sink (`type: eventhub`)

The sink batches documents, encodes them as JSON, and publishes to the configured event hub. Failures bubble up to the streaming error policy so you can retry, dead-letter, or stop the pipeline.

```yaml
sink:
  type: eventhub
  options:
    connectionString: "Endpoint=sb://your-namespace.servicebus.windows.net/;SharedAccessKeyName=..."
    eventHubName: orders-out
    batchSize: 100                  # documents per Event Hubs batch
    flushTimeout: PT10S             # wait time for send operations
    partitionId: "0"               # optional fixed partition
    partitionKey: customer-id       # optional partition key
```

| Option | Description | Default |
| --- | --- | --- |
| `connectionString` | Event Hubs connection string (may include entity path). | **Required** |
| `eventHubName` | Event hub to publish to (omit if present in the connection string). | `null` |
| `batchSize` | Maximum documents batched per send. | `100` |
| `flushTimeout` | ISO-8601 duration to wait for a send to complete. | `PT10S` |
| `partitionId` | Optional fixed partition id for publishing. | `null` |
| `partitionKey` | Optional partition key applied to the batch. | `null` |

---

## Operational Notes

- The connectors rely on the standard `azure-messaging-eventhubs` SDK; ensure the Fluxion service can reach the namespace and that the connection string has `Listen`/`Send` permissions as appropriate.
- Combine the connectors with `StreamingErrorPolicy` to control retries, dead-letter routing, or stop-on-failure behaviour.
- When running locally without Azure access, you can supply a mock connection string so configuration validation passes, but the runtime will attempt to connect once the pipeline starts.
