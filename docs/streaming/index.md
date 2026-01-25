# Streaming Engine Overview

The streaming runtime turns SrotaX’s aggregation engine into an always-on
orchestrator. It reads from connectors, executes declarative pipelines, applies
error policies, stores checkpoints, and exposes metrics so teams can operate
deterministic data flows.

---

## 1. Prerequisites

| Requirement | Notes |
| --- | --- |
| SrotaX modules | `fluxion-core`, `fluxion-connect`, `fluxion-enrich` (optional), plus your pipeline definitions. |
| Runtime host | JVM service/worker that runs streaming executors. |
| Checkpoint store | JDBC/Redis/custom store for offsets and state. |
| Observability | `StreamingMetricsListener` or Micrometer binding for metrics. |
| Scaling plan | Strategy for partitions/shards to avoid contention. |

---

## 2. Architecture components

| Component | Purpose |
| --- | --- |
| Streaming pipeline definition | Binds a `StreamingSource`, a list of SrotaX stages, and a `StreamingSink`. |
| Streaming pipeline orchestrator | Drives fetch → transform → deliver cycles, enforces batching, checkpoints, retries, and invokes metrics hooks. |
| Streaming context | Carries per-run metadata (cursor positions, retry counters, shared attributes). |
| Error policies | Describe how the orchestrator reacts to failures (retry, skip, dead-letter, fail fast). |

### Module relationships

| Module | Role in streaming |
| --- | --- |
| SrotaX Core | Executes stages deterministically. |
| SrotaX Connect | Supplies ingress/egress connectors. |
| SrotaX Enrich | Adds network-aware operators (`$httpCall`, `$sqlQuery`, …). |

---

## 3. Pipeline lifecycle

1. **Fetch** – Read a batch from the configured `StreamingSource`.
2. **Transform** – Run SrotaX stages (and Enrich operators) on the batch.
3. **Deliver** – Push results to the `StreamingSink`.
4. **Checkpoint** – Persist offsets/state so restarts resume correctly.
5. **Observe** – Emit metrics via `StreamingMetricsListener` for dashboards/alerts.

Each step is pluggable; you can swap sources, sinks, error policies, and
listeners without altering pipeline definitions.

---

## 4. Error-handling strategies

`StreamingErrorPolicy` dictates how an exception is handled. Common choices:

| Policy | Behaviour | Use when |
| --- | --- | --- |
| `failFast()` | Abort immediately. | CI/tests or fail-open systems. |
| `retry(int maxAttempts)` | Exponential backoff until limit. | Transient network/service instability. |
| `skipAndContinue()` | Drop the batch and keep going. | Non-critical enrichment where gaps are acceptable. |
| `deadLetter(StreamingSink)` | Send the batch to a DLQ and continue. | Audit-heavy workloads needing post-mortems. |
| Builder pattern | Compose custom retries, circuit breakers, alerts. | Complex pipelines with bespoke recovery. |

Fine-grained handling can be embedded in individual stages (e.g., `$function`
with custom try/catch logic).

---

## 5. Observability

Typical metrics captured via `StreamingMetricsListener` or Micrometer:

- `stream.batch.duration` – processing time per batch.
- `stream.batch.size` – documents in/out per cycle.
- `stream.lag` – source lag (Kafka offsets, cursor age, etc.).
- `stream.retries` / `stream.failures` – counts when error policies trigger.
- `stream.backpressure` – queue depth or wait time between fetches.

Tag metrics with pipeline name, connector ID, environment, tenant, etc. Route
critical alerts (lag spikes, sustained retries) to on-call channels.

---

## 6. Deployment checklist

| Item | Why it matters |
| --- | --- |
| Batch sizing | Stay within connector quotas (Kafka often 250–1000 records). |
| Scaling strategy | Partition-aware scaling prevents checkpoint contention. |
| Secrets/config | Load connector credentials from a secret manager; avoid hard-coded values. |
| Rolling upgrades | Warm standby instances, verify checkpoint compatibility before deploying. |
| Canary/validation | Replay or synthetic runs to confirm deterministic outputs. |
| Failure drills | Regularly test retries, DLQ routing, and checkpoint recovery. |

---

## 7. When to choose streaming

Use the Streaming Engine when you need:

- Continuous ingestion/fan-out (Kafka, HTTP, Event Hubs, custom sources).
- Deterministic replay with durable checkpoints for compliance and audit.
- Real-time enrichment, windowing, or anomaly detection.
- Built-in metrics for throughput, lag, and retries.

**Sample projects** – runnable demos live in the [`fluxion-samples`](https://github.com/aredlavenkat/fluxion-samples/tree/main) repository:

- Kafka topic-to-topic pipeline: `streaming-kafka`
- MongoDB change-stream pipeline: `streaming-mongo`

Prefer the Rule Engine for single-document evaluation, request/response
services, or approval workflows orchestrated via Temporal.

---

## 8. Stage selection

Not every aggregation stage is stream-friendly. Refer to the
[Stage Support Matrix](stage-compatibility.md) for accumulator guidance and
stream-vs-batch recommendations.

---

## 9. Reference files

| Path | Description |
| --- | --- |
| `fluxion-core/src/main/java/.../StreamingPipelineExecutor.java` | Core execution loop (fetch/transform/deliver/checkpoint). |
| `fluxion-core/src/main/java/.../StreamingPipelineOrchestrator.java` | Builder/orchestrator API for configuring pipelines. |
| `fluxion-core/src/main/java/.../StreamingRuntimeConfig.java` | Runtime options (batch size, listeners, error policy). |
| `fluxion-core/src/main/java/.../StreamingErrorPolicy.java` | Error-handling strategies. |
| `http://docs.srotax.com/streaming/quickstart/` | Hands-on tutorial building a Kafka → HTTP pipeline. |

Use these resources when implementing or reviewing streaming integrations.
