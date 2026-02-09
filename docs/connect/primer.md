# Connectors at a Glance

An overview of how Fluxion connectors are modeled, packaged, and executed—aimed
to mirror the clarity of the Camunda connector docs.

---

## 1) What a connector is

- **Inbound runtimes (triggers):** polling, webhook, streaming, timer. They emit
  events into pipelines.
- **Outbound runtimes (actions):** HTTP/JavaBean/pipeline-call actions push data
  out or invoke downstream work.
- **Transports vs. runtimes:** transports encapsulate I/O (Kafka, EventHub,
  Mongo, HTTP). Runtimes orchestrate how that transport is executed.

---

## 2) Authoring models

| Mode | When to use | How it works |
| --- | --- | --- |
| **Manifest-first** | Declarative/config-as-code; classpath manifests | JSON manifest (`operations`, `execution` blocks). `ManifestConnectorDispatcher` runs actions/triggers from the manifest. |
| **SDK/SPI-first** | Java services needing programmatic control | Implement `SourceConnectorProvider` / `SinkConnectorProvider`, register via `ServiceLoader`, then instantiate via `ConnectorRegistry`/`ConnectorFactory`. |

Both resolve to the same providers; you can ship default manifests inside the
connector jar and still expose the SPI.

---

## 3) Execution types (renamed)

| `execution.type` | Kind | Use |
| --- | --- | --- |
| `http` | action | Call external HTTP APIs with retry/CB. |
| `javaBean` | action/trigger | Invoke app-provided beans for custom logic. |
| `pipelineCall` | action | Call another pipeline/version. |
| `polling` | trigger | Scheduled pulls from systems that don’t push. |
| `webhook` | trigger | Listen on an HTTP endpoint and emit payloads. |
| `streaming` | trigger | Long-lived source→sink streaming (e.g., Kafka→HTTP sink). |
| `timer` | trigger | Cron/interval ticks. |

---

## 4) Package layout (example)

```
my-connector/
  src/main/java/.../MySourceProvider.java      # implements SourceConnectorProvider
  src/main/java/.../MySinkProvider.java        # implements SinkConnectorProvider
  src/main/resources/META-INF/services/
    ai.fluxion.core.engine.connectors.SourceConnectorProvider
    ai.fluxion.core.engine.connectors.SinkConnectorProvider
  src/main/resources/manifests/
    my-connector.json                          # optional manifest(s)
```

- Providers define descriptor + option schema.
- Options are validated by `ConnectorRegistry` and exposed to UIs/CLIs.
- Manifests can be bundled for drop-in defaults.

---

## 5) Runtime flow

1. **Discovery:** `ConnectorRegistry` loads providers via `ServiceLoader`.
2. **Validation:** `SourceConnectorConfig` / `ConnectorConfig` validate options
   against provider schemas.
3. **Execution:** streaming uses `StreamingPipelineExecutor` (source→stages→sink
   with backpressure); actions use `ManifestConnectorDispatcher` (HTTP, bean,
   pipelineCall, webhook, polling, streaming triggers).
4. **Operators:** enrichment operators (`$httpCall`, `$sqlQuery`, etc.) resolve
   connection refs to these connectors; streaming pipelines wire sources/sinks
   by `type`.

---

## 6) Resilience, security, observability

- **Resilience:** HTTP execution supports retry/circuit-breaker headers or
  manifest fields; streaming connectors inherit backpressure/batching; timers/
  polling provide intervals and handler hooks.
- **Security:** Secrets come from `ConnectorContext::secretResolver`; keep creds
  in vaults, not manifests.
- **Observability:** `ConnectorLogger`, `ConnectorMeterRegistry`, `ConnectorTracer`
  flow through contexts—wire them to your logging/metrics/tracing backends.

---

## 7) Quick starts

- **Manifest action (HTTP):** place a manifest under `resources/manifests`, then
  `new ManifestConnectorDispatcher().executeAction(manifest, "op", ctx, body)`.
- **Manifest streaming:** use `execution.type=streaming` with `source` and `sink`
  maps (Kafka → HTTP sink). `startTrigger(...)` returns a `Flux` of documents.
- **SDK streaming:** build configs, then:

```java
StreamingSource source = ConnectorFactory.createSource(sourceCfg, SourceConnectorContext.from(new StreamingContext()));
StreamingSink sink = ConnectorFactory.createSink(sinkCfg);
new StreamingPipelineExecutor().processStream(source, stages, sink, new StreamingContext());
```

---

## 8) Choosing a model

- **Pick manifest-first** for low-code/operator consoles and tenant-level
  customization without rebuilds.
- **Pick SDK-first** when embedding in services, needing dynamic secrets, or
  building bespoke transports.
- **Mix** them when you want default manifests shipped with your connector jar
  but still expose the SPI for power users.

---

## 9) Execution examples (manifest + SDK)

**HTTP action (manifest)**
```json
{
  "operations": { "call": "call" },
  "operationDefs": {
    "call": {
      "kind": "action",
      "execution": { "type": "http", "method": "POST", "urlTemplate": "https://api.example.com/orders" }
    }
  }
}
```
**SDK**
```java
Object out = dispatcher.executeAction(manifest, "call", ctx, Map.of("orderId","A-1"));
```

**JavaBean action/trigger (manifest)**
```json
"execution": { "type": "javaBean", "beanName": "myHandler" }
```
**SDK**
```java
dispatcher.registerActionHandler("myHandler", (c, in) -> Map.of("ok", true));
dispatcher.executeAction(manifest, "myOp", ctx, Map.of());
```

