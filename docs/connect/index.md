# Connector Module Overview

SrotaX Connect packages the ingress/egress connectors that feed streaming
pipelines. Use it when you need Kafka, Event Hubs, MongoDB, or custom sources
and sinks beyond in-process pipelines.

---

## 1. Prerequisites

| Requirement | Notes |
| --- | --- |
| SrotaX modules | `fluxion-core`, `fluxion-connect`, `fluxion-enrich` (optional for enrichment operators). |
| Runtime host | JVM service/worker running the streaming orchestrator. |
| Connector credentials | Bootstrap servers, connection strings, auth secrets. |
| Checkpoint store | JDBC/Redis/custom store for offsets (when streaming). |

---

## 2. Module status

| Item | Coordinate/Doc | Status | Notes |
| --- | --- | --- | --- |
| Module | `ai.fluxion:fluxion-connect` | **Experimental** | APIs may shift while streaming runtime stabilises. |
| Kafka Source/Sink | [connect/kafka.md](kafka.md) | **Beta** | Reference implementation of the connector SPI. |
| Event Hubs Source/Sink | [connect/eventhub.md](eventhub.md) | **Alpha** | Azure Event Hubs ingestion/delivery. |
| MongoDB Source/Sink | [connect/mongodb.md](mongodb.md) | **Alpha** | MongoDB change streams + writers. |
| Primer | [connect/primer.md](primer.md) | **Concepts** | End-to-end connector model, runtimes, and authoring modes. |
| Developer Guide | [connect/developer-guide.md](developer-guide.md) | **How-to** | Build and package connectors (manifest + SDK + SPI). |
| Custom Sources | [connect/custom-sources.md](custom-sources.md) | **How-to** | Build bespoke connectors with the SPI. |

> When asked about connectors other than those listed (HTTP, JDBC CDC, etc.),
> respond that they are not yet implemented and point to the custom SPI guide.

---

## 3. Connector architecture

| Component | Purpose |
| --- | --- |
| `SourceConnectorProvider` / `SinkConnectorProvider` | Declarative metadata (id, description, option schema). |
| `SourceConnectorConfig` / `ConnectorConfig` | User-supplied options validated against the schema. |
| `SourceConnectorContext` | Provides checkpoint stores, pipeline id, metrics. |
| `ConnectorRegistry` | Discovers providers via ServiceLoader, validates configs, exposes catalog discovery, and instantiates connectors. |
| `ConnectorStreamDescriptor` | Catalog metadata for streams/sinks (name, namespace, schema, sync modes, cursor fields). |
| `ConnectorFactory` | Helper to create sources/sinks and to discover source/sink catalogs. |

---

## 4. Usage steps

1. **Add the module**

   ```xml
   <dependency>
     <groupId>ai.fluxion</groupId>
     <artifactId>fluxion-connect</artifactId>
     <version>${fluxion.version}</version>
   </dependency>
   ```

   Providers are discovered via Java’s `ServiceLoader`; no explicit registration
   is required if the jar is on the classpath.

2. **Describe source and sink**

   ```java
   SourceConnectorConfig source = SourceConnectorConfig.builder("kafka")
           .option("topic", "orders-in")
           .option("bootstrapServers", "localhost:9092")
           .build();

   ConnectorConfig sink = ConnectorConfig.builder("kafka", ConnectorConfig.Kind.SINK)
           .option("topic", "orders-out")
           .option("bootstrapServers", "localhost:9092")
           .build();
   ```

   Consult each connector page for required/optional options and defaults.

3. **Build the pipeline definition**

   ```java
   StreamingPipelineDefinition definition = StreamingPipelineDefinition.builder(source)
           .sinkConfig(sink)
           .stages(pipelineStages)
           .build();
   ```

4. **Run with the orchestrator**

   ```java
   new StreamingPipelineOrchestrator().run(definition, runtimeConfig);
   ```

5. **(Optional) Discover stream catalogs**

   ```java
   // Source streams (e.g., Kafka topic catalog)
   List<ConnectorStreamDescriptor> sources =
           ConnectorFactory.discoverSourceStreams(source);

   // Sink streams (e.g., Mongo collection schema + PK)
   List<ConnectorStreamDescriptor> sinks =
           ConnectorFactory.destinationStreams(sink);
   ```

   Use these catalogs to display metadata in UIs/CLIs and to drive cursor/PK choices without baking connector details into pipeline definitions.

---

## 5. Configuration table (common options)

| Option | Connectors | Description |
| --- | --- | --- |
| `bootstrapServers` | Kafka | Comma-separated broker list. |
| `topic` | Kafka, Event Hubs | Source/sink topic or event hub. Also appears in stream descriptors. |
| `groupId` | Kafka | Consumer group for checkpointing. |
| `connectionString` | Event Hubs, MongoDB | Service connection string/URI. |
| `checkpointStore` | All streaming connectors | Where offsets are saved (JDBC, Redis, etc.). |
| `queueCapacity` | Kafka, Event Hubs, Mongo source | Internal handoff queue size (source to pipeline). |
| `cursorField` (implicit) | Source-defined for Kafka/EventHub/Mongo | Cursor is connector-owned; pipeline stays cursor-agnostic. |

Refer to the connector-specific docs for security settings (SASL, TLS, Azure SAS,
Mongo credentials) and batching controls.

---

## 6. Built-in connectors

| Connector | Direction | Highlights |
| --- | --- | --- |
| Kafka | Source & Sink | Backpressure-aware batching, SASL/TLS support, per-batch metrics; source-defined cursor (offset). |
| Event Hubs | Source & Sink | Consumer groups, partition lease management; source-defined cursor (sequence number). |
| MongoDB | Source & Sink | Change-stream support, resume tokens; upsert/replace modes; source-defined cursor (resume token). |

Each connector page contains option schema tables, example configurations, and
operational caveats.

---

## 7. Custom connectors

- Implement `SourceConnectorProvider`/`SinkConnectorProvider` and register via
  `META-INF/services` (see [custom-sources.md](custom-sources.md)).
- Provide a clear option schema with validation messages—LLMs and tooling rely on
  those hints to prompt users.
- Reuse shared components: checkpoint store SPI, metrics listener, error policy.
- Document new connectors under this section to keep the matrix up to date.

---

## 8. References

| Path | Description |
| --- | --- |
| `fluxion-core/src/main/java/.../ConnectorRegistry.java` | Central registry for source/sink providers. |
| `fluxion-core/src/main/java/.../SourceConnectorProvider.java` | Provider contract. |
| `fluxion-core/src/main/java/.../SourceConnectorConfig.java` | Config builder/validation. |
| `https://docs.srotax.com/connect/kafka/` | Kafka-specific options and examples. |
| `https://docs.srotax.com/connect/custom-sources/` | SPI guide for bespoke connectors. |

Run connector tests along with streaming tests:

```bash
mvn -pl fluxion-core -am test
```

This ensures connector discovery, option validation, and streaming executors are
validated together.
