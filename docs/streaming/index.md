# Streaming Engine Overview

The Fluxion Streaming Engine turns the core aggregation runtime into a managed,
always-on pipeline orchestrator. It continuously ingests events from Connect
sources, runs them through Fluxion Core stages (with optional Enrich operators),
and delivers results to sinks with deterministic replay guarantees.

## Key concepts

- **Streaming pipelines** – Declarative definitions that bind a source, one or
  more Fluxion Core stages, and a sink. Pipelines can run forever or until a
  completion signal arrives.
- **Streaming pipeline orchestrator** – Coordinates execution loops, manages
  batching, retries, and checkpointing, and exposes metrics hooks.
- **Streaming context** – Holds runtime state such as cursor positions, failure
  counts, or custom metadata that operators can consult.
- **Error policies** – Determine how the orchestrator behaves when a stage,
  source, or sink fails (retry, skip, dead-letter, or fail-fast).

## How it relates to other modules

| Module        | Role inside Streaming Engine                                                                    |
| ------------- | ------------------------------------------------------------------------------------------------ |
| Fluxion Core  | Executes each pipeline stage deterministically, ensuring results match batch/offline runs.      |
| Fluxion Enrich| Adds optional network-aware operators invoked during streaming (HTTP lookups, SQL queries, …).  |
| Fluxion Connect| Supplies the streaming sources and sinks; their SPI implementations plug directly into the orchestrator. |

## When to use the Streaming Engine

Choose the Streaming Engine when you need:

- Continuous ingestion or fan-out from Kafka, HTTP, Event Hubs, or custom sources.
- Deterministic replay for governance or audit requirements.
- Built-in observability for throughput, lag, and error rates.
- Tight loops that can react to enrichment lookups or rule decisions in real time.

If you only need ad-hoc or scheduled rule evaluation, the Rule Engine is the
lighter-weight runtime. Many teams adopt both: Streaming Engine for event
orchestration, Rule Engine for approval and governance workflows.

## Execution flow

Every streaming pipeline follows the same loop:

1. **Fetch** – The orchestrator pulls a batch from the configured `StreamingSource`.
2. **Transform** – Fluxion Core executes the declared stages. Enrich operators can
   call out to external systems when required.
3. **Deliver** – The transformed batch is handed to the `StreamingSink`.
4. **Checkpoint** – Commit offsets/cursors so the next batch resumes from the
   correct position.
5. **Metrics & hooks** – The orchestrator invokes optional listeners so you can
   emit telemetry or update dashboards.

Each step is pluggable, which is why the orchestrator leans so heavily on Core,
Connect, and Enrich.

## Error handling strategies

`StreamingErrorPolicy` controls how the orchestrator responds when a source,
stage, or sink throws an exception. The most common options are:

| Policy                           | Behaviour                                                                              | Recommended use case                                    |
| -------------------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| `failFast()`                     | Abort the pipeline immediately.                                                        | CI/test environments, fail-open systems.                |
| `retry(int maxAttempts)`        | Retry the failing batch with exponential backoff until the limit is reached.          | Transient network glitches or short-lived outages.      |
| `skipAndContinue()`             | Drop the failing batch and move on.                                                    | Non-critical enrichment where gaps are acceptable.      |
| `deadLetter(StreamingSink)`     | Forward the failing batch to a secondary sink (e.g., Kafka DLQ) and continue.         | Mission-critical pipelines with audit requirements.     |
| Custom policy via builder       | Combine retries, circuit-breaker windows, or custom callbacks.                        | Complex workloads that blend alerting and back-pressure.|

You can swap policies at runtime by updating the `StreamingRuntimeConfig` and
restarting the pipeline. For per-stage handling, wrap the stage in `$function`
and implement localized recovery logic in the embedded script or service call.

## Observability & metrics

Implement `StreamingMetricsListener` (or use the Micrometer/Dropwizard adapters)
to instrument pipelines. Typical counters and gauges include:

- `stream.batch.duration` – time spent processing a batch.
- `stream.batch.size` – input vs output document counts.
- `stream.lag` – source-specific lag (Kafka offsets, HTTP cursor age, etc.).
- `stream.retries` / `stream.failures` – surfaced when error policies trigger.
- `stream.backpressure` – queue depth or wait time between fetches.

Expose metrics with dimensional tags such as pipeline name, connector ID, and
environment so that dashboards can aggregate across tenants. Many teams also
wire alerting directly off the listener, e.g., push to PagerDuty or Slack when
lag breaches a threshold.

## Deployment considerations

- **Capacity planning** – Size batch counts and concurrency to stay within
  connector quotas. Kafka sources often run best with batch sizes between 250
  and 1,000 records.
- **Horizontal scaling** – Parallelize by partition (Kafka) or sharding key.
  Keep checkpoint stores isolated per instance to avoid cursor contention.
- **Configuration & secrets** – Store connector credentials in your secret
  manager. Inject them at runtime rather than hard-coding inside the pipeline
  definition.
- **Upgrade strategy** – For long-running pipelines, use rolling restarts so a
  standby instance warms up before the active instance drains. Validate that
  checkpoint stores are backward compatible before upgrading major versions.
- **Verification** – Pair production pipelines with synthetic canaries that run
  a subset of data. Compare derived metrics (totals, counts) against batch jobs
  to confirm determinism.
- **Failure drills** – Regularly exercise retries, dead-letter routing, and
  resuming from checkpoints to ensure operational runbooks remain accurate.
