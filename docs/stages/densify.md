# $densify

`$densify` generates synthetic documents to close gaps in numeric or time-based series. Use it after sorting (and optionally partitioning) to ensure downstream stages operate on regular intervals.

---

## 📌 Syntax

```json
{
  "$densify": {
    "field": "<path>",
    "partitionByFields": [ "<field>", ... ],   // optional
    "range": {
      "step": <number>,
      "unit": "day" | "hour" | ...,           // optional (dates)
      "bounds": "full" | [ <lower>, <upper> ]
    }
  }
}
```

- `field`: Numeric or temporal field to densify. For date/time, supply a `unit` (`millisecond`, `second`, `minute`, `hour`, or `day`).
- `partitionByFields`: Partition keys evaluated per document; densification runs independently per partition.
- `bounds`: `"full"` uses the first and last values; arrays extend the range explicitly.

---

## 🛒 Example – Fill Missing Daily Observations

```json
{
  "$densify": {
    "field": "eventDate",
    "partitionByFields": ["sku"],
    "range": {
      "step": 1,
      "unit": "day",
      "bounds": "full"
    }
  }
}
```

If only 2023-01-01 and 2023-01-03 exist for a SKU, `$densify` inserts a synthetic document for 2023-01-02 with the partition fields present and other fields unset.

---

## 💡 Tips

- Run `$sort` beforehand so documents arrive ordered by partition and densify field.
- Combine with `$fill` to backfill values (linear interpolation, LOCF, or constants) after generating gap rows.
- Bounds array values accept literals, numeric epochs, or ISO date strings.

---

## 🔗 Related Stages

- [`$fill`](./fill.md)
- [`$setWindowFields`](./setWindowFields.md)
- [`$bucket`](./bucket.md)

---
