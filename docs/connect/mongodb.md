# MongoDB Connectors

Fluxion Connect ships MongoDB source and sink connectors so pipelines can watch
change streams and persist results back to collections.

---

## 1. Prerequisites

| Requirement | Notes |
| --- | --- |
| Dependency | `ai.fluxion:fluxion-connect` plus MongoDB driver (`org.mongodb:mongodb-driver-sync`). |
| Cluster type | Replica set or sharded cluster (change streams are not supported on stand‑alone instances). |
| Permissions | User with change-stream privileges (`read`, `readWrite`, `changeStream`). |
| Checkpoint store | JDBC/Redis/custom store for streaming offsets. |

---

## 2. Source configuration (`type: mongodb`)

### YAML snippet

```yaml
source:
  type: mongodb
  options:
    connectionString: "mongodb://mongo0:27017,mongo1:27017/?replicaSet=rs0"
    database: orders
    collection: events
    queueCapacity: 64
    maxBatchSize: 128
    maxAwaitTime: PT5S
    fullDocument: update_lookup
    pipeline:
      - { $match: { operationType: { $in: ["insert", "replace", "update"] } } }
```

### Options

| Option | Description | Default |
| --- | --- | --- |
| `connectionString` | MongoDB connection string pointing to replica set / sharded cluster. | **Required** |
| `database` | Database to monitor. | **Required** |
| `collection` | Collection to monitor. | **Required** |
| `pipeline` | Additional aggregation stages applied to the change stream. | `[]` |
| `queueCapacity` | Internal queue size feeding the pipeline. | `64` |
| `maxBatchSize` | Maximum events per batch. | `128` |
| `maxAwaitTime` | Wait before emitting empty batches (ISO-8601). | `PT5S` |
| `fullDocument` | `default`, `update_lookup`, `when_available`, or `required`. | `default` |

---

## 3. Sink configuration (`type: mongodb`)

### YAML snippet

```yaml
sink:
  type: mongodb
  options:
    connectionString: "mongodb://mongo0:27017,mongo1:27017/?replicaSet=rs0"
    database: analytics
    collection: enriched_orders
    mode: upsert
    keyField: _id
    writeConcern: MAJORITY
```

### Options

| Option | Description | Default |
| --- | --- | --- |
| `connectionString` | MongoDB connection string. | **Required** |
| `database` | Database to write to. | **Required** |
| `collection` | Collection to write to. | **Required** |
| `mode` | `insert`, `replace`, or `upsert`. | `insert` |
| `keyField` | Field used to identify documents for replace/upsert. | `_id` |
| `writeConcern` | Write concern (`ACKNOWLEDGED`, `MAJORITY`, etc.). | `ACKNOWLEDGED` |

---

## 4. Troubleshooting

| Symptom | Possible cause | Remedy |
| --- | --- | --- |
| `IllegalArgumentException: connectionString missing` | Config validation failure. | Supply a replica-set/sharded connection string. |
| `MongoCommandException: Change streams not enabled` | Connecting to standalone server. | Configure a replica set or sharded cluster. |
| Events lagging | Change stream backlog or slow consumer. | Increase `queueCapacity`/`maxBatchSize` and inspect MongoDB performance metrics. |
| Sink duplicate key errors | Upsert/replace without matching `keyField`. | Set `mode` appropriately and ensure documents contain the key field. |

---

## 5. Testing

- Run MongoDB connector tests:
  ```bash
  mvn -pl fluxion-core -am test -Dtest=*Mongo*
  ```
- Integration tests require access to a MongoDB replica set; for local use, spin
  up `docker compose` with `--replSet` enabled or use MongoDB’s test containers.

---

## 6. References

| Path | Description |
| --- | --- |
| `fluxion-core/src/main/java/.../MongoSourceConnectorProvider.java` | Source provider implementation. |
| `fluxion-core/src/main/java/.../MongoSinkConnectorProvider.java` | Sink provider implementation. |
| MongoDB docs | [Change streams](https://www.mongodb.com/docs/manual/changeStreams/), [write concern](https://www.mongodb.com/docs/manual/reference/write-concern/). |

Use these templates to wire MongoDB into streaming pipelines, adjusting
queue/batch settings to balance latency versus throughput.
