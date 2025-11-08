# $subPipeline

Invoke one or more child pipelines (registered or inline) from within the current pipeline. Use it to encapsulate reusable logic or run several enrichment flows and merge their results.

## Syntax

```json
{
  "$subPipeline": [
    "child-name@version",
    { "pipeline": [ { "$set": { "processed": true } } ] },
    {
      "parallel": [ "enrich-geo", { "ref": "enrich-fraud", "globals": { "tenant": "$tenantId" } } ],
      "merge": "facet"
    }
  ]
}
```

- The stage value is an array of steps executed in order.
- A string entry references a pipeline registered in `PipelineRegistry` (`name@version` syntax is optional).
- `{ "pipeline": [ ... ] }` embeds stages inline.
- `{ "parallel": [...] }` runs multiple child pipelines concurrently. Use `merge` to control how branch outputs are combined (`concat`, `zip`, or `facet`).
- Each entry can optionally provide `globals`, `input`, or `onError` overrides.

## Examples

### Sequential invocation

```json
{
  "$subPipeline": [
    "cleanse",
    {
      "pipeline": [
        { "$addFields": { "score": { "$multiply": ["$value", 1.1] } } }
      ]
    },
    "post-process"
  ]
}
```

**Input**

```json
[
  { "value": 10, "tenantId": "acme" },
  { "value": 5, "tenantId": "globex" }
]
```

**Output**

```json
[
  { "value": 11, "tenantId": "acme" },
  { "value": 5.5, "tenantId": "globex" }
]
```

### Parallel enrichment with facet merge

```json
{
  "$subPipeline": [
    {
      "parallel": [ "enrich-geo", "enrich-fraud" ],
      "merge": "facet"
    }
  ]
}
```

Result shape:

```json
{
  "branch0": [ { "value": 10, "geo": "US" } ],
  "branch1": [ { "value": 10, "fraudScore": 0.42 } ]
}
```

Use `$project`, `$unwind`, or downstream stages to pick the facet you need or reshape the output.

### Nested mix of sequential + parallel

```json
{
  "$subPipeline": [
    "cleanse@v2",
    {
      "parallel": [
        "enrich-geo",
        {
          "ref": "enrich-fraud",
          "globals": { "$$TENANT": "$tenantId" },
          "onError": "skip"
        }
      ],
      "merge": "concat"
    },
    {
      "pipeline": [ { "$project": { "value": 1, "geo": 1, "fraudScore": 1 } } ]
    }
  ]
}
```

This example cleanses input documents, runs two enrichments in parallel (skipping the fraud branch if it fails), concatenates their outputs, and finally projects the merged fields before returning to the parent pipeline.

### Registering child pipelines

Named references (e.g., `"cleanse@v2"`, `"enrich-geo"`) resolve through `PipelineRegistry`. Register reusable pipelines during bootstrap:

```json
{
  "pipelines": [
    {
      "name": "cleanse",
      "version": "2",
      "stages": [
        { "$project": { "value": "$value" } }
      ]
    },
    {
      "name": "enrich-geo",
      "stages": [
        { "$addFields": { "geo": "US" } }
      ]
    }
  ]
}
```

Load this DSL at startup, register each entry with `PipelineRegistry`, or embed the stages inline via `{ "pipeline": [ ... ] }` if you don’t need reuse.
