# $lookup

The `$lookup` stage performs a left outer join to another collection in the same database to enrich documents with matching data.

---

## 📌 Syntax (Basic Form)

```json
{
  "$lookup": {
    "from": "<foreign_collection>",
    "localField": "<field_in_local_doc>",
    "foreignField": "<field_in_foreign_doc>",
    "as": "<output_field>"
  }
}
```

---

## ✅ Base Example – Enrich Orders with Product Details

### 📥 Input Document (Order)

```json
{ "orderId": 1, "productId": 101 }
```

### 📚 Foreign Collection (`products`)

```json
[
  { "_id": 101, "name": "Mouse", "price": 50 },
  { "_id": 102, "name": "Keyboard", "price": 100 }
]
```

### 📌 Stage

```json
{
  "$lookup": {
    "from": "products",
    "localField": "productId",
    "foreignField": "_id",
    "as": "productDetails"
  }
}
```

### 📤 Output Document

```json
{
  "orderId": 1,
  "productId": 101,
  "productDetails": [
    { "_id": 101, "name": "Mouse", "price": 50 }
  ]
}
```

---

## 🧱 Deep Nested Pipeline Example – Filter + Transform Product Match

```json
{
  "$lookup": {
    "from": "products",
    "let": { "pid": "$productId" },
    "pipeline": [
      {
        "$match": {
          "$expr": { "$eq": ["$_id", "$$pid"] }
        }
      },
      {
        "$project": {
          "name": 1,
          "price": 1,
          "isExpensive": { "$gt": ["$price", 100] },
          "_id": 0
        }
      }
    ],
    "as": "productInfo"
  }
}
```

### 📥 Input Document

```json
{ "productId": 102 }
```

### 📚 Foreign Collection (`products`)

```json
[
  { "_id": 102, "name": "Keyboard", "price": 120 },
  { "_id": 103, "name": "Monitor", "price": 200 }
]
```

### 📤 Output

```json
{
  "productId": 102,
  "productInfo": [
    {
      "name": "Keyboard",
      "price": 120,
      "isExpensive": true
    }
  ]
}
```

---

## ➕ Supported Accumulators (Used After `$unwind`)

If joined documents are unwound, you can apply grouping with these:

| Accumulator | Description |
|-------------|-------------|
| `$sum` | Count or total values |
| `$avg` | Mean of numeric values |
| `$min` / `$max` | Min/max across joined fields |
| `$push` | Collect into array |
| `$addToSet` | Unique values only |
| `$first` / `$last` | First/last matched records |

---

## 🔧 Common Operators Used in `$lookup` Pipelines

| Operator | Purpose |
|----------|---------|
| `$eq` | Join condition (via `$expr`) |
| `$project` | Field reshaping |
| `$match` | Filter after join |
| `$let` / `$$var` | Use variables in pipeline |
| `$gt`, `$cond` | Conditional logic in projections |

---

## 🧠 Best Practices

- Use `$unwind` after `$lookup` when expecting a single match
- Use `$project` to clean large foreign documents
- Use `$let` + `$expr` for flexible joins

---

## 🔗 Related

- `$unwind` – deconstruct joined arrays
- `$group` – aggregate joined data
- `$expr`, `$function` – join on expressions

---
