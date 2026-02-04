# Build Custom Connectors (per execution type)

Use this page when you need a focused recipe for each action/trigger type—both
manifest-first and SDK/SPI-first.

## HTTP action {#http-action}
- **Manifest:** `execution.type=http` with `method`, `urlTemplate`, headers, retry/CB.
- **SDK:** `ManifestConnectorDispatcher.executeAction(manifest, op, ctx, body)`.
- **When to customize:** add auth headers, retry/CB config, or templating.

## JavaBean action/trigger {#javabean-actiontrigger}
- **Manifest:** `execution.type=javaBean`, `beanName`.
- **SDK:** register handlers on the dispatcher:
  ```java
  dispatcher.registerActionHandler("myHandler", (c, in) -> Map.of("ok", true));
  dispatcher.executeAction(manifest, "myOp", ctx, Map.of());
  ```
- **Trigger:** same beanName with `startTrigger(...)`.

## Pipeline call action {#pipeline-call-action}
- **Manifest:** `execution.type=pipelineCall`, `targetPipeline`, optional `version`.
- **SDK:** register a `PipelineCallInvoker` or default invoker on the dispatcher.

## Webhook trigger {#webhook-trigger}
- **Manifest:** `execution.type=webhook`, `path`, `method`.
- **SDK:** `dispatcher.startTrigger(manifest, op, ctx, Map.of("port", 8080))` returns a Flux of payloads.
- **Note:** Dispatcher hosts the HTTP listener; no extra provider needed.

## Polling trigger {#polling-trigger}
- **Manifest:** `execution.type=polling`, `intervalMillis`, optional `handlerBean`.
- **SDK:** register a trigger handler or rely on the default tick payloads.

## Streaming trigger (source → sink) {#streaming-trigger-source--sink}
- **Manifest:** `execution.type=streaming` with `source` and `sink` maps; set `type` to a provider id (e.g., `kafka`, `eventhub`, `mongodb`, `http` sink).
- **SDK (SPI):** implement `SourceConnectorProvider` / `SinkConnectorProvider`, register via `META-INF/services`, then build `SourceConnectorConfig` / `ConnectorConfig` and run `StreamingPipelineExecutor`.
- **Defaults:** built-in providers live in:
  - `fluxion-connect-kafka` (`type=kafka`)
  - `fluxion-connect-eventhub` (`type=eventhub`)
  - `fluxion-connect-mongo` (`type=mongodb`)
  - `fluxion-connect` HTTP sink (`type=http`)

### Write a custom streaming source
> Provider vs runtime: the provider is the factory (discovered via ServiceLoader). The runtime is the class that actually emits records. In your provider’s `create(...)` you instantiate the runtime—typically a subclass of `AbstractAsyncStreamingSource` (for built-in worker/queue/end-of-stream handling) or a custom `StreamingSource` if you need a different threading model.
1. Implement `SourceConnectorProvider`:
   ```java
   public class MySourceProvider implements SourceConnectorProvider {
       public SourceConnectorDescriptor descriptor() {
           return SourceConnectorDescriptor.builder("my-source", "My Streaming Source").build();
       }
       public List<ConnectorOption> options() { /* declare schema */ }
       public StreamingSource create(SourceConnectorContext ctx, SourceConnectorConfig cfg) {
           return new MyStreamingSource(cfg.getString("endpoint", null));
       }
   }
   ```
2. Implement `MyStreamingSource` by extending `AbstractAsyncStreamingSource` or implementing `StreamingSource`, emitting `Document` batches.
3. Register via `src/main/resources/META-INF/services/ai.fluxion.core.engine.connectors.SourceConnectorProvider`.
4. Use in a manifest: `"source": { "type": "my-source", ... }`.

### Write a custom streaming sink
1. Implement `SinkConnectorProvider`:
   ```java
   public class MySinkProvider implements SinkConnectorProvider {
       public SinkConnectorDescriptor descriptor() {
           return SinkConnectorDescriptor.builder("my-sink", "My Streaming Sink").build();
       }
       public List<ConnectorOption> options() { /* declare schema */ }
       public StreamingSink create(ConnectorConfig cfg) {
           return new MyStreamingSink(cfg.getString("endpoint", null));
       }
   }
   ```
2. Implement `MyStreamingSink` (accept batches of `Document`), optionally AutoCloseable.
3. Register via `src/main/resources/META-INF/services/ai.fluxion.core.engine.connectors.SinkConnectorProvider`.
4. Use in a manifest: `"sink": { "type": "my-sink", ... }`.

Minimal `StreamingPipelineExecutor` wiring:
```java
SourceConnectorConfig src = SourceConnectorConfig.builder("my-source").option("endpoint", "...").build();
ConnectorConfig sink = ConnectorConfig.builder("my-sink", ConnectorConfig.Kind.SINK).option("endpoint", "...").build();
StreamingSource source = ConnectorFactory.createSource(src, SourceConnectorContext.from(new StreamingContext()));
StreamingSink dest = ConnectorFactory.createSink(sink);
new StreamingPipelineExecutor().processStream(source, List.of(), dest, new StreamingContext());
```

## Timer trigger {#timer-trigger}
- **Manifest:** `execution.type=timer`, `cron` (ISO-8601 duration or cron string).
- **SDK:** `dispatcher.startTrigger(manifest, op, ctx, Map.of())` emits ticks.

---

See also:
- [Primer](primer.md) for end-to-end context and examples.
- [Manifest & SDK](manifest-and-sdk.md) for the full manifest schema and SPI details.
