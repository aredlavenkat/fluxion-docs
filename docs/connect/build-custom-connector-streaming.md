# Build Custom Connector: Streaming Trigger (source → sink)

**When to use:** long-lived pipelines that read from a source and deliver to a
sink. Built-in provider ids: `kafka`, `eventhub`, `mongodb`, `http` sink.

## Manifest
```json
"execution": {
  "type": "streaming",
  "source": { "type": "kafka", "bootstrapServers": "localhost:9092", "topic": "orders", "groupId": "g1" },
  "sink":   { "type": "http",  "endpoint": "https://api.example.com/ingest" }
}
```

## SDK/SPI overview
- Implement providers via `SourceConnectorProvider` / `SinkConnectorProvider`.
- Register via `META-INF/services`.
- Build configs, then run `StreamingPipelineExecutor`.

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

## Minimal executor wiring
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

## Full example: custom source provider

Manifest (`src/main/resources/manifests/orders.stream.json`):
```json
{
  "schemaVersion": "1.0.0",
  "id": "demo.orders.stream",
  "version": "1.0.0",
  "operations": { "orders": { "$ref": "#/operationDefs/orders" } },
  "operationDefs": {
    "orders": {
      "operationId": "orders",
      "kind": "trigger",
      "execution": {
        "type": "streaming",
        "stream": "orders",
        "source": { "type": "demo-orders" },
        "sink": { "type": "http", "endpoint": "https://my.service/ingest" }
      }
    }
  }
}
```

Source provider (`OrdersStreamProvider.java`):
```java
public final class OrdersStreamProvider implements SourceConnectorProvider {
    @Override
    public SourceConnectorDescriptor descriptor() {
        return SourceConnectorDescriptor.builder("demo-orders", "Demo Orders Stream").build();
    }
    @Override
    public List<ConnectorOption> options() { return List.of(/* schema */); }
    @Override
    public StreamingSource create(SourceConnectorContext ctx, SourceConnectorConfig cfg) {
        ReactiveOrdersClient client = new ReactiveOrdersClient(cfg.getString("endpoint", null));
        return new OrdersStreamingSource(client); // implement emitting Documents
    }
}
```

ServiceLoader (`src/main/resources/META-INF/services/ai.fluxion.core.engine.connectors.SourceConnectorProvider`):
```
com.acme.connectors.OrdersStreamProvider
```
