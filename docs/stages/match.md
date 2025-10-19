# $match

Filters documents by a condition.

---

## Syntax

```json
{ "$match": { <fieldOrOperator>: <valueOrExpression> } }
```

Use the same field and operator syntax as MongoDB queries (`$eq`, `$gte`, logical expressions, nested documents, etc.).

---

## ✅ Basic Example

### 📌 Stage

```json
{
  "$match": {
    "status": "active"
  }
}
```

### 📥 Input

```json
{
  "orderId": 1,
  "status": "active",
  "total": 1200
}
```

### 📤 Output

```json
{
  "orderId": 1,
  "status": "active",
  "total": 1200
}
```

---

## 🧱 Deep Nested Pipeline Usage (Ecommerce)

```json
[
  {
    "$match": {
      "orderDate": {
        "$gte": "2024-01-01"
      }
    }
  },
  {
    "$unwind": "$items"
  },
  {
    "$match": {
      "items.price": {
        "$gt": 100
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "orderId": 2,
  "orderDate": "2025-02-10",
  "items": [
    {
      "name": "Laptop",
      "price": 900
    },
    {
      "name": "Mouse",
      "price": 50
    }
  ]
}
```

### 📤 Output Documents

```json
[
  {
    "orderId": 2,
    "orderDate": "2025-02-10",
    "items": {
      "name": "Laptop",
      "price": 900
    }
  }
]
```

---

## ➕ Supported Accumulators

None for this stage

---

## 🔧 Common Operators

$gte, $lt, $eq, $expr

---
