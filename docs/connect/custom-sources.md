# Custom Sources & JDBC Integration

Build bespoke streaming sources (e.g., JDBC, REST, proprietary queues) by
implementing the streaming SPI. This guide shows how to implement a JDBC-backed
source, signal end-of-stream, and resume from checkpoints.

---

## 1. Prerequisites

| Requirement | Notes |
| --- | --- |
| Dependency | `ai.fluxion:fluxion-connect` (streaming runtime + registries) and your JDBC driver. |
| Data source | Database/API with incremental offsets (primary key, timestamp, resume token). |
| Checkpoint store | Persist offsets using `StreamingContext.stateStore()` or external storage. |

---

## 2. When to build a custom source

- No built-in connector exists for your system.
- You need custom batching/backpressure behaviour.
- You want to enrich/transform records before they hit the pipeline.

The contract is straightforward: implement `StreamingSource` or extend
`AbstractAsyncStreamingSource`, returning batches of `Document`s. The pipeline
executor owns polling, backpressure, and shutdown.

---

## 3. Minimal JDBC streaming source (revised)

```java
class JdbcStreamingSource extends AbstractAsyncStreamingSource {
    private final DataSource dataSource;
    private final int batchSize;
    private long offset;
    private boolean finished;

    JdbcStreamingSource(DataSource dataSource, int queueCapacity, int batchSize, long startOffset) {
        super(queueCapacity);
        this.dataSource = dataSource;
        this.batchSize = batchSize;
        this.offset = startOffset;
    }

    @Override
    protected List<Document> poll() {
        if (finished) {
            return null; // signal end-of-stream to executor
        }
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement("""
                 SELECT id, amount
                 FROM orders
                 WHERE id > ?
                 ORDER BY id ASC
                 LIMIT ?
             """)) {
            ps.setLong(1, offset);
            ps.setInt(2, batchSize);
            try (ResultSet rs = ps.executeQuery()) {
                List<Document> batch = new ArrayList<>();
                while (rs.next()) {
                    Document doc = new Document();
                    long id = rs.getLong("id");
                    doc.put("_id", id);
                    doc.put("amount", rs.getBigDecimal("amount"));
                    batch.add(doc);
                    offset = id;
                }
                if (batch.isEmpty()) {
                    finished = true;
                } else {
                    persistCheckpoint(offset);
                }
                return batch;
            }
        } catch (SQLException e) {
            throw new RuntimeException("JDBC source failed", e);
        }
    }

    private void persistCheckpoint(long id) {
        // store to StreamingContext.stateStore() in a real implementation
    }
}
```

**Highlights**
- Return `null` to end the stream; empty batch marks completion.
- Persist `offset` so you can resume after restarts.
- Wrap polling in retry/backoff or use `StreamingErrorPolicy` for transient DB errors.

---

## 4. Wiring into a pipeline

```java
StreamingSource source = new JdbcStreamingSource(dataSource, 64, 500);
StreamingSink sink = documents -> documents.forEach(System.out::println);

StreamingRuntimeConfig config = StreamingRuntimeConfig.builder()
        .directHandoff(true)
        .build();

StreamingPipelineExecutor executor =
        new StreamingPipelineExecutor(500, config, StreamingErrorPolicy.failFast());
executor.processStream(source, stages, sink, new StreamingContext());
```

Steps:

1. Instantiate source and sink.
2. Build `StreamingRuntimeConfig` and `StreamingContext`.
3. Invoke `processStream(..)` or use `StreamingPipelineOrchestrator`.

Nothing happens until you call the executor.

---

## 5. Stopping & restarting

### Signal completion

```java
if (batch.isEmpty()) {
    finished = true;
    return null; // executor shuts the pipeline down
}
```

### Persist checkpoints

```java
private void storeCheckpoint(long id, StreamingContext context) {
    context.stateStore().put("jdbc-source", "lastId", id);
}
```

### Resume from checkpoint

```java
long lastId = Optional.ofNullable(context.stateStore()
        .<Long>get("jdbc-source", "lastId"))
        .orElse(0L);
StreamingSource source = new JdbcStreamingSource(dataSource, 64, 500 /* batch */, lastId);
new StreamingPipelineExecutor(500, config, StreamingErrorPolicy.failFast())
        .processStream(source, stages, sink, context);
```

Also restore sink state if downstream systems require idempotence.

---

## 6. Checklist

| Step | Ensure |
| --- | --- |
| Source implementation | Converts each record to a SrotaX `Document`. |
| Queue/backpressure | Tune `queueCapacity` to balance latency vs. memory. |
| Checkpoints | Persist offsets/resume tokens via `StreamingContext`. |
| Completion signal | Return `null` or empty batch when no records remain. |
| Error handling | Combine with `StreamingErrorPolicy` for retries/DLQs. |