**Pipeline call (manifest)**
```json
"execution": { "type": "pipelineCall", "targetPipeline": "orders", "version": "v1" }
```

**Polling trigger (manifest)**
```json
"execution": { "type": "polling", "intervalMillis": 5000, "handlerBean": "poller" }
```
**SDK**
```java
dispatcher.registerTriggerHandler("poller", (c, cfg) -> Flux.interval(Duration.ofSeconds(5)).map(i -> Map.of("tick", i)));
```

**Webhook trigger (manifest)**
```json
"execution": { "type": "webhook", "path": "/events", "method": "POST" }
```
**SDK**
```java
Flux<Map<String,Object>> flux = dispatcher.startTrigger(manifest, "hookOp", ctx, Map.of("port", 8080));
```

**Streaming trigger (manifest)**
```json
"execution": {
  "type": "streaming",
  "stream": "orders",
  "source": { "type": "kafka", "bootstrapServers": "localhost:9092", "topic": "orders", "groupId": "g1" },
  "sink":   { "type": "http",  "endpoint": "http://localhost:8081/ingest" }
}
```
**Streaming with a pre-sink process (pipeline)**
- Add a single top-level `process` block to run a pipeline before delivering to the sink/fan-out.
- Supports `type: "pipeline"` / `"pipelineCall"` with `pipeline`/`version` (defaults to `v1`).
- Output from the process pipeline becomes the input to the sink (or chained/fan-out sinks).
```json
"execution": {
  "type": "streaming",
  "stream": "orders",
  "process": { "type": "pipeline", "pipeline": "enrich-orders", "version": "v1" },
  "sink": {
    "fanout": [
      { "type": "http", "endpoint": "http://localhost:8081/a" },
      { "type": "kafka", "bootstrapServers": "localhost:9092", "topic": "orders-enriched" }
    ]
  }
}
```
**SDK**
```java
SourceConnectorConfig src = SourceConnectorConfig.builder("kafka")
    .option("bootstrapServers","localhost:9092").option("topic","orders").build();
ConnectorConfig sink = ConnectorConfig.builder("http", ConnectorConfig.Kind.SINK)
    .option("endpoint","http://localhost:8081/ingest").build();
StreamingSource source = ConnectorFactory.createSource(src, SourceConnectorContext.from(new StreamingContext()));
StreamingSink httpSink = ConnectorFactory.createSink(sink);
new StreamingPipelineExecutor().processStream(source, List.of(), httpSink, new StreamingContext());
```

**Timer trigger (manifest)**
```json
"execution": { "type": "timer", "cron": "PT30S" }
```

**Pipeline invoker & resilience (Spring Boot starter)**
- The starter auto-registers a `PipelineCallInvoker` that POSTs to the pipeline-service endpoint `/api/pipelines/{name}/{version}:run`. It is used by:
  - `execution.type=pipelineCall` actions
  - Streaming `process` blocks (top-level)
  - Streaming sinks with `type=pipeline` / `pipelineCall`
- Configure target service, timeout, and Resilience4j policies:
```yaml
fluxion:
  connect:
    pipeline:
      base-url: http://fluxion-pipeline-service:8085
      timeout-ms: 10000
resilience4j:
  retry:
    instances:
      pipeline-service:
        max-attempts: 3
        wait-duration: 500ms
  circuitbreaker:
    instances:
      pipeline-service:
        failure-rate-threshold: 50
        wait-duration-in-open-state: 5s
        permitted-number-of-calls-in-half-open-state: 3
        sliding-window-size: 10
```
- Inputs are sent as `document` in the POST body; the pipeline response (Map/Document/list) is forwarded to the next sink or returned to the caller.

---

## 10) Built-in connectors & supported executions

| Connector type | Module | Supports | Execution usage |
| --- | --- | --- | --- |
| HTTP sink (`type=http`) | fluxion-connect | Action (http), Streaming sink | `execution.type=http` for actions; in streaming manifests use `sink.type=http` to post batches. |
| Kafka (`type=kafka`) | fluxion-connect-kafka | Streaming source & sink | `execution.type=streaming` with `source.type=kafka` and/or `sink.type=kafka`. |
| EventHub (`type=eventhub`) | fluxion-connect-eventhub | Streaming source & sink | `execution.type=streaming` with `source.type=eventhub` and/or `sink.type=eventhub`. |
| MongoDB (`type=mongodb`) | fluxion-connect-mongo | Streaming source & sink | `execution.type=streaming` with `source.type=mongodb` and/or `sink.type=mongodb`. |
| JavaBean | dispatcher (handlers) | Action/trigger | `execution.type=javaBean` with `beanName`; register handlers in dispatcher. |
| Pipeline call | dispatcher | Action | `execution.type=pipelineCall` targeting another pipeline. |
| Webhook | dispatcher | Trigger | `execution.type=webhook` to start an HTTP listener and emit payloads. |
| Polling | dispatcher | Trigger | `execution.type=polling` interval or handler bean. |
| Timer | dispatcher | Trigger | `execution.type=timer` with cron/ISO-8601 duration. |

---

## 11) Next steps

- [Module overview](index.md)
- [Manifest + SDK format](manifest-and-sdk.md)
- Connector specifics: [Kafka](kafka.md), [EventHub](eventhub.md), [Mongo](mongodb.md)
- [Developer guide](developer-guide.md)
