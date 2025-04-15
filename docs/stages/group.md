# $group

The `$group` stage aggregates documents by a specified `_id` expression and applies accumulator operations to grouped documents.

---

## 📌 Syntax

```json
{
  "$group": {
    "_id": <expression>,
    "field1": { "<accumulator>": <expression> },
    ...
  }
}
```

---

## 📦 Ecommerce Example – Total Sales and Item Count per Category

```json
{
  "$group": {
    "_id": "$category",
    "totalSales": { "$sum": "$price" },
    "itemCount": { "$sum": 1 }
  }
}
```

---

### Input Documents

```json
[
  { "product": "Laptop", "category": "Electronics", "price": 900 },
  { "product": "Phone", "category": "Electronics", "price": 600 },
  { "product": "Shoes", "category": "Fashion", "price": 75 },
  { "product": "Book", "category": "Books", "price": 20 }
]
```

---

### Output Documents

```json
[
  { "_id": "Electronics", "totalSales": 1500, "itemCount": 2 },
  { "_id": "Fashion", "totalSales": 75, "itemCount": 1 },
  { "_id": "Books", "totalSales": 20, "itemCount": 1 }
]
```

---

## 🔁 Group by Composite Keys

```json
{
  "$group": {
    "_id": { "category": "$category", "brand": "$brand" },
    "count": { "$sum": 1 }
  }
}
```

---

## ➕ Supported Accumulators

| Accumulator | Description |
|-------------|-------------|
| `$sum` | Total or count |
| `$avg` | Average value |
| `$min` / `$max` | Extremes |
| `$push` | Array of values |
| `$addToSet` | Unique values |
| `$first` / `$last` | First/last by input order |

---

## 🎯 Ecommerce Accumulator Examples

```json
{
  "$group": {
    "_id": "$category",
    "avgPrice": { "$avg": "$price" },
    "minPrice": { "$min": "$price" },
    "maxPrice": { "$max": "$price" },
    "allBrands": { "$push": "$brand" },
    "uniqueBrands": { "$addToSet": "$brand" },
    "firstItem": { "$first": "$product" },
    "lastItem": { "$last": "$product" }
  }
}
```

---

## 🔧 Operators in Expressions

Operators may appear in `_id` and value expressions.

| Operator | Example |
|----------|---------|
| `$toUpper` | `{ "$toUpper": "$category" }` |
| `$cond` | `{ "$cond": { "if": ..., "then": ..., "else": ... } }` |
| `$convert` | Convert field to number or string |
| `$multiply` | Pre-process price with quantity |

---

### Example – Group by Uppercase Category and Conditional Total

```json
{
  "$group": {
    "_id": { "$toUpper": "$category" },
    "total": {
      "$sum": {
        "$cond": {
          "if": { "$gte": ["$price", 100] },
          "then": "$price",
          "else": 0
        }
      }
    }
  }
}
```

---

## ✅ Best Practices

- Use `$sum: 1` to count documents
- Use `$cond` or `$map` to pre-process values
- Output field names can differ from input fields

---

## 🔗 Related Stages

- [`$project`](./project.md)
- [`$bucket`](./bucket.md)
- [`$facet`](./facet.md)
- [`$replaceRoot`](./replaceRoot.md)

---