---

## 7. Batch job pattern

Treat finite sources (JDBC, CSV) like batch jobs:

- Iterate rows sequentially.
- Run decision logic inside SrotaX stages.
- Use sinks to update downstream systems.
- Once the source is exhausted and signals completion, the executor terminates.

You can still enable parallelism by adjusting `StreamingRuntimeConfig` (e.g.,
`directHandoff(false)`, custom worker pools).

---

## 8. Testing

- Write unit tests for your source implementation with in-memory databases (H2).
- Run streaming tests:
  ```bash
  mvn -pl fluxion-connect -am test -Dtest=*StreamingPipeline*
  ```

---

## 9. Discover/catalog (optional)

If your source exposes a catalog (for UI/CLI use), implement `discoverStreams`
on your `SourceConnectorProvider` and return `ConnectorStreamDescriptor` entries:

- `name`, `namespace` — identify the stream.
- `jsonSchema` — optional schema hints for tooling.
- `cursorFields` / `sourceDefinedCursor` — set `sourceDefinedCursor=true` if the
  connector owns resume tokens (Kafka offsets, JDBC PKs, custom tokens).
  resume tokens). Otherwise list candidate cursor fields.
- `primaryKeys` — optional PK hints for sinks that upsert/replace.
- `supportedSyncModes` — `FULL_REFRESH` and/or `INCREMENTAL`.

Pipeline definitions stay connector-agnostic; discovery is served via the
pipeline-service discovery endpoints.

---

## 10. Manifest-driven custom source (complete example)

**1) Implement and register your provider**
```java
// src/main/java/com/acme/connectors/OrdersSourceProvider.java
public class OrdersSourceProvider implements SourceConnectorProvider {
    public SourceConnectorDescriptor descriptor() {
        return SourceConnectorDescriptor.builder("acme-orders", "Acme Orders Source").build();
    }
    public List<ConnectorOption> options() {
        return List.of(
            ConnectorOption.builder("endpoint", ConnectorOptionType.STRING).required(true).build(),
            ConnectorOption.builder("apiKey", ConnectorOptionType.STRING).build()
        );
    }
    public StreamingSource create(SourceConnectorContext ctx, SourceConnectorConfig cfg) {
        String endpoint = cfg.getString("endpoint", null);
        String apiKey = cfg.getString("apiKey", null);
        return new OrdersStreamingSource(endpoint, apiKey); // emits Documents
    }
}
```
Register it:
```
# src/main/resources/META-INF/services/ai.fluxion.core.engine.connectors.SourceConnectorProvider
com.acme.connectors.OrdersSourceProvider
```

**2) Manifest wiring the source into a streaming trigger**
```json
{
  "schemaVersion": "1.0.0",
  "id": "acme.orders.stream",
  "version": "1.0.0",
  "displayName": "Acme Orders Stream",
  "operations": { "stream": "stream" },
  "operationDefs": {
    "stream": {
      "kind": "trigger",
      "execution": {
        "type": "streaming",
        "stream": "orders",
        "source": {
          "type": "acme-orders",
          "endpoint": "https://api.acme.com/orders",
          "apiKey": "{{ACME_API_KEY}}"
        },
        "sink": {
          "type": "http",
          "endpoint": "https://my.service/ingest"
        }
      }
    }
  }
}
```

**3) Run it via the dispatcher**
```java
ConnectorManifest manifest = new ConnectorManifestLoader()
    .load(Path.of("manifests/acme-orders-stream.json"));
ManifestConnectorDispatcher dispatcher = new ManifestConnectorDispatcher();
ConnectorContext ctx = new SimpleConnectorContext(/* tenant, pipeline, etc. */);

Flux<Map<String,Object>> events =
    dispatcher.startTrigger(manifest, "stream", ctx, Map.of());

events.take(10).collectList().block(); // consume
```

---

## 11. References

| Path | Description |
| --- | --- |
| `fluxion-core/src/main/java/.../StreamingSource.java` | Core source interface. |
| `fluxion-core/src/main/java/.../AbstractAsyncStreamingSource.java` | Base class with polling thread + queue. |
| `fluxion-core/src/main/java/.../StreamingPipelineExecutor.java` | Orchestrator entry point. |
| `fluxion-core/src/main/java/.../ConnectorStreamDescriptor.java` | Catalog metadata for streams (name/namespace/schema/cursor hints). |
| `fluxion-core/src/main/java/.../SourceConnectorProvider.java` | Override `discoverStreams` to expose catalogs. |
| `https://docs.srotax.com/streaming/quickstart/` | Pipeline example (Kafka → HTTP). |
| `https://docs.srotax.com/connect/` | Connector overview and discovery usage. |

Use this template to add JDBC or other bespoke sources to the SrotaX streaming runtime.
