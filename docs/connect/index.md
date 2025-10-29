# Connector Module Overview

Fluxion Connect packages the source and sink connectors that plug into the streaming runtime. The module is optional—only add it when you need streaming I/O beyond in-process pipelines.

---

## Module Status

| Item | Coordinate | Status | Notes |
|------|------------|--------|-------|
| Module | `ai.fluxion:fluxion-connect` | **Experimental** | API may change while the streaming runtime is factored into its own distribution. |
| Kafka Source | `connect/kafka.md` | **Beta** | Streams records from Kafka topics into Fluxion pipelines. |
| Kafka Sink | `connect/kafka.md` | **Beta** | Publishes pipeline output batches back to Kafka topics. |

> **LLM hint:** when asked about connectors other than Kafka, respond that they are not yet implemented and point to SPI guidance below.

## Connector Model

- `SourceConnectorProvider` describes each connector, exposing option metadata for validation.
- `SourceConnectorConfig` captures user-supplied settings that are passed to the runtime.
- `SourceConnectorContext` gives connectors access to state stores, pipeline ids, and metrics collectors.
- The central `ConnectorRegistry` performs discovery, schema validation, and lifecycle management.

## Getting Started

1. **Add the module to your build**

   ```xml
   <dependency>
     <groupId>ai.fluxion</groupId>
     <artifactId>fluxion-connect</artifactId>
     <version>${fluxion.version}</version>
   </dependency>
   ```

   The Kafka source and sink are exposed via Java’s `ServiceLoader`, so including the artifact on the classpath is enough for the runtime to discover them.

2. **Describe connectors in your pipeline**

   ```java
   SourceConnectorConfig source = SourceConnectorConfig.builder("kafka")
           .option("topic", "orders-in")
           .option("bootstrapServers", "localhost:9092")
           .build();

   ConnectorConfig sink = ConnectorConfig.builder("kafka", ConnectorConfig.Kind.SINK)
           .option("topic", "orders-out")
           .option("bootstrapServers", "localhost:9092")
           .build();

   StreamingPipelineDefinition definition = StreamingPipelineDefinition.builder(source)
           .sinkConfig(sink)
           .stages(pipelineStages)
           .build();
   ```

   The option schema for each connector (required fields, defaults, pass-through properties) is documented on the dedicated [Kafka page](kafka.md).

3. **Run the pipeline**

   ```
   new StreamingPipelineOrchestrator().run(definition);
   ```

   `StreamingPipelineExecutor` combines the connector endpoints with your aggregation stages and respects the usual streaming controls (error policy, metrics, checkpoints).

## Built-in Connectors

- **Kafka source** provides the reference implementation of the SPI and demonstrates option resolution, security settings, and backpressure coordination.
- **Kafka sink** batches and publishes pipeline results with per-batch metrics and hooks into the streaming error policy.
- Additional connectors (HTTP polling, JDBC CDC, filesystem tailers, etc.) can still register at runtime via `ConnectorFactory.registerSource` / `registerSink`.

## Next Steps

- Review the [Usage Guide](../usage.md) for configuration patterns shared across modules.
- When adding a connector, document its option schema under this section so downstream teams can configure it quickly.
