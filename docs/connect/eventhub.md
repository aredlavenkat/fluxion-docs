# Azure Event Hubs Connectors

SrotaX Connect includes Event Hubs source and sink connectors so pipelines can
ingest from and publish back to Azure Event Hubs without extra glue code.

---

## 1. Prerequisites

| Requirement | Notes |
| --- | --- |
| Dependency | `ai.fluxion:fluxion-connect` plus `com.azure:azure-messaging-eventhubs`. |
| Azure namespace | Event Hubs namespace reachable from the SrotaX service. |
| Access | SAS connection string with `Listen`/`Send` permissions as needed. |
| Checkpoint store | JDBC/Redis/custom store when running streaming pipelines. |

---

## 2. Source configuration (`type: eventhub`)

### YAML snippet

```yaml
source:
  type: eventhub
  options:
    connectionString: "Endpoint=sb://your-namespace.servicebus.windows.net/;SharedAccessKeyName=..."
    eventHubName: orders
    consumerGroup: $Default
    queueCapacity: 64
    maxBatchSize: 128
    maxWaitTime: PT5S
    startPosition: earliest
    prefetchCount: 300
```

### Options

| Option | Description | Default |
| --- | --- | --- |
| `connectionString` | Azure Event Hubs connection string (can include entity path). | **Required** |
| `eventHubName` | Event hub to read from (omit if set in the connection string). | `null` |
| `consumerGroup` | Consumer group used by the receiver. | `$Default` |
| `queueCapacity` | Internal queue size between Event Hubs client and pipeline. | `32` |
| `maxBatchSize` | Maximum events delivered per pipeline batch. | `128` |
| `maxWaitTime` | Time to wait before emitting an empty batch (ISO-8601). | `PT5S` |
| `startPosition` | `earliest` or `latest`. | `earliest` |
| `prefetchCount` | Client prefetch size. | `null` |

---

## 3. Sink configuration (`type: eventhub`)

### YAML snippet

```yaml
sink:
  type: eventhub
  options:
    connectionString: "Endpoint=sb://your-namespace.servicebus.windows.net/;SharedAccessKeyName=..."
    eventHubName: orders-out
    batchSize: 100
    flushTimeout: PT10S
    partitionId: "0"
    partitionKey: customer-id
```

### Options

| Option | Description | Default |
| --- | --- | --- |
| `connectionString` | Event Hubs connection string. | **Required** |
| `eventHubName` | Event hub to publish to. | `null` |
| `batchSize` | Maximum documents batched per send. | `100` |
| `flushTimeout` | Wait for send completion (ISO-8601). | `PT10S` |
| `partitionId` | Fixed partition id. Mutually exclusive with `partitionKey`. | `null` |
| `partitionKey` | Partition key applied to the batch. | `null` |

---

## 4. Using the Event Hubs connectors in code

The Event Hub source/sink providers are discovered via ServiceLoader. Configure them via your `source`/`sink` blocks and wire them into the streaming executor:

```java
EventHubStreamingSource source = new EventHubStreamingSource(
    "Endpoint=sb://...;SharedAccessKeyName=Listen;SharedAccessKey=...",
    "orders",
    "$Default",
    64,   // queue capacity
    128,  // max batch size
    Duration.ofSeconds(5),
    300   // prefetch
);

EventHubSink sink = new EventHubSink(
    "Endpoint=sb://...;SharedAccessKeyName=Send;SharedAccessKey=...",
    "orders-out",
    100,                // batch size
    Duration.ofSeconds(10),
    null,               // partitionId
    "customer-id"       // partitionKey
);

List<Stage> stages = List.of(new Stage(Map.of("$set", Map.of("processed", true))));
StreamingRuntimeConfig runtime = StreamingRuntimeConfig.builder().queueCapacity(64).build();
StreamingPipelineExecutor executor = new StreamingPipelineExecutor(32, runtime, StreamingErrorPolicy.failFast());
executor.processStream(source, stages, sink, new StreamingContext());
```

Map YAML/JSON `options` into these constructor arguments; the providers handle Event Hubs client setup, batching, and backpressure.

---

## 5. Stream catalogs (for UIs/CLIs)

- **Source** (`discoverStreams`): `name=eventHubName`, `namespace=eventhub`, `supportedSyncModes=[FULL_REFRESH, INCREMENTAL]`, `cursorFields=["sequenceNumber"]`, `sourceDefinedCursor=true` (connector-owned).
- **Sink** (`destinationStreams`): `name=eventHubName`, `namespace=eventhub`, `supportedSyncModes=[FULL_REFRESH]`.

Fetch via `/api/connectors/discovery/sources|sinks` to keep pipeline specs connector-agnostic.

---

## 6. Troubleshooting

| Symptom | Possible cause | Remedy |
| --- | --- | --- |
| `IllegalArgumentException: connectionString missing` | Config validation failure. | Provide a connection string with `Endpoint`, `SharedAccessKeyName`, `SharedAccessKey`. |
| `com.azure.messaging.eventhubs.EventHubsException$ResourceNotFound` | Wrong `eventHubName` or insufficient permissions. | Verify event hub exists and credentials have access. |
| Events stalled | Consumer group checkpoint ahead of stream. | Change `consumerGroup` or reset checkpoints. |
| Sink retries indefinitely | Namespace unreachable or throttled. | Adjust `StreamingErrorPolicy` (dead-letter/skip) and review Azure throttling limits. |

---

## 7. Testing

- Run Event Hubs connector tests:
  ```bash
  mvn -pl fluxion-core -am test -Dtest=*EventHub*
  ```
- Local testing requires an Azure Event Hubs namespace (no official local emulator). Use a dev namespace with shared access keys.

---

## 8. References

| Path | Description |
| --- | --- |
| `fluxion-core/src/main/java/.../EventHubSourceConnectorProvider.java` | Source provider implementation. |
| `fluxion-core/src/main/java/.../EventHubSinkConnectorProvider.java` | Sink provider implementation. |
| Azure docs | [Event Hubs connection strings](https://learn.microsoft.com/azure/event-hubs/event-hubs-get-connection-string) |

Use these templates to bootstrap your streaming pipelines; tweak batching and
queue sizes based on throughput and latency requirements.
