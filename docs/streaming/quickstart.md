# Streaming Engine Quick Start

This walkthrough demonstrates how to stand up a simple streaming pipeline that
ingests events from Kafka, applies a Fluxion Core aggregation, and delivers the
results to an HTTP endpoint.

## Prerequisites

- Fluxion Core modules available on the classpath (`fluxion-core`, `fluxion-connect`, `fluxion-enrich` if enrichment is needed).
- Access to a Kafka cluster and topic that will act as the source.
- Java 17+ (the samples use records, builder APIs, and switch expressions).

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

```java
KafkaStreamingSource source =
    KafkaStreamingSource.builder()
        .bootstrapServers(System.getenv("KAFKA_BOOTSTRAP"))
        .topic("orders.v1")
        .groupId("fluxion-ltv")
        .build();

HttpStreamingSink sink =
    HttpStreamingSink.builder()
        .endpoint("https://api.example.com/ltv")
        .method("POST")
        .retryPolicy(RetryPolicy.exponentialBackoff())
        .transformer(documents -> documents.stream()
            .map(document -> Map.of(
                "customerId", document.getString("_id"),
                "orderCount", document.getInteger("orderCount"),
                "lifetimeValue", document.getDouble("lifetimeValue")))
            .toList())
        .build();
```

Both connectors implement `StreamingSource` / `StreamingSink` and can be swapped
for custom implementations as needed.

## 3. Configure the orchestrator

```java
StreamingRuntimeConfig config =
    StreamingRuntimeConfig.builder()
        .pipelineName("orders-ltv")
        .batchSize(500)
        .metricsListener(new MicrometerStreamingMetrics())
        .checkpointStore(new JdbcCheckpointStore(dataSource))
        .errorPolicy(StreamingErrorPolicy.retry(3))
        .build();
```

The configuration captures operational concerns: batching, metrics, checkpoint
storage, and error handling.

## 4. Run the pipeline

```java
StreamingPipelineDefinition definition =
    StreamingPipelineDefinition.builder(source)
        .stages(JsonStageLoader.load("rules/orders-ltv.json"))
        .sink(sink)
        .build();

new StreamingPipelineOrchestrator().run(definition, config);
```

The orchestrator handles the execution loop until the source signals completion
or a fatal error policy triggers a shutdown.

## 5. Observe and iterate

- Use the built-in metrics hooks (`StreamingMetricsListener`) to track lag,
  throughput, and error rates.
- Checkpoint stores allow controlled restarts by persisting offsets/cursors.
- Adjust error policies to determine whether a failure retries, skips, or routes
  events to a dead-letter queue.

## Next steps

1. Explore additional sink/source combos in the [Fluxion Connect](../connect/index.md) section.
2. Add enrichment by dropping `$httpCall` or `$sqlQuery` operators from the
   [Fluxion Enrich](../enrich/index.md) module into your stages.
3. Harden the pipeline with the operational guides on resilience, metrics, and
   deployment in production environments.
4. Dive into advanced topics for error policies, observability, and deployment
   in the [Streaming Engine overview](index.md#error-handling-strategies).
5. Review which aggregation stages fit streaming versus batch workloads in the
   [Stage Support Matrix](stage-compatibility.md).
