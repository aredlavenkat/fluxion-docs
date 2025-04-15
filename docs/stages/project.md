# $project

Reshapes documents by including, excluding, or computing fields.

---

## ✅ Basic Example

### 📌 Stage

```json
{
  "$project": {
    "product": 1,
    "price": 1,
    "_id": 0
  }
}
```

### 📥 Input

```json
{
  "product": "Laptop",
  "price": 1200,
  "internal": true
}
```

### 📤 Output

```json
{
  "product": "Laptop",
  "price": 1200
}
```

---

## 🧱 Deep Nested Pipeline Usage (Ecommerce)

```json
[
  {
    "$unwind": "$items"
  },
  {
    "$project": {
      "item": "$items.name",
      "cost": "$items.price"
    }
  }
]
```

### 📥 Input Document

```json
{
  "orderId": 1,
  "items": [
    {
      "name": "Phone",
      "price": 500
    },
    {
      "name": "Case",
      "price": 30
    }
  ]
}
```

### 📤 Output Documents

```json
[
  {
    "item": "Phone",
    "cost": 500
  },
  {
    "item": "Case",
    "cost": 30
  }
]
```

---

## ➕ Supported Accumulators

None for this stage

---

## 🔧 Common Operators

$literal, $ifNull, $toUpper

---
