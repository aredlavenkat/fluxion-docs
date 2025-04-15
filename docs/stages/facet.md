# $facet

The `$facet` stage allows you to run **multiple aggregation pipelines in parallel** on the same input documents. Each pipeline outputs to a separate field in the result.

Ideal for generating **dashboards**, **side-by-side summaries**, or **multi-metric analysis**.

---

## 📌 Syntax

```json
{
  "$facet": {
    "facetName1": [ <pipeline1> ],
    "facetName2": [ <pipeline2> ],
    ...
  }
}
```

Each value must be a valid aggregation pipeline (array of stages).

---

## 📦 Ecommerce Example – Dashboard: Top Categories, Brands, and Price Segments

```json
{
  "$facet": {
    "topCategories": [
      { "$group": { "_id": "$category", "totalSales": { "$sum": "$price" } } },
      { "$sort": { "totalSales": -1 } },
      { "$limit": 3 }
    ],
    "topBrands": [
      { "$group": { "_id": "$brand", "count": { "$sum": 1 } } },
      { "$sort": { "count": -1 } },
      { "$limit": 3 }
    ],
    "priceSegments": [
      {
        "$bucket": {
          "groupBy": "$price",
          "boundaries": [0, 100, 500, 1000],
          "default": "Other",
          "output": { "count": { "$sum": 1 } }
        }
      }
    ]
  }
}
```

---

### Input Documents

```json
[
  { "product": "Laptop", "price": 900, "category": "Electronics", "brand": "BrandA" },
  { "product": "Phone", "price": 600, "category": "Electronics", "brand": "BrandB" },
  { "product": "Shoes", "price": 75, "category": "Fashion", "brand": "BrandA" },
  { "product": "Book", "price": 20, "category": "Books", "brand": "BrandC" }
]
```

---

### Output Document

```json
{
  "topCategories": [
    { "_id": "Electronics", "totalSales": 1500 },
    { "_id": "Fashion", "totalSales": 75 },
    { "_id": "Books", "totalSales": 20 }
  ],
  "topBrands": [
    { "_id": "BrandA", "count": 2 },
    { "_id": "BrandB", "count": 1 },
    { "_id": "BrandC", "count": 1 }
  ],
  "priceSegments": [
    { "_id": 0, "count": 2 },
    { "_id": 500, "count": 1 },
    { "_id": 1000, "count": 1 }
  ]
}
```

---

## ➕ Supported Accumulators

Any accumulator supported by `$group`, `$bucket`, or `$bucketAuto` is usable inside `$facet` pipelines:

| Accumulator | Description |
|-------------|-------------|
| `$sum` | Count or total values |
| `$avg` | Mean price or rating |
| `$min` / `$max` | Price extremes |
| `$push`, `$addToSet` | Collect values |

---

## 🔧 Operators in Expressions

Operators are used inside stages nested in each facet pipeline:

| Operator | Example |
|----------|---------|
| `$multiply` | `{ "$multiply": ["$price", "$qty"] }` |
| `$cond` | `{ "$cond": { "if": ..., "then": ..., "else": ... } }` |
| `$map` | Transform arrays |
| `$reduce` | Total prices |

---

## ✅ Best Practices

- Use `$facet` to **aggregate once and branch** for multiple metrics.
- Combine `$facet` with `$group`, `$bucket`, `$sort`, `$project`, etc.
- Avoid overly deep pipelines in each facet to keep it efficient.

---

## 🔗 Related

- [`$group`](./group.md)
- [`$bucket`](./bucket.md)
- [`$project`](./project.md)
- [`$match`](./match.md)

---
