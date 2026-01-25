# Kafka Connectors

SrotaX Connect exposes Kafka as both a streaming **source** and **sink**. The
connectors share the same SPI used across the platform, so the configuration
maps cleanly onto streaming pipelines, error policies, and metrics.

---

## 1. Prerequisites

| Requirement | Notes |
| --- | --- |
| Dependency | Add `ai.fluxion:fluxion-connect` plus Kafka client (`org.apache.kafka:kafka-clients`). |
| Kafka cluster | Brokers reachable from the SrotaX service. Tested on Kafka 2.8+. |
| Credentials (optional) | SASL/TLS material if connecting to secure clusters. |
| Checkpoint store | JDBC/Redis/custom store for offsets when streaming. |

---

## 2. Source configuration (`type: kafka`)

### YAML snippet

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
    enable.auto.commit: false   # passes through to Kafka consumer properties
```

### Options

| Option | Description | Default |
| --- | --- | --- |
| `topic` | Kafka topic to subscribe to. | **Required** |
| `bootstrapServers` | Comma-separated list of brokers. | **Required** |
| `pollTimeout` | Consumer poll wait (ISO-8601 duration). | `PT0.5S` |
| `queueCapacity` | Internal queue size feeding the pipeline. | `16` |
| `keyDeserializer` | Kafka key deserializer class. | `StringDeserializer` |
| `valueDeserializer` | Kafka value deserializer class. | `StringDeserializer` |
| `groupId` | Consumer group id override. | `fluxion-<topic>` |
| `securityProtocol` | Kafka `security.protocol` (e.g., `SSL`, `SASL_SSL`). | `null` |
| `saslMechanism` | Kafka `sasl.mechanism` (e.g., `SCRAM-SHA-512`). | `null` |
| `saslJaasConfig` | JAAS config string for SASL. | `null` |

All additional keys under `options` are copied into the consumer `Properties`
object, so you can enable idempotent consumers, partition assignment strategies,
or custom timeouts.

---

## 3. Sink configuration (`type: kafka`)

### YAML snippet

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
    compression.type: gzip
```

### Options

| Option | Description | Default |
| --- | --- | --- |
| `topic` | Kafka topic to publish to. | **Required** |
| `bootstrapServers` | Comma-separated broker list. | **Required** |
| `batchSize` | Records per send batch. | `100` |
| `flushTimeout` | Wait for acknowledgements (ISO-8601). | `PT10S` |
| `acks` | Producer acknowledgement level (`acks`). | `1` |
| `keySerializer` | Kafka key serializer class. | `StringSerializer` |
| `valueSerializer` | Kafka value serializer class. | `StringSerializer` |
| `securityProtocol` / `saslMechanism` / `saslJaasConfig` | Same as source. | `null` |

Additional producer options (linger, retries, idempotence, compression) are
passed through untouched.

---

## 4. Stream catalogs (for UIs/CLIs)

Kafka connectors expose catalogs for discovery endpoints:

- **Source** (`discoverStreams`): `name=topic`, `namespace=groupId or "kafka"`, `supportedSyncModes=[FULL_REFRESH, INCREMENTAL]`, `cursorFields=["offset"]`, `sourceDefinedCursor=true` (offsets are connector-owned).
- **Sink** (`destinationStreams`): `name=topic`, `namespace=bootstrapServers`, `supportedSyncModes=[FULL_REFRESH]`.

Fetch these via `/api/connectors/discovery/sources|sinks` so pipeline specs stay connector-agnostic.

---

## 5. Security options

| Scenario | Required settings |
| --- | --- |
| TLS without auth | `securityProtocol: SSL` plus truststore configuration (system properties or Kafka props). |
| SASL/PLAIN | `securityProtocol: SASL_SSL`, `saslMechanism: PLAIN`, `saslJaasConfig: ...`. |
| SCRAM | `securityProtocol: SASL_SSL`, `saslMechanism: SCRAM-SHA-512`, JAAS string with username/password. |
| Kerberos | `securityProtocol: SASL_PLAINTEXT`, `saslMechanism: GSSAPI`, JAAS config referencing keytab/krb5. |

Consult Kafka’s security docs for the exact property names; any additional keys
(e.g., `ssl.truststore.location`) can be added directly to `options`.

---

## 6. Troubleshooting

| Symptom | Possible cause | Remedy |
| --- | --- | --- |
| `NoSuchMethodError` on Kafka classes | Version mismatch between `fluxion-connect` and your Kafka client. | Align Kafka client version with the cluster (2.8+ recommended). |
| Consumer stuck at oldest offset | `groupId` shared across environments. | Set a unique group id per environment or use `auto.offset.reset`. |
| `TopicAuthorizationFailed` | Missing ACLs or credentials. | Verify SASL/TLS settings and broker ACLs. |
| Sink retries endlessly | Downstream brokers unavailable. | Adjust `StreamingErrorPolicy` (dead-letter/skip) or configure Kafka producer retries. |

---

## 7. Testing

- Run connector tests alongside streaming tests:
  ```bash
  mvn -pl fluxion-core -am test -Dtest=*Kafka*
  ```
- Local broker for manual testing: `docker run --rm -p 9092:9092 confluentinc/cp-kafka`.

---

## 8. References

| Path | Description |
| --- | --- |
| `fluxion-core/src/main/java/.../KafkaSourceConnectorProvider.java` | Source provider implementation. |
| `fluxion-core/src/main/java/.../KafkaSinkConnectorProvider.java` | Sink provider implementation. |
| `http://docs.srotax.com/streaming/quickstart/` | Kafka → HTTP streaming tutorial. |

Use these snippets as templates for your own pipeline definitions and adjust
properties according to your cluster’s configuration.
