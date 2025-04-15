# $sort

The `$sort` stage sorts all input documents and passes them along in sorted order.

You can sort by multiple fields and directions (`1` for ascending, `-1` for descending).

---

## 📌 Syntax

```json
{ "$sort": { "<field1>": 1, "<field2>": -1 } }
```

---

## ✅ Base Example – Sort Products by Price (Descending)

### 📥 Input Documents

```json
[
  { "product": "Book", "price": 10 },
  { "product": "Monitor", "price": 200 },
  { "product": "Mouse", "price": 50 }
]
```

### 📌 Stage

```json
{ "$sort": { "price": -1 } }
```

### 📤 Output

```json
[
  { "product": "Monitor", "price": 200 },
  { "product": "Mouse", "price": 50 },
  { "product": "Book", "price": 10 }
]
```

---

## 🧱 Deep Nested Example – Filter, Compute Total, then Sort

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "item": "$items.name",
      "total": {
        "$multiply": ["$items.price", "$items.quantity"]
      }
    }
  },
  { "$sort": { "total": -1 } }
]
```

### 📥 Input Document

```json
{
  "orderId": 1,
  "items": [
    { "name": "Laptop", "price": 1000, "quantity": 1 },
    { "name": "Mouse", "price": 50, "quantity": 2 }
  ]
}
```

### 📤 Output Documents

```json
[
  { "item": "Laptop", "total": 1000 },
  { "item": "Mouse", "total": 100 }
]
```

---

## ➕ Accumulators Used (Post-Sort Grouping)

You can combine `$sort` with stages like `$group`:

| Accumulator | Use Case |
|-------------|----------|
| `$sum` | Group totals after sorting |
| `$avg` | Find top averages |
| `$max` / `$min` | Determine max/min sorted results |
| `$first` / `$last` | Retain first/last after sort |
| `$push` | Preserve full value sequence |

---

## 🔧 Common Operators in $sort Pipelines

| Operator | Purpose |
|----------|---------|
| `$multiply` | Compute totals before sorting |
| `$project` | Shape fields before sort |
| `$unwind` | Flatten arrays for item-level sorting |
| `$cond`, `$add` | Conditional or derived fields |

---

## 🧠 Tips

- Use index-backed sort for better performance
- Avoid sorting huge datasets unless paginated
- Chain `$limit` after `$sort` for top-N

---

## 🔗 Related

- `$limit`, `$skip` – pagination
- `$group` – post-sort aggregation
- `$project` – prep sort field

---
