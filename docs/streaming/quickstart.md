# Streaming Engine Quick Start

This walkthrough demonstrates how to stand up a simple streaming pipeline that
ingests events from Kafka, applies a SrotaX Core aggregation, and delivers the
results to an HTTP endpoint.

## Prerequisites

| Requirement | Notes |
| --- | --- |
| SrotaX modules | `fluxion-core`, `fluxion-connect` (includes enrichment operators). |
| Kafka cluster | Bootstrap servers + topic for ingest. |
| Runtime | Java 21+ (examples use records, builders, switch expressions). |
| State store | Implementation of `StateStore` for offsets (examples use the in-memory store). |

## 1. Define the aggregation stages

Create the same JSON/DSL snippet you would run inside the Rule Engine. The
Streaming Engine executes identical stages, so behavioural parity is guaranteed.
Persisting or forwarding the results happens through the sink, not a `$merge`
stage.

```json
{
  "pipeline": [
    {"$match": {"status": "PAID"}},
    {"$group": {
      "_id": "$customerId",
      "orderCount": {"$count": {}},
      "lifetimeValue": {"$sum": "$orderValue"}
    }}
  ]
}
```

## 2. Wire up source and sink connectors

### Using built-in providers (Kafka → HTTP)

```java
SourceConnectorConfig sourceConfig = SourceConnectorConfig.builder("kafka")
        .option("topic", "orders.v1")
        .option("bootstrapServers", System.getenv("KAFKA_BOOTSTRAP"))
        .option("groupId", "orders-ltv")
        .build();

ConnectorConfig sinkConfig = ConnectorConfig.builder("http", ConnectorConfig.Kind.SINK)
        .option("endpoint", System.getenv("ORDERS_HTTP_ENDPOINT"))
        .option("allowEmpty", false)
        .build();
```

`endpoint` can be swapped for `connectionRef` if you preload HTTP connections via
`ConnectorRegistryInitializer`.

### Using manifest/SDK-driven connectors (custom source/sink)

Implement `SourceConnectorProvider`/`SinkConnectorProvider`, register via ServiceLoader, and reference by `type` in YAML/JSON:

```yaml
source:
  type: myCustomSource
  options:
    apiKey: ${API_KEY}
    endpoint: https://api.example.com/events

sink:
  type: myCustomSink
  options:
    queue: payments
```

In code, load the configs and let the registry resolve them to connectors:

```java
SourceConnectorConfig sourceConfig =
    SourceConnectorConfig.builder("myCustomSource")
        .option("apiKey", System.getenv("API_KEY"))
        .option("endpoint", "https://api.example.com/events")
        .build();

ConnectorConfig sinkConfig =
    ConnectorConfig.builder("myCustomSink", ConnectorConfig.Kind.SINK)
        .option("queue", "payments")
        .build();
```

Connector providers in `fluxion-connect` resolve these configs into runtime
`StreamingSource` and `StreamingSink` instances. Swapping connectors only requires changing the config payload.

## 3. Configure the orchestrator

```java
StageMetrics metrics = new StageMetrics();

StreamingRuntimeConfig runtimeConfig =
    StreamingRuntimeConfig.builder()
        .microBatchSize(500)
        .queueCapacity(2_048)
        .sourceQueueCapacity(32)
        .workerThreadPoolSize(8)
        .build();
```

`StageMetrics` captures per-stage timings and counters. The runtime config tunes
batching, threading, and queue capacities.

## 4. Run the pipeline

```java
StreamingPipelineDefinition definition =
    StreamingPipelineDefinition.builder(sourceConfig)
        .stages(JsonStageLoader.load("rules/orders-ltv.json"))
        .sinkConfig(sinkConfig)
        .pipelineId("orders-ltv")
        .runtimeConfig(runtimeConfig)
        .metrics(metrics)
        .stateStore(new InMemoryStateStore())
        .build();

StreamingPipelineHandle handle = new StreamingPipelineOrchestrator().run(definition);
```

The orchestrator handles the execution loop until the source signals completion
or a fatal error policy triggers a shutdown. `handle.metrics()` returns the same
`StageMetrics` instance registered in `MetricsRegistry`.

## 5. Observe and iterate

- Use the built-in stage metrics (`StageMetrics` + `MetricsRegistry`) to track lag,
  throughput, and error rates.
- State stores allow controlled restarts by persisting offsets/cursors.
- Adjust error policies to determine whether a failure retries, skips, or routes
  events to a dead-letter queue.

## Next steps

1. Explore additional sink/source combos in the [SrotaX Connect](../connect/index.md) section.
2. Add enrichment by dropping `$httpCall` or `$sqlQuery` operators (packaged in
   `fluxion-connect`) into your stages.
3. Harden the pipeline with the operational guides on resilience, metrics, and
   deployment in production environments.
4. Dive into advanced topics for error policies, observability, and deployment
   in the [Streaming Engine overview](index.md#error-handling-strategies).
5. Review which aggregation stages fit streaming versus batch workloads in the
   [Stage Support Matrix](stage-compatibility.md).
6. Run the streaming module tests with `mvn -pl fluxion-connect -am test` to
   validate connectors and executors before deploying.
7. Try the runnable demo in [`fluxion-samples`](https://github.com/aredlavenkat/fluxion-samples/tree/main)
   (`streaming-kafka` for Kafka source + HTTP sink).
