# $min

The `$min` operator returns the **smallest value** from a list of expressions or grouped values.

---

## 📌 Syntax

```json
{ "$min": <expression> }
```

Used in:
- `$group` (accumulator)
- Array contexts (e.g., `$map`, `$reduce`)
- Arithmetic expressions

---

## ✅ Base Example – Minimum in Array

### 📥 Input Document

```json
{ "numbers": [15, 5, 8] }
```

### 📌 Expression

```json
{ "$min": "$numbers" }
```

### 📤 Output

```json
5
```

---

## 🧱 Ecommerce Example – Minimum Discounted Price Per Brand

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$group": {
      "_id": "$items.brand",
      "minPrice": { "$min": "$items.price" }
    }
  }
]
```

### 📥 Input Documents

```json
[
  {
    "items": [
      { "brand": "A", "price": 50 },
      { "brand": "A", "price": 30 },
      { "brand": "B", "price": 40 }
    ]
  }
]
```

### 📤 Output

```json
[
  { "_id": "A", "minPrice": 30 },
  { "_id": "B", "minPrice": 40 }
]
```

---

## 🔧 Common Use Cases

- Calculate lowest value in a group
- Use in price comparisons
- Extract min date/time in event logs

---

## 🔗 Related Operators

- `$max`, `$avg`, `$first`, `$last`, `$reduce`

---

## 🧠 Notes

- In `$group`, must be used with grouped documents
- Use with caution on mixed-type arrays

---
