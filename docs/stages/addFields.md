# $addFields

The `$addFields` stage adds new fields to existing documents or modifies existing ones. It is often used to compute derived values during aggregation.

This stage does **not filter or remove documents** — it enriches them.

---

## 📌 Syntax

```json
{ "$addFields": { "fieldName": <expression> } }
```

You can specify multiple fields in a single `$addFields` stage.

---

## 🛒 Ecommerce Example – Add Computed `total` Field per Order

```json
{
  "$addFields": {
    "total": {
      "$sum": {
        "$map": {
          "input": "$items",
          "as": "item",
          "in": {
            "$multiply": ["$$item.price", "$$item.quantity"]
          }
        }
      }
    }
  }
}
```

### Input Document

```json
{
  "orderId": 1,
  "items": [
    { "name": "Laptop", "price": 1200, "quantity": 1 },
    { "name": "Mouse", "price": 50, "quantity": 2 }
  ]
}
```

### Output Document

```json
{
  "orderId": 1,
  "items": [...],
  "total": 1300
}
```

---

## 📦 Add Nested Field – `shipping.isFree`

```json
{
  "$addFields": {
    "shipping.isFree": {
      "$cond": {
        "if": { "$gte": ["$total", 500] },
        "then": true,
        "else": false
      }
    }
  }
}
```

### Input Document

```json
{
  "total": 700,
  "shipping": {}
}
```

### Output Document

```json
{
  "total": 700,
  "shipping": {
    "isFree": true
  }
}
```

---

## 🔁 Modify Existing Fields

```json
{
  "$addFields": {
    "customerName": { "$toUpper": "$customerName" }
  }
}
```

### Input Document

```json
{
  "customerName": "alice"
}
```

### Output Document

```json
{
  "customerName": "ALICE"
}
```

---

## 🧠 Combine with System Variables

```json
{
  "$addFields": {
    "updatedAt": "$$NOW",
    "originalDoc": "$$ROOT"
  }
}
```

### Input Document

```json
{
  "product": "Monitor",
  "price": 250
}
```

### Output Document

```json
{
  "product": "Monitor",
  "price": 250,
  "updatedAt": "2024-04-14T12:00:00Z",
  "originalDoc": {
    "product": "Monitor",
    "price": 250
  }
}
```

---

## ⚠️ Special Support – `$$REMOVE`

In Fluxion, you can use `$$REMOVE` to dynamically remove a field:

```json
{
  "$addFields": {
    "discount": {
      "$cond": {
        "if": { "$eq": ["$isMember", false] },
        "then": "$$REMOVE",
        "else": "$discount"
      }
    }
  }
}
```

### Input Document

```json
{
  "isMember": false,
  "discount": 20
}
```

### Output Document

```json
{
  "isMember": false
}
```

---

## 🔍 Behavior

| Aspect         | Behavior |
|----------------|----------|
| Missing fields | Will be added |
| Existing fields | Will be overwritten |
| Nested paths   | Supported (e.g., `"a.b.c"`) |
| `null` values  | Allowed |
| Removed fields | Supported via `$$REMOVE` |

---

## ✅ Used With

- `$cond`, `$map`, `$filter`, `$project`, `$mergeObjects`

---

## 🔗 Related Stages

- [`$set`](./set.md) – alias of `$addFields`
- [`$project`](./project.md) – for shaping output fields
---
