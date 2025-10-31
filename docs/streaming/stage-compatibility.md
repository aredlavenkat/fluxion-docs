# Stage Support Matrix

Fluxion exposes the full MongoDB-style aggregation vocabulary, but the runtime
you choose determines how practical a stage is. The Streaming Engine must emit
results incrementally, whereas a future batch job engine can materialise whole
result sets before emitting. This guide calls out how each stage behaves today.

> **Legend**  
> ✅ – Well suited to streaming pipelines  
> ⚠️ – Supported with caveats (state, ordering, performance)  
> ⏳ – More appropriate for batch-style execution; avoid in streaming unless the
>     pipeline structure is tightly controlled.

## Stateless/document-scoped stages

| Stage            | Streaming | Batch | Notes |
| ---------------- | --------- | ----- | ----- |
| `$match`         | ✅        | ✅    | Filters each document; ideal for ingress throttling. |
| `$project` / `$set` / `$addFields` | ✅ | ✅ | Reshape or add computed fields without global state. |
| `$unset` / `$replaceRoot` / `$replaceWith` | ✅ | ✅ | Useful for tidying payloads before sinks receive them. |
| `$limit` / `$skip` | ⚠️ | ✅ | Limit/skip only make sense in streaming when used *after* `$unwind` to constrain per-document arrays. For global limits, prefer batch mode. |
| `$unwind`        | ✅        | ✅    | Enables array fan-out. Unlocks later stages (e.g., `$group`) because repeated keys now represent atomic events. |
| `$sampleRate`    | ✅        | ✅    | Streaming-safe sampling for observability or rate limiting. |

## Stateful aggregations

| Stage            | Streaming | Batch | Notes |
| ---------------- | --------- | ----- | ----- |
| `$group`         | ⚠️        | ✅    | Supported in streaming when grouping by a stable key (e.g., customerId) and using incremental accumulators (`$sum`, `$avg`, `$min`, `$max`, `$count`, `$push`, `$addToSet`). Avoid grouping by expressions that require whole-stream ordering or accumulators that need global knowledge (`$first`, `$last`, `$stdDevSamp`, `$stdDevPop`). See the [$group reference](../stages/group.md) for the full accumulator catalogue. |
| `$setWindowFields` | ⚠️     | ✅    | Works with streaming windows (tumbling, hopping) when a bounded window is declared. Window functions such as `$rank`, `$shift`, `$derivative`, and accumulator-backed windows (`$sum`, `$avg`, etc.) are safe in streaming if the window bounds are finite and the partition key is stable. Unbounded windows devolve into global state—reserve them for batch use. See the [$setWindowFields guide](../stages/setWindowFields.md) for function specifics. |
| `$bucket` / `$bucketAuto` | ⏳ | ✅ | Require knowledge of global min/max or distribution. Reserve for batch or bounded replays. |
| `$densify`       | ⏳        | ✅    | Needs complete interval knowledge to backfill gaps; better suited to batch jobs. |
| `$facet`         | ⚠️        | ✅    | Streaming facets run the same pipeline on the same event, so ensure each sub-pipeline is itself streaming-friendly. Fan-out to sinks if results diverge wildly. |
| `$sort`          | ⏳        | ✅    | Global sort is incompatible with infinite streams. For streaming, sort within a window (e.g., by using `$setWindowFields` + `$push` + client-side ordering) or rely on upstream ordering. |

## Expression-focused stages

| Stage            | Streaming | Batch | Notes |
| ---------------- | --------- | ----- | ----- |
| `$fill`          | ⚠️        | ✅    | Works in streaming when backing values can be sourced from window state or defaults. Window-aware fills that need previous/next documents should run inside bounded windows. |
| `$function`      | ✅        | ✅    | Ideal for bespoke logic. In streaming, ensure functions are side-effect safe and idempotent to support retries. |
| `$densify` / `$sampleRate` | see above | see above | Already covered but included here for quick scanning. |

## Output-oriented stages

| Stage            | Streaming | Batch | Notes |
| ---------------- | --------- | ----- | ----- |
| `$merge` / `$out` | ⏳       | ✅    | Not part of the current Fluxion stage set. For streaming, write to sinks through Fluxion Connect instead. Once batch jobs are available, `$out`-style materialisation becomes more attractive. |
| `$lookup`        | ⚠️        | ✅    | Supported via Enrich or native stages. Streaming lookups must guard against high latency; pair with caching or asynchronous enrichment. |
| `$graphLookup`   | ⏳        | ✅    | Depth-first traversal is expensive for streaming; reserve for batch workloads. |

## Guidance for structuring pipelines

1. **Normalise arrays early.** Apply `$unwind` as soon as possible so downstream
   stages operate on single logical events. This is especially important in
   streaming pipelines that need `$group` or `$setWindowFields`.
2. **Isolate batch-heavy logic.** If a pipeline requires `$sort`, `$densify`,
   or `$bucketAuto`, split it: stream the critical detection piece, and hand off
   the expensive report-building stages to a scheduled batch job.
3. **Use sinks for persistence.** Until a batch job engine ships with `$out`
   semantics, the recommended way to materialise results is via Fluxion Connect
   sinks (HTTP, SQL, Kafka, MongoDB, …).
4. **Budget state carefully.** Stateful stages keep per-key accumulators in
   memory. Pick grouping keys with bounded cardinality and monitor metrics such
   as `stream.state.size`.

## Looking ahead: batch job engine

The forthcoming batch job engine will reuse the same pipeline definitions but
execute over bounded document sets. Expect the following stages to shine in that
environment:

- `$bucket` / `$bucketAuto` for histogramming over complete datasets.
- `$densify` and `$fill` for time series repair.
- `$sort` + `$group` for report-ready ordering.
- `$graphLookup` or expensive `$lookup` joins that need full data stores.

Document your intent using pipeline metadata today so it is clear which stages
assume bounded versus unbounded inputs. That will make migrating to batch
execution straightforward once the engine lands.

## Accumulator cheat sheet for streaming

The following accumulators are generally safe in long-lived pipelines because
they can be updated incrementally per key:

- `$sum`, `$avg`, `$min`, `$max`, `$count`
- `$push`, `$addToSet` (monitor cardinality to avoid unbounded arrays)
- `$first` / `$last` **only** when scoped to a finite window (e.g., inside `$setWindowFields` with bounded bounds)

Prefer to defer these accumulators to batch execution where the full dataset is
available:

- `$stdDevPop`, `$stdDevSamp`, `$covariancePop`, `$covarianceSamp`
- `$mergeObjects` when it expects to see *all* documents to build a final shape
- `$bottomN` / `$topN` style accumulators (coming soon) that inherently need global ordering

For a comprehensive list of accumulators and their semantics, see the
[operators reference](../operators/index.md). Use this matrix to decide when a
feature is practical in streaming—then dive into the stage documentation for
syntax details and examples.
