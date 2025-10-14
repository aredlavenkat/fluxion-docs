# Kafka Connectors

Fluxion ships Kafka source and sink connectors that share the same SPI. Both accept serializer/security overrides and pass additional Kafka properties straight through to the underlying client.

## Source (`type: kafka`)

```yaml
source:
  type: kafka
  options:
    topic: orders
    bootstrapServers: localhost:9092
    pollTimeout: PT0.5S
    queueCapacity: 64
    keyDeserializer: org.apache.kafka.common.serialization.StringDeserializer
    valueDeserializer: org.apache.kafka.common.serialization.StringDeserializer
    securityProtocol: SASL_SSL
    saslMechanism: PLAIN
    saslJaasConfig: >
      org.apache.kafka.common.security.plain.PlainLoginModule required
      username="user" password="pass";
```

| Option | Description | Default |
| --- | --- | --- |
| `topic` | Kafka topic to subscribe to. | **Required** |
| `bootstrapServers` | Comma-separated broker list. | **Required** |
| `pollTimeout` | ISO-8601 duration for empty poll backoff. | `PT0.5S` |
| `queueCapacity` | Internal queue size between consumer thread and pipeline. | `16` |
| `keyDeserializer` | Kafka key deserializer class name. | `org.apache.kafka.common.serialization.StringDeserializer` |
| `valueDeserializer` | Kafka value deserializer class name. | `org.apache.kafka.common.serialization.StringDeserializer` |
| `securityProtocol` | Optional `security.protocol` override. | `null` |
| `saslMechanism` | Optional `sasl.mechanism` (e.g. `SCRAM-SHA-512`). | `null` |
| `saslJaasConfig` | JAAS config string for SASL. | `null` |

Any additional options provided are forwarded to the consumer `Properties` untouched.

## Sink (`type: kafka`)

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
| `flushTimeout` | ISO-8601 duration to await acknowledgements. | `PT10S` |
| `acks` | Producer acknowledgement level (`acks`). | `1` |
| `keySerializer` | Kafka key serializer class. | `org.apache.kafka.common.serialization.StringSerializer` |
| `valueSerializer` | Kafka value serializer class. | `org.apache.kafka.common.serialization.StringSerializer` |
| `securityProtocol` | Optional `security.protocol`. | `null` |
| `saslMechanism` | Optional `sasl.mechanism`. | `null` |
| `saslJaasConfig` | JAAS config string for SASL. | `null` |

The sink batches JSON-encoded documents, tracks publish/failure metrics, and flushes the producer after each batch. As with the source, any extra option key/value pairs are copied directly into the producer `Properties`.
