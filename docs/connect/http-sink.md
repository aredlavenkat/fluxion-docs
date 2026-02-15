# HTTP Sink Connector

Posts batches of documents to an HTTP endpoint. Discovered via `type=http` and
implemented by `HttpSinkProvider` (module: `fluxion-connect`). You can either
point directly at an endpoint or reuse a named HTTP connection from the
connector registry (`connectionRef`) with shared headers/timeouts/resilience.

## Options

| Option | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `endpoint` | string | No (yes if no `connectionRef`) | — | Target HTTP endpoint to post batches to. |
| `connectionRef` | string | No | — | Name of a registered HTTP connection (registry stores base URL, headers, resilience). |
| `connectionScope` | string | No | `default::default` | Optional scope for resolving `connectionRef` (e.g., `tenant::pipeline`). |
| `retryInstance` | string | No | — | Resilience4j retry instance name (overrides connection defaults). |
| `circuitBreakerInstance` | string | No | — | Resilience4j circuit breaker instance name (overrides connection defaults). |
| `allowEmpty` | boolean | No | `false` | Whether to send empty batches. |
| `fanout` | array | No | — | List of additional sinks to call after this sink completes (synchronous). |

## Manifest usage (streaming sink)

```json
"execution": {
  "type": "streaming",
  "source": { "type": "kafka", "bootstrapServers": "localhost:9092", "topic": "orders", "groupId": "g1" },
  "sink":   { "type": "http",  "connectionRef": "orders-api", "allowEmpty": false }
}
```

Register `orders-api` via `ConnectorRegistryInitializer.loadFromSystemProperties(...)`
or programmatically (see Manifest & SDK guide).

## SDK usage

```java
ConnectorConfig sink = ConnectorConfig.builder("http", ConnectorConfig.Kind.SINK)
    .option("connectionRef", "orders-api")
    .option("allowEmpty", false)
    .build();
StreamingSink httpSink = ConnectorFactory.createSink(sink);
```

### Registry-backed connections (optional)

Load HTTP connections at startup so multiple sinks/operators can reuse them:

```java
ConnectorRegistryInitializer.loadFromSystemProperties(); // reads connections/templates paths
// or registry.registerConnection(scope, "orders-api", Map.of("endpoint", "https://api.example.com/ingest"));
```

`retryInstance`/`circuitBreakerInstance` resolve from the shared Resilience4j
registries (populated by the Spring Boot starter or manually).

Use alongside a streaming source (Kafka/custom) with `StreamingPipelineExecutor`
to deliver batches to your HTTP endpoint.

### Chaining and fanout examples

```json
// Pipeline sink feeding two HTTP sinks
"sink": {
  "type": "pipeline",
  "pipeline": "enrich-orders",
  "next": [
    { "type": "http", "connectionRef": "http-a" },
    { "type": "http", "connectionRef": "http-b" }
  ]
}

// Fanout directly from the HTTP sink (synchronous)
"sink": {
  "type": "http",
  "connectionRef": "http-a",
  "fanout": [
    { "type": "http", "connectionRef": "http-b" },
    { "type": "kafka", "connectionRef": "kafka-out" }
  ]
}
```
