# $set

Adds or modifies fields in documents (alias for $addFields).

---

## Syntax

```json
{ "$set": { "fieldName": <expression>, ... } }
```

`$set` accepts the same payload as `$addFields`; both names map to the same stage handler.

---

## ✅ Basic Example

### 📌 Stage

```json
{
  "$set": {
    "shippingFee": 10
  }
}
```

### 📥 Input

```json
{
  "product": "Laptop",
  "price": 1200
}
```

### 📤 Output

```json
{
  "product": "Laptop",
  "price": 1200,
  "shippingFee": 10
}
```

---

## 🧱 Deep Nested Pipeline Usage (Ecommerce)

```json
[
  {
    "$set": {
      "totalWithTax": {
        "$add": [
          "$total",
          {
            "$multiply": [
              "$total",
              0.13
            ]
          }
        ]
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "total": 1000
}
```

### 📤 Output Documents

```json
{
  "total": 1000,
  "totalWithTax": 1130.0
}
```

---

## ➕ Supported Accumulators

None for this stage

---

## 🔧 Common Operators

$add, $multiply

---
