# MongoDB Connectors

Fluxion Connect now ships MongoDB source and sink connectors so pipelines can listen to change streams and persist results back to collections. These connectors target replica-set or sharded deployments that support change streams.

---

## Source (`type: mongodb`)

Watch changes from a collection and feed them into the streaming runtime. Each change event is converted into a Fluxion `Document` with the full document contents (when available) plus change metadata under `_meta`.

```yaml
source:
  type: mongodb
  options:
    connectionString: "mongodb://mongo0:27017,mongo1:27017/?replicaSet=rs0"
    database: orders
    collection: events
    queueCapacity: 64            # internal queue size between change stream and pipeline
    maxBatchSize: 128            # number of events delivered per batch
    maxAwaitTime: PT5S           # how long to wait before yielding an empty batch
    fullDocument: update_lookup  # fetch full document for updates
    pipeline:
      - { $match: { operationType: { $in: ["insert", "replace", "update"] } } }
```

| Option | Description | Default |
| --- | --- | --- |
| `connectionString` | MongoDB connection string with replica-set/sharded topology. | **Required** |
| `database` | Database to monitor. | **Required** |
| `collection` | Collection to monitor. | **Required** |
| `pipeline` | Optional aggregation pipeline applied to the change stream. | `[]` |
| `queueCapacity` | Internal queue capacity feeding the pipeline. | `64` |
| `maxBatchSize` | Maximum change events per batch. | `128` |
| `maxAwaitTime` | Maximum wait before yielding when no events arrive. | `PT5S` |
| `fullDocument` | `default`, `update_lookup`, `when_available`, or `required`. | `default` |

---

## Sink (`type: mongodb`)

Use the sink to persist processed documents back into MongoDB. It supports insert, replace, and upsert semantics depending on your workload.

```yaml
sink:
  type: mongodb
  options:
    connectionString: "mongodb://mongo0:27017,mongo1:27017/?replicaSet=rs0"
    database: analytics
    collection: enriched_orders
    mode: upsert                 # insert | replace | upsert
    keyField: _id                # document field used for replace/upsert filters
    writeConcern: MAJORITY       # optional write concern override
```

| Option | Description | Default |
| --- | --- | --- |
| `connectionString` | MongoDB connection string. | **Required** |
| `database` | Database to write to. | **Required** |
| `collection` | Collection to write to. | **Required** |
| `mode` | Insert, replace, or upsert behaviour. | `insert` |
| `keyField` | Field used to identify documents for replace/upsert. | `_id` |
| `writeConcern` | Write concern to apply (ACKNOWLEDGED, MAJORITY, etc.). | `ACKNOWLEDGED` |

---

## Operational Notes

- Change streams require a replica set or sharded cluster; standalone servers are not supported.
- For large bursts, tune `queueCapacity`, `maxBatchSize`, and MongoDB’s `maxAwaitTime` to balance latency and throughput.
- Combine the sink with `StreamingErrorPolicy` to control retries or dead-lettering when MongoDB becomes unavailable.
- Integration tests require access to a real MongoDB instance; configuration validation tests run without a cluster.
