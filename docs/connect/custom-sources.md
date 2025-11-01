# Custom Sources & JDBC Integration

Build bespoke streaming sources (e.g., JDBC, REST, proprietary queues) by
implementing the streaming SPI. This guide shows how to implement a JDBC-backed
source, signal end-of-stream, and resume from checkpoints.

---

## 1. Prerequisites

| Requirement | Notes |
| --- | --- |
| Dependency | `ai.fluxion:fluxion-core` (streaming runtime) and your JDBC driver. |
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

## 3. Minimal JDBC source example

```java
class JdbcStreamingSource extends AbstractAsyncStreamingSource {
    private final DataSource dataSource;
    private final int batchSize;
    private long offset;
    private boolean finished;

    JdbcStreamingSource(DataSource dataSource, int queueCapacity, int batchSize) {
        super(queueCapacity);
        this.dataSource = dataSource;
        this.batchSize = batchSize;
    }

    @Override
    protected List<Document> poll() {
        if (finished) {
            return null; // signals end-of-stream
        }
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(
                     "SELECT id, amount FROM orders WHERE id > ? ORDER BY id ASC LIMIT ?")) {
            ps.setLong(1, offset);
            ps.setInt(2, batchSize);
            try (ResultSet rs = ps.executeQuery()) {
                List<Document> batch = new ArrayList<>();
                while (rs.next()) {
                    Document doc = new Document();
                    doc.put("_id", rs.getLong("id"));
                    doc.put("amount", rs.getBigDecimal("amount"));
                    batch.add(doc);
                    offset = rs.getLong("id");
                }
                if (batch.isEmpty()) {
                    finished = true;
                }
                return batch;
            }
        } catch (SQLException e) {
            throw new RuntimeException("JDBC source failed", e);
        }
    }
}
```

**Highlights**

- Returning `null` tells `AbstractAsyncStreamingSource` to enqueue an
  end-of-stream marker. The executor flushes remaining stages and closes sinks.
- Wrap polling in retry/backoff logic (or rely on `StreamingErrorPolicy`) if
  your data source can transiently fail.
- Persist `offset` (primary key) to resume after restarts.

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
| Source implementation | Converts each record to a Fluxion `Document`. |
| Queue/backpressure | Tune `queueCapacity` to balance latency vs. memory. |
| Checkpoints | Persist offsets/resume tokens via `StreamingContext`. |
| Completion signal | Return `null` or empty batch when no records remain. |
| Error handling | Combine with `StreamingErrorPolicy` for retries/DLQs. |

---

## 7. Batch job pattern

Treat finite sources (JDBC, CSV) like batch jobs:

- Iterate rows sequentially.
- Run decision logic inside Fluxion stages.
- Use sinks to update downstream systems.
- Once the source is exhausted and signals completion, the executor terminates.

You can still enable parallelism by adjusting `StreamingRuntimeConfig` (e.g.,
`directHandoff(false)`, custom worker pools).

---

## 8. Testing

- Write unit tests for your source implementation with in-memory databases (H2).
- Run streaming tests:
  ```bash
  mvn -pl fluxion-core -am test -Dtest=*StreamingPipeline*
  ```

---

## 9. References

| Path | Description |
| --- | --- |
| `fluxion-core/src/main/java/.../StreamingSource.java` | Core source interface. |
| `fluxion-core/src/main/java/.../AbstractAsyncStreamingSource.java` | Base class with polling thread + queue. |
| `fluxion-core/src/main/java/.../StreamingPipelineExecutor.java` | Orchestrator entry point. |
| `fluxion-docs/docs/streaming/quickstart.md` | Pipeline example (Kafka → HTTP). |

Use this template to add JDBC or other bespoke sources to the Fluxion streaming runtime.
