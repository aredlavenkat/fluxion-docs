# $bucketAuto

The `$bucketAuto` stage automatically categorizes documents into a specified number of **equi-sized buckets** based on a specified `groupBy` expression.

Unlike `$bucket`, it does not require manually defined boundaries — the system computes them based on data distribution.

---

## 📌 Syntax

```json
{
  "$bucketAuto": {
    "groupBy": <expression>,
    "buckets": <number>,
    "output": {
      <field>: { <accumulator>: <expression> }
    }
  }
}
```

---

## 📦 Ecommerce Example – Auto-Bucket Orders by `total` Field

```json
{
  "$bucketAuto": {
    "groupBy": "$total",
    "buckets": 3,
    "output": {
      "count": { "$sum": 1 },
      "averageTotal": { "$avg": "$total" }
    }
  }
}
```

---

### Input Documents

```json
[
  { "orderId": 1, "total": 45 },
  { "orderId": 2, "total": 120 },
  { "orderId": 3, "total": 300 },
  { "orderId": 4, "total": 550 },
  { "orderId": 5, "total": 800 },
  { "orderId": 6, "total": 1100 }
]
```

---

### Output Documents (Example Buckets)

```json
[
  {
    "_id": { "min": 45, "max": 300 },
    "count": 3,
    "averageTotal": 155
  },
  {
    "_id": { "min": 300, "max": 800 },
    "count": 2,
    "averageTotal": 675
  },
  {
    "_id": { "min": 800, "max": 1100 },
    "count": 1,
    "averageTotal": 1100
  }
]
```

ℹ️ The ranges and averages are automatically determined.

---

## ➕ Supported Accumulators in `$bucketAuto`

| Accumulator | Description | Example Use |
|-------------|-------------|-------------|
| `$sum` | Count documents or totals | Count of orders |
| `$avg` | Average of values | Avg order value |
| `$min` | Minimum | Lowest total |
| `$max` | Maximum | Highest total |
| `$push` | Push array of values | Push productIds |
| `$addToSet` | Unique entries | Unique customer types |

---

## 🔧 Operator Support in `groupBy` Expression

| Operator | Example |
|----------|---------|
| `$subtract` | `{ "$subtract": ["$price", "$discount"] }` |
| `$convert` | `{ "$convert": { "input": "$price", "to": "int" } }` |
| `$cond` | `{ "$cond": { "if": { "$gte": ["$total", 1000] }, "then": "$total", "else": 0 } }` |

---

## 📈 Example with Operator in `groupBy`

```json
{
  "$bucketAuto": {
    "groupBy": {
      "$convert": {
        "input": "$total",
        "to": "int"
      }
    },
    "buckets": 4,
    "output": {
      "orderCount": { "$sum": 1 },
      "maxTotal": { "$max": "$total" }
    }
  }
}
```

---

## ✅ Best Practices

- Use for automatic segmentation when manual ranges aren’t known
- Useful for analytics dashboards and histogram-like insights
- Output buckets may not be evenly sized by count

---

## 🔗 Related

- [`$bucket`](./bucket.md)
- [`$group`](./group.md)
- [`$facet`](./facet.md)

---
