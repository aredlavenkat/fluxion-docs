# $unwind

The `$unwind` stage deconstructs an array field from the input document to output a document for **each element**.

---

## 📌 Syntax

```json
{ "$unwind": "$<arrayField>" }
```

---

## ✅ Base Example – Unwind Items in Order

### 📥 Input Document

```json
{
  "orderId": 1,
  "items": [
    { "name": "Laptop", "price": 1200 },
    { "name": "Mouse", "price": 50 }
  ]
}
```

### 📌 Stage

```json
{ "$unwind": "$items" }
```

### 📤 Output Documents

```json
[
  {
    "orderId": 1,
    "items": { "name": "Laptop", "price": 1200 }
  },
  {
    "orderId": 1,
    "items": { "name": "Mouse", "price": 50 }
  }
]
```

---

## 🧱 Deep Nested Example – Unwind, Compute, and Group by Category

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$group": {
      "_id": "$items.category",
      "totalSales": { "$sum": "$items.price" },
      "count": { "$sum": 1 }
    }
  },
  { "$sort": { "totalSales": -1 } }
]
```

### 📥 Input Document

```json
{
  "orderId": 5,
  "items": [
    { "name": "Laptop", "price": 1000, "category": "Electronics" },
    { "name": "Shoes", "price": 150, "category": "Fashion" },
    { "name": "Monitor", "price": 500, "category": "Electronics" }
  ]
}
```

### 📤 Output Documents

```json
[
  { "_id": "Electronics", "totalSales": 1500, "count": 2 },
  { "_id": "Fashion", "totalSales": 150, "count": 1 }
]
```

---

## ➕ Accumulators Used with `$unwind` + `$group`

| Accumulator | Use Case |
|-------------|----------|
| `$sum` | Count or compute totals |
| `$avg` | Category-level average price |
| `$min` / `$max` | Lowest/highest item per category |
| `$push` | List all item names per category |
| `$addToSet` | Unique brands per category |

---

## 🔧 Common Operators

| Operator | Use Case |
|----------|----------|
| `$group` | Aggregate after unwinding |
| `$project` | Select item fields |
| `$sort` | Rank results by totals |
| `$multiply` | Compute per-item total |

---

## 🧠 Tips

- Combine with `$group` for powerful analytics
- Preserve empty arrays with:  
  ```json
  { "$unwind": { "path": "$items", "preserveNullAndEmptyArrays": true } }
  ```
- Works well with deeply nested item structures

---

## 🔗 Related

- `$group`, `$bucket`, `$facet` – to summarize arrays
- `$project`, `$replaceRoot` – to restructure

---
