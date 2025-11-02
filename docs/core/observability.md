# Observability & Metrics

Fluxion Core exposes stage-level metrics and an OpenTelemetry bridge so you can
monitor pipeline behaviour. This guide catalogs the available metrics, how to
configure exports, and how to wire them into observability stacks.

---

## 1. Stage metrics

`StageMetrics` captures per-stage counts and timings:

| Metric | Description |
| --- | --- |
| `invocations` | Number of times a stage executed. |
| `inputCount` | Documents consumed by the stage. |
| `outputCount` | Documents emitted by the stage. |
| `durationNanos` | Total execution duration (nanoseconds). |
| `queueSize` | Latest queue size between stages (streaming pipelines). |
| `maxQueueSize` | Maximum queue size observed. |

Access metrics programmatically via `RuleExecutionContext.stageMetrics()` (rule
engine) or by registering listeners on streaming pipelines.

---

## 2. OpenTelemetry bridge

`StageMetricsOtelBridge` translates `StageMetrics` updates into OpenTelemetry
metrics. When enabled, it publishes the following instruments:

| Instrument | Type | Attributes |
| --- | --- | --- |
| `fluxion_stage_invocations_total` | Counter | `pipeline`, `stage` |
| `fluxion_stage_input_docs_total` | Counter | `pipeline`, `stage` |
| `fluxion_stage_output_docs_total` | Counter | `pipeline`, `stage` |
| `fluxion_stage_duration_ms` | Histogram (ms) | `pipeline`, `stage` |
| `fluxion_stage_queue_size_last` | Gauge | `pipeline`, `stage` (streaming only) |
| `fluxion_stage_queue_size_max` | Gauge | `pipeline`, `stage` (streaming only) |

### Enabling the bridge

The bridge uses `OpenTelemetryManager.meter()` under the hood. Configure your
OpenTelemetry SDK (OTLP, Prometheus, etc.) before running pipelines. Example
(using OTLP exporter):

```java
SdkMeterProvider meterProvider = SdkMeterProvider.builder()
        .registerMetricReader(OtlpGrpcMetricExporter.builder().setEndpoint("http://otel-collector:4317").build())
        .build();
OpenTelemetry openTelemetry = OpenTelemetrySdk.builder().setMeterProvider(meterProvider).build();
OpenTelemetryManager.initialize(openTelemetry);
```

Once initialised, `StageMetricsOtelBridge` automatically reports metrics.

### Environment variables

If you use the auto-configured OpenTelemetry SDK, set standard `OTEL_*`
environment variables (e.g., `OTEL_EXPORTER_OTLP_ENDPOINT`, `OTEL_RESOURCE_ATTRIBUTES`).

---

## 3. Custom listeners & logging

For bespoke observability (logging, proprietary metrics), attach custom
`StageMetrics` instances:

- **Rules:** consume `RuleExecutionResult` and inspect `RuleExecutionContext.stageMetrics()`.
- **Streaming:** supply a custom `StageMetrics` via `StreamingPipelineDefinition.Builder#metrics(...)`
  and read snapshots from `StreamingPipelineHandle.metrics()` or `MetricsRegistry.getInstance()`.

```java
class LoggingStageMetrics extends StageMetrics {
    @Override
    public void record(String stageId, int in, int out, long nanos) {
        super.record(stageId, in, out, nanos);
        log.info("stage={} in={} out={} duration={}µs",
                 stageId, in, out, nanos / 1_000);
    }
}

StageMetrics metrics = new LoggingStageMetrics();
StreamingPipelineDefinition definition = StreamingPipelineDefinition.builder(sourceConfig)
        .metrics(metrics)
        // ...
        .build();
```

---

## 4. Testing metrics

- Unit tests: assert stage metrics manually via `RuleEvaluationResult.stageMetrics()`.
- Integration tests: start a test OTLP collector (or in-memory exporter) and
  verify metric names/values.

Example (using OpenTelemetry SDK in tests):

```java
InMemoryMetricReader reader = InMemoryMetricReader.create();
SdkMeterProvider meterProvider = SdkMeterProvider.builder()
        .registerMetricReader(reader)
        .build();
OpenTelemetryManager.initialize(OpenTelemetrySdk.builder().setMeterProvider(meterProvider).build());

// run pipelines, then inspect reader.collectAllMetrics()
```

Run Fluxion core tests to ensure metrics wiring stays intact:

```bash
mvn -pl fluxion-core test -Dtest=*StageMetrics*
```

---

## 5. References

| Path | Description |
| --- | --- |
| `fluxion-core/src/main/java/ai/fluxion/core/aggregation/stages/StageMetrics.java` | Captures stage-level metrics. |
| `fluxion-core/src/main/java/ai/fluxion/core/engine/metrics/MetricsRegistry.java` | Registry exposing pipeline snapshots. |
| `fluxion-core/src/main/java/ai/fluxion/core/engine/streaming/orchestrator/StreamingPipelineHandle.java` | Accesses streaming metrics at runtime. |
| `fluxion-core/src/main/java/ai/fluxion/core/engine/observability/StageMetricsOtelBridge.java` | OpenTelemetry bridge implementation. |
| `fluxion-core/src/main/java/ai/fluxion/core/engine/telemetry/OpenTelemetryManager.java` | Helper for OTel SDK integration. |
| OpenTelemetry | [Metrics overview](https://opentelemetry.io/docs/specs/otel/metrics/). |

For rule-engine specific metrics, see [`docs/rules/glossary.md`](../rules/glossary.md).
