# $isArray

The `$isArray` operator checks whether a given expression resolves to an **array**.

---

## 📌 Syntax

```json
{ "$isArray": <expression> }
```

Returns `true` if the result is an array, otherwise `false`.

---

## ✅ Base Example 1 – Array Check

### 📥 Input Document

```json
{ "tags": ["sale", "new"] }
```

### 📌 Expression

```json
{ "$isArray": "$tags" }
```

### 📤 Output

```json
true
```

---

## ✅ Base Example 2 – Not an Array

### 📥 Input Document

```json
{ "price": 19.99 }
```

### 📌 Expression

```json
{ "$isArray": "$price" }
```

### 📤 Output

```json
false
```

---

## 🧱 Ecommerce Example – Validate Variant Field

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "product": "$items.name",
      "hasVariants": { "$isArray": "$items.variants" }
    }
  }
]
```

### 📥 Input Document

```json
{
  "items": [
    { "name": "Shirt", "variants": ["S", "M", "L"] },
    { "name": "Gift Card", "variants": null }
  ]
}
```

### 📤 Output

```json
[
  { "product": "Shirt", "hasVariants": true },
  { "product": "Gift Card", "hasVariants": false }
]
```

---

## 🔧 Common Use Cases

- Input validation
- Conditional logic on array-dependent fields
- Filter dynamic JSON structures

---

## 🔗 Related Operators

- `$size`, `$filter`, `$type`, `$ifNull`, `$cond`

---

## 🧠 Notes

- Use in combination with `$cond` to branch logic.
- Helps clean malformed documents with inconsistent schemas.

---
