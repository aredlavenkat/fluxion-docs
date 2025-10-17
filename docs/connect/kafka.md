# Kafka Connectors

Fluxion exposes Kafka as both a streaming **source** and **sink**. The connectors reuse the SPI we use for every transport, so the configuration is intentionally symmetrical: serializers, security settings, batching behaviour, and any additional Kafka property can be injected directly.

---

## Source (`type: kafka`)

Use the source connector to subscribe to a topic and push records into a pipeline. Each record is converted to a Fluxion document (`_meta` carries Kafka metadata such as topic, partition, offset, timestamp, and key).

```yaml
source:
  type: kafka
  options:
    topic: orders
    bootstrapServers: localhost:9092
    pollTimeout: PT0.5S        # optional ISO-8601 duration
    queueCapacity: 64          # internal queue between consumer and stages
    keyDeserializer: org.apache.kafka.common.serialization.StringDeserializer
    valueDeserializer: org.apache.kafka.common.serialization.StringDeserializer
    securityProtocol: SASL_SSL
    saslMechanism: PLAIN
    saslJaasConfig: >
      org.apache.kafka.common.security.plain.PlainLoginModule required
      username="user" password="pass";
    enable.auto.commit: false  # additional Kafka option is passed through
```

| Option | Description | Default |
| --- | --- | --- |
| `topic` | Kafka topic to subscribe to. | **Required** |
| `bootstrapServers` | Comma-separated broker list. | **Required** |
| `pollTimeout` | Wait time when no records are returned. | `PT0.5S` |
| `queueCapacity` | Size of the internal queue feeding the next stage. | `16` |
| `keyDeserializer` | Kafka key deserializer class. | `StringDeserializer` |
| `valueDeserializer` | Kafka value deserializer class. | `StringDeserializer` |
| `securityProtocol` | Optional `security.protocol` override (e.g. `SSL`, `SASL_SSL`). | `null` |
| `saslMechanism` | Optional `sasl.mechanism` (e.g. `SCRAM-SHA-512`). | `null` |
| `saslJaasConfig` | JAAS config string when SASL is enabled. | `null` |
| `groupId` | Overrides consumer group id. | `fluxion-<topic>` |

All other keys under `options` are copied straight into the consumer `Properties`, so you can configure commits, partition assignment, etc.

---

## Sink (`type: kafka`)

The sink batches JSON-encoded documents, tracks success/failure statistics, and flushes once every batch or when the timeout expires. If an exception occurs we bubble it up so the streaming error-policy can react (retry, dead-letter, skip).

```yaml
sink:
  type: kafka
  options:
    topic: orders-out
    bootstrapServers: localhost:9092
    batchSize: 200
    flushTimeout: PT5S
    acks: all
    keySerializer: org.apache.kafka.common.serialization.StringSerializer
    valueSerializer: org.apache.kafka.common.serialization.StringSerializer
    securityProtocol: SASL_SSL
    saslMechanism: PLAIN
    saslJaasConfig: >
      org.apache.kafka.common.security.plain.PlainLoginModule required
      username="user" password="pass";
    compression.type: gzip
```

| Option | Description | Default |
| --- | --- | --- |
| `topic` | Kafka topic to publish to. | **Required** |
| `bootstrapServers` | Comma-separated broker list. | **Required** |
| `batchSize` | Number of documents per send batch. | `100` |
| `flushTimeout` | ISO-8601 duration to wait for all acknowledgements. | `PT10S` |
| `acks` | Producer acknowledgement level (`acks`). | `1` |
| `keySerializer` | Kafka key serializer class. | `StringSerializer` |
| `valueSerializer` | Kafka value serializer class. | `StringSerializer` |
| `securityProtocol` | Optional `security.protocol`. | `null` |
| `saslMechanism` | Optional `sasl.mechanism`. | `null` |
| `saslJaasConfig` | JAAS config string when SASL is enabled. | `null` |

Additional producer properties (idempotence, linger.ms, compression, etc.) are copied straight onto the Kafka producer configuration.

---

## Operational Notes

- **Metrics:** the sink tracks successful and failed records plus batch counts; wire these into your monitoring layer using the streaming metrics APIs.
- **Retry / error policies:** combine the sink with `StreamingErrorPolicy` to control retries, dead-letter routing, or batch skipping if Kafka becomes unavailable.
- **Testing:** the codebase uses Kafka Testcontainers for integration tests. If you need a local broker, run `docker run --rm -p 9092:9092 confluentinc/cp-kafka` (or reuse the compose file in your project).
