# $size

The `$size` operator returns the **number of elements** in an array.

---

## 📌 Syntax

```json
{ "$size": <arrayExpression> }
```

The input must resolve to an array.

---

## ✅ Base Example – Count Elements

### 📥 Input Document

```json
{ "tags": ["sale", "new", "exclusive"] }
```

### 📌 Expression

```json
{ "$size": "$tags" }
```

### 📤 Output

```json
3
```

---

## ✅ Base Example – Empty Array

### 📥 Input Document

```json
{ "items": [] }
```

### 📌 Expression

```json
{ "$size": "$items" }
```

### 📤 Output

```json
0
```

---

## 🧱 Ecommerce Example – Count Item Features

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "product": "$items.name",
      "featureCount": { "$size": "$items.features" }
    }
  }
]
```

### 📥 Input Document

```json
{
  "items": [
    {
      "name": "Backpack",
      "features": ["Padded", "Waterproof", "Adjustable Straps"]
    },
    {
      "name": "Watch",
      "features": []
    }
  ]
}
```

### 📤 Output

```json
[
  { "product": "Backpack", "featureCount": 3 },
  { "product": "Watch", "featureCount": 0 }
]
```

---

## 🔧 Common Use Cases

- Count number of tags, features, options
- Validate minimum/maximum items
- Use with `$cond` or `$switch` for conditional logic

---

## 🔗 Related Operators

- `$isArray`, `$map`, `$filter`, `$arrayElemAt`

---

## 🧠 Notes

- Returns an error if input is not an array
- Use `$ifNull` or `$cond` for safety

---
