# $reverseArray

The `$reverseArray` operator returns a new array with the elements **in reverse order**.

---

## 📌 Syntax

```json
{ "$reverseArray": <arrayExpression> }
```

---

## ✅ Base Example – Reverse Numbers

### 📥 Input Document

```json
{ "nums": [1, 2, 3, 4, 5] }
```

### 📌 Expression

```json
{ "$reverseArray": "$nums" }
```

### 📤 Output

```json
[5, 4, 3, 2, 1]
```

---

## ✅ Base Example – Reverse Strings

### 📥 Input Document

```json
{ "tags": ["new", "sale", "exclusive"] }
```

### 📌 Expression

```json
{ "$reverseArray": "$tags" }
```

### 📤 Output

```json
["exclusive", "sale", "new"]
```

---

## 🧱 Ecommerce Example – Show Most Recent Orders First

### 📌 Pipeline

```json
[
  {
    "$project": {
      "customer": "$name",
      "recentOrders": {
        "$reverseArray": "$orders"
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "name": "Alice",
  "orders": [
    { "orderId": 1, "amount": 50 },
    { "orderId": 2, "amount": 75 },
    { "orderId": 3, "amount": 30 }
  ]
}
```

### 📤 Output

```json
{
  "customer": "Alice",
  "recentOrders": [
    { "orderId": 3, "amount": 30 },
    { "orderId": 2, "amount": 75 },
    { "orderId": 1, "amount": 50 }
  ]
}
```

---

## 🔧 Common Use Cases

- Display latest entries first
- Change processing order
- Use before slicing or accessing via index

---

## 🔗 Related Operators

- `$slice`, `$arrayElemAt`, `$map`, `$concatArrays`

---

## 🧠 Notes

- Input must be an array; otherwise returns `null`.
- Does not modify the original array.

---
