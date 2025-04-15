# $bucket

The `$bucket` stage groups documents into buckets defined by specified boundaries. It is commonly used for **range-based grouping** (e.g., price bands, age bands).

---

## 📌 Syntax

```json
{
  "$bucket": {
    "groupBy": <expression>,
    "boundaries": [ <lower1>, <upper1>, <upper2>, ... ],
    "default": "Other",
    "output": {
      <field>: { <accumulator>: <expression> }
    }
  }
}
```

---

## 📦 Ecommerce Example – Group Orders by Total Sales Price

```json
{
  "$bucket": {
    "groupBy": "$total",
    "boundaries": [0, 100, 500, 1000],
    "default": "Other",
    "output": {
      "orderCount": { "$sum": 1 },
      "avgTotal": { "$avg": "$total" },
      "maxTotal": { "$max": "$total" }
    }
  }
}
```

---

### Input Documents

```json
[
  { "orderId": 1, "total": 45 },
  { "orderId": 2, "total": 200 },
  { "orderId": 3, "total": 800 },
  { "orderId": 4, "total": 1200 }
]
```

---

### Output Documents

```json
[
  { "_id": 0, "orderCount": 1, "avgTotal": 45, "maxTotal": 45 },
  { "_id": 100, "orderCount": 1, "avgTotal": 200, "maxTotal": 200 },
  { "_id": 500, "orderCount": 1, "avgTotal": 800, "maxTotal": 800 },
  { "_id": "Other", "orderCount": 1, "avgTotal": 1200, "maxTotal": 1200 }
]
```

---

## ➕ Supported Accumulators in `$bucket`

| Accumulator | Description | Example Use |
|-------------|-------------|-------------|
| `$sum` | Total count/value | Number of orders |
| `$avg` | Average of values | Avg price per bucket |
| `$min` | Minimum value | Lowest total |
| `$max` | Maximum value | Highest total |
| `$push` | Collect values | Order IDs |
| `$addToSet` | Unique entries | Customer segments |

---

## 🔧 Operator Support in `groupBy` Expression

| Operator | Example |
|----------|---------|
| `$subtract` | `{ "$subtract": ["$price", "$discount"] }` |
| `$convert` | `{ "$convert": { "input": "$amount", "to": "int" } }` |
| `$cond` | `{ "$cond": { "if": { "$gt": ["$total", 500] }, "then": "$total", "else": 0 } }` |

---

## 📈 Example with Operator in `groupBy`

```json
{
  "$bucket": {
    "groupBy": {
      "$convert": {
        "input": "$price",
        "to": "int"
      }
    },
    "boundaries": [0, 100, 500],
    "default": "Other",
    "output": {
      "productCount": { "$sum": 1 },
      "totalPrice": { "$sum": "$price" }
    }
  }
}
```

---

## ✅ Best Practices

- Sort your boundaries in ascending order
- `default` handles out-of-bound values
- Works best on numeric fields (e.g., price, age, rating)

---

## 🔗 Related

- [`$bucketAuto`](./bucketAuto.md)
- [`$group`](./group.md)
- [`$facet`](./facet.md)

---
