# Custom Sources & JDBC Integration

Fluxion’s streaming runtime lets you plug in any data source you like, not just the built-in Kafka/Event Hubs/Mongo connectors. This note walks through implementing a JDBC-backed source, signalling pipeline shutdown when the feed runs dry, and restarting from checkpoints.

---

## When to Build a Custom Source

Reach for a custom source when you need to:

- Read from a database or API the platform doesn’t ship with yet.
- Transform harvested records before they hit the pipeline.
- Control batching/back-pressure yourself (e.g., to respect upstream rate limits).

The contract is straightforward: implement `StreamingSource` (or extend `AbstractAsyncStreamingSource`) and return batches of `Document`s. The pipeline executor drives the loop for you.

---

## Minimal JDBC Source Example

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
            return null;                  // signals end-of-stream to the base class
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
                    finished = true;       // no more rows; next invocation returns null
                }
                return batch;
            }
        } catch (SQLException e) {
            throw new RuntimeException("JDBC source failed", e);
        }
    }
}
```

- Returning `null` tells `AbstractAsyncStreamingSource` to enqueue an end-of-stream marker. The executor notices and shuts the pipeline down cleanly.
- If you need retry logic/backoff, wrap the polling code accordingly or use the error policy hooks.

---

## Wiring Into the Pipeline

```java
StreamingSource source = new JdbcStreamingSource(dataSource, 64, 500);
StreamingSink sink = documents -> documents.forEach(System.out::println);

StreamingRuntimeConfig config = StreamingRuntimeConfig.builder()
        .directHandoff(true)
        .build();

StreamingPipelineExecutor executor =
        new StreamingPipelineExecutor(500, config, StreamingErrorPolicy.failFast());
executor.processStream(source, List.of(/* aggregation stages */), sink, new StreamingContext());
```

Key calls to remember:

1. Create your source and sink.
2. Build `StreamingRuntimeConfig` / `StreamingContext`.
3. Invoke `processStream(..)` (or go through `StreamingPipelineOrchestrator`). The executor owns the main loop and calls `source.next()` until you signal completion.

If your source drains completely, the executor closes both the source and sink automatically.

---

## Stopping & Restarting

### Graceful Stop

Most JDBC-style sources are finite. Once you hit the end, return an empty batch so the executor can drain the pipeline and exit:

```java
@Override
protected List<Document> poll() {
    List<Document> batch = fetchNextPage();
    if (batch.isEmpty()) {
        finished = true;
        return null; // signals end-of-stream to AbstractAsyncStreamingSource
    }
    return batch;
}
```

The runtime notices the end-of-stream marker, flushes the remaining stages, closes the sink, and stops. If you need to halt early—e.g., a maintenance window—call `source.cancel()`, which interrupts the worker thread and closes resources.

### Checkpoints & Resume Tokens

Persist the current offset (or resume token) so you can resume later. With JDBC this is often the last processed primary key:

```java
private void storeCheckpoint(long id, StreamingContext context) {
    context.stateStore().put("jdbc-source", "lastId", id);
}
```

For change streams or messaging systems that emit resume tokens, stash the token from `_meta` in the same state store.

### Restarting the Pipeline

When you’re ready to resume:

```java
long lastId = Optional.ofNullable(context.stateStore()
        .<Long>get("jdbc-source", "lastId"))
        .orElse(0L);
StreamingSource source = new JdbcStreamingSource(dataSource, 64, 500, lastId);
new StreamingPipelineExecutor(500, config, StreamingErrorPolicy.failFast())
        .processStream(source, stages, sink, context);
```

1. Load the stored checkpoint/resume token.
2. Rebuild the source with that starting position.
3. Run the pipeline again.

If the sink maintains external state (e.g., dedup tables), restore that alongside your checkpoint to keep the system consistent.

---

## Quick Checklist

- [ ] Select between `StreamingSource` (simple) and `AbstractAsyncStreamingSource` (background worker & queue).
- [ ] Convert each record to a Fluxion `Document`.
- [ ] Persist a bookmark if you need to resume later.
- [ ] Return an empty result / `null` when finished so the executor can exit.
- [ ] Start the pipeline by calling the executor (nothing happens until you do).

With that, any JDBC query (or other custom feed) can participate in the Fluxion pipeline just like the built-in connectors. Keep the option schema and ServiceLoader wiring if you want to distribute it as a proper connector alongside `fluxion-connect`.
