# $arrayElemAt

The `$arrayElemAt` operator returns the **element at a specified index** in an array.

---

## 📌 Syntax

```json
{ "$arrayElemAt": [ <arrayExpression>, <index> ] }
```

- Indexing starts at 0.
- Negative indexes count from the end.

---

## ✅ Base Example – Get First Tag

### 📥 Input Document

```json
{ "tags": ["sale", "new", "trending"] }
```

### 📌 Expression

```json
{ "$arrayElemAt": ["$tags", 0] }
```

### 📤 Output

```json
"sale"
```

---

## ✅ Base Example – Get Last Element

### 📥 Input Document

```json
{ "tags": ["sale", "new", "trending"] }
```

### 📌 Expression

```json
{ "$arrayElemAt": ["$tags", -1] }
```

### 📤 Output

```json
"trending"
```

---

## 🧱 Ecommerce Example – Access Top-Scoring Review

### 📌 Pipeline

```json
[
  {
    "$project": {
      "product": "$name",
      "topReview": { "$arrayElemAt": ["$reviews", 0] }
    }
  }
]
```

### 📥 Input Document

```json
{
  "name": "Laptop",
  "reviews": [
    { "user": "Alice", "rating": 5 },
    { "user": "Bob", "rating": 4 }
  ]
}
```

### 📤 Output

```json
{
  "product": "Laptop",
  "topReview": { "user": "Alice", "rating": 5 }
}
```

---

## 🔧 Common Use Cases

- Access top/last items
- Index-based lookups
- Extract highlights or featured content

---

## 🔗 Related Operators

- `$slice`, `$filter`, `$map`, `$reduce`

---

## 🧠 Notes

- Index out of bounds returns `null`.
- Combine with `$size` or `$cond` for safer access.

---
