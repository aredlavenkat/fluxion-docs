# Advanced Pipeline Gallery

Real-world snippets showcasing nested facets, accumulator-heavy groups, and
expression operators. Copy these into your tests/services or slice them apart
when building tutorials.

---

## Scenario 1 – Category & Price Buckets

| Goal | Group sales by category while simultaneously bucketing item prices. |
| --- | --- |
| Features | `$match`, `$unwind`, `$facet`, `$group`, `$project`, `$bucket` |
| Where used | `fluxion-core/src/test/.../FacetBucketTest.java` |

```json
[
  { "$match": { "orderDate": { "$gte": "2024-01-01" } } },
  { "$unwind": "$items" },
  { "$facet": {
    "byCategory": [
      { "$group": {
          "_id": "$items.category",
          "totalSales": { "$sum": "$items.price" },
          "averagePrice": { "$avg": "$items.price" },
          "count": { "$sum": 1 }
      }},
      { "$project": {
          "category": "$_id",
          "totalSales": 1,
          "averagePrice": 1,
          "count": 1,
          "_id": 0
      }}
    ],
    "priceBuckets": [
      { "$bucket": {
          "groupBy": "$items.price",
          "boundaries": [0, 100, 500, 1000],
          "default": "Other",
          "output": {
            "itemCount": { "$sum": 1 },
            "avgPrice": { "$avg": "$items.price" }
          }
      }}
    ]
  }}
]
```

**Tips**

- Ensure `orderDate` is indexed upstream if you replay large histories.
- Use `$bucketAuto` when you don’t know price boundaries ahead of time.

---

## Scenario 2 – Accumulator Cheat Sheet

| Goal | Demonstrate common accumulators inside a single `$group`. |
| --- | --- |
| Features | `$sum`, `$avg`, `$min`, `$max`, `$first`, `$last`, `$addToSet`, `$push` |
| Test ref | `fluxion-core/src/test/.../AccumulatorShowcaseTest.java` |

```json
[
  {
    "$group": {
      "_id": "$category",
      "total": { "$sum": "$amount" },
      "avg": { "$avg": "$amount" },
      "min": { "$min": "$amount" },
      "max": { "$max": "$amount" },
      "first": { "$first": "$amount" },
      "last": { "$last": "$amount" },
      "uniqueBrands": { "$addToSet": "$brand" },
      "allItems": { "$push": "$item" }
    }
  }
]
```

**Tips**

- Monitor array sizes for `$push`/`$addToSet` to avoid runaway memory usage.
- For streaming pipelines, prefer incremental accumulators (`$sum`, `$avg`).

---

## Scenario 3 – Expression Operators ($map`, `$filter`, `$reduce`, `$cond`, `$switch`)

| Operator | Purpose | Snippet |
| --- | --- | --- |
| `$map` | Compute totals per item | `{ "$map": { "input": "$items", "as": "item", "in": { "total": { "$multiply": ["$$item.price", "$$item.qty"] } } } }` |
| `$filter` | Keep items with qty > 1 | `{ "$filter": { "input": "$items", "as": "item", "cond": { "$gt": ["$$item.qty", 1] } } }` |
| `$reduce` | Sum prices | `{ "$reduce": { "input": "$items", "initialValue": 0, "in": { "$add": ["$$value", "$$this.price"] } } }` |
| `$cond` | Label price brackets | `{ "$cond": { "if": { "$gte": ["$price", 1000] }, "then": "expensive", "else": "cheap" } }` |
| `$switch` | Multi-tier labeling | `{ "$switch": { "branches": [ { "case": { "$lt": ["$price", 100] }, "then": "low" }, { "case": { "$lt": ["$price", 500] }, "then": "mid" } ], "default": "high" } }` |

**Tips**

- `$map`/`$filter`/`$reduce` operate on arrays; they’re composable for complex
  calculations.
- Combine `$cond` or `$switch` with `$set`/`$addFields` for classification tasks.

---

## Running the gallery locally

Use the standard executor to run any of the pipelines above:

```java
PipelineExecutor executor = new PipelineExecutor();
List<Document> result = executor.run(inputDocuments, stages, Map.of());
```

For regression testing, copy examples into unit tests and run:

```bash
mvn -pl fluxion-core test
```

The tests under `fluxion-core/src/test/java/...` contain ready-made fixtures you
can adapt.
