# Build Custom Connector: Streaming Trigger (source → sink)

- **Manifest:** `execution.type=streaming` with `source` and `sink` maps; set `type` to a provider id (e.g., `kafka`, `eventhub`, `mongodb`, `http` sink).
- **SDK/SPI:** implement `SourceConnectorProvider` / `SinkConnectorProvider`, register via `META-INF/services`, build configs, and run `StreamingPipelineExecutor`.
- **Built-ins:** `type=kafka` (connect-kafka), `type=eventhub` (connect-eventhub), `type=mongodb` (connect-mongo), `type=http` sink (core).

## Write a custom streaming source
*Provider vs runtime:* the provider is the factory (ServiceLoader). The runtime emits records. Have the provider’s `create(...)` return your runtime—usually a subclass of `AbstractAsyncStreamingSource` (recommended) or a custom `StreamingSource` if you need bespoke threading/backpressure.

1) Provider:
```java
public class MySourceProvider implements SourceConnectorProvider {
    public SourceConnectorDescriptor descriptor() {
        return SourceConnectorDescriptor.builder("my-source", "My Streaming Source").build();
    }
    public List<ConnectorOption> options() { /* schema */ }
    public StreamingSource create(SourceConnectorContext ctx, SourceConnectorConfig cfg) {
        return new MyStreamingSource(cfg.getString("endpoint", null));
    }
}
```

2) Runtime: implement `MyStreamingSource` (extend `AbstractAsyncStreamingSource`), emit `Document` batches.

3) ServiceLoader registration:
```
src/main/resources/META-INF/services/ai.fluxion.core.engine.connectors.SourceConnectorProvider
com.acme.connectors.MySourceProvider
```

4) Manifest usage: `"source": { "type": "my-source", ... }`.

## Write a custom streaming sink
1) Provider:
```java
public class MySinkProvider implements SinkConnectorProvider {
    public SinkConnectorDescriptor descriptor() {
        return SinkConnectorDescriptor.builder("my-sink", "My Streaming Sink").build();
    }
    public List<ConnectorOption> options() { /* schema */ }
    public StreamingSink create(ConnectorConfig cfg) {
        return new MyStreamingSink(cfg.getString("endpoint", null));
    }
}
```

2) Runtime: implement `MyStreamingSink` (consumes batches of `Document`, optionally AutoCloseable).

3) ServiceLoader registration:
```
src/main/resources/META-INF/services/ai.fluxion.core.engine.connectors.SinkConnectorProvider
com.acme.connectors.MySinkProvider
```

4) Manifest usage: `"sink": { "type": "my-sink", ... }`.

Minimal executor wiring:
```java
SourceConnectorConfig src = SourceConnectorConfig.builder("my-source")
    .option("endpoint", "...")
    .build();
ConnectorConfig sink = ConnectorConfig.builder("my-sink", ConnectorConfig.Kind.SINK)
    .option("endpoint", "...")
    .build();
StreamingSource source = ConnectorFactory.createSource(src, SourceConnectorContext.from(new StreamingContext()));
StreamingSink dest = ConnectorFactory.createSink(sink);
new StreamingPipelineExecutor().processStream(source, List.of(), dest, new StreamingContext());
```
