# $setWindowFields

The `$setWindowFields` stage annotates each input document with one or more windowed expressions. Documents are partitioned and sorted, and the specified window functions are evaluated per row, producing new fields alongside the original payload.

---

## 📌 Syntax

```json
{
  "$setWindowFields": {
    "partitionBy": <expression>,          // optional
    "sortBy": { "<field>": 1 | -1 },
    "output": {
      "<field>": {
        "<windowFunction>": <spec>,
        "window": { "documents": [ <lower>, <upper> ] } // optional bounds
      },
      ...
    }
  }
}
```

> Bounds accept `"unbounded"`, `"current"`, or integer offsets relative to the current document, just like MongoDB. Omit `window` to default to `["unbounded", "current"]`.

---

## 🧠 Key Options

| Option | Description |
|--------|-------------|
| `partitionBy` | Groups documents into independent partitions before applying window functions. |
| `sortBy` | Required ordering within each partition. Accepts a single field with ascending or descending direction. |
| `output` | Map of output field names to window function specs. Each spec may also include a custom `window` bound. |

---

## 🛒 Example – Rolling Metrics Per Customer Tier

```json
{
  "$setWindowFields": {
    "partitionBy": "$customer.tier",
    "sortBy": { "eventTime": 1 },
    "output": {
      "prevOrder": {
        "$shift": { "output": "$order.total", "by": 1, "default": null }
      },
      "rank": { "$rank": {} },
      "velocity": {
        "$derivative": { "input": "$order.total", "unit": "minute" },
        "window": { "documents": [ -1, 0 ] }
      }
    }
  }
}
```

Each emitted document keeps the original fields and adds `prevOrder`, `rank`, and `velocity` computed over the partitioned, ordered stream.

---

## 🧩 Supported Window Functions

| Function | Description |
|----------|-------------|
| `$shift` | Returns a value at a relative row offset, with optional default. |
| `$rank` / `$denseRank` | Positional rank (with or without gaps) within the partition. |
| `$derivative` | Rate of change across the window, scaled by a time unit. |
| `$integral` | Trapezoidal integration of numeric values over elapsed time. |
| `$expMovingAvg` | Exponential moving average; accepts `alpha` or `N`. |
| `$stdDevPop`, `$stdDevSamp` | Population or sample standard deviation of numeric input. |
| `$covariancePop`, `$covarianceSamp` | Covariance of paired expressions across the window. |

Standard accumulators (`$sum`, `$avg`, `$min`, `$max`, etc.) are also available through the shared accumulator registry.

---

## 💡 Tips

- Ensure the `sortBy` field is monotonic (numeric epoch or ISO string). `$derivative` and `$integral` rely on the resolved time delta.
- Combine `partitionBy` with compound expressions to segment by tenant, device, region, etc.
- Bounds of `[ "unbounded", "current" ]` produce cumulative metrics. Use offsets like `[-2, 0]` for sliding windows.
- Custom accumulators registered via the Fluxion registry are automatically available inside `$setWindowFields`.

---

## 🔗 Related Stages

- [`$group`](./group.md)
- [`$bucket`](./bucket.md)
- [`$bucketAuto`](./bucketAuto.md)
- [`$sort`](./sort.md)

---
