# $max

The `$max` operator returns the **highest value** from a list of expressions or grouped values.

---

## 📌 Syntax

```json
{ "$max": <expression> }
```

Used in:
- `$group` (accumulator)
- Array contexts
- Numeric comparisons

---

## ✅ Base Example – Maximum in Array

### 📥 Input Document

```json
{ "scores": [85, 92, 78] }
```

### 📌 Expression

```json
{ "$max": "$scores" }
```

### 📤 Output

```json
92
```

---

## 🧱 Ecommerce Example – Maximum Product Price Per Brand

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$group": {
      "_id": "$items.brand",
      "maxPrice": { "$max": "$items.price" }
    }
  }
]
```

### 📥 Input Documents

```json
[
  {
    "items": [
      { "brand": "A", "price": 100 },
      { "brand": "A", "price": 130 },
      { "brand": "B", "price": 90 }
    ]
  }
]
```

### 📤 Output

```json
[
  { "_id": "A", "maxPrice": 130 },
  { "_id": "B", "maxPrice": 90 }
]
```

---

## 🔧 Common Use Cases

- Determine highest price, score, value
- Identify latest timestamps
- Combine with `$group` for summary statistics

---

## 🔗 Related Operators

- `$min`, `$avg`, `$sum`, `$reduce`, `$first`, `$last`

---

## 🧠 Notes

- Can be used directly in array or accumulator contexts
- Behavior on `null` values depends on placement/order

---
