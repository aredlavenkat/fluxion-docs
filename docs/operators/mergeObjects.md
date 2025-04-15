# $mergeObjects

The `$mergeObjects` operator merges multiple documents (objects) into a single document. Later fields overwrite earlier ones.

---

## 📌 Syntax

```json
{ "$mergeObjects": [ <doc1>, <doc2>, ... ] }
```

- Accepts an array of documents or document expressions
- Fields in later documents override those in earlier ones

---

## ✅ Base Example – Merge Static Objects

### 📥 Input Document

```json
{ "a": { "x": 1 }, "b": { "y": 2, "x": 3 } }
```

### 📌 Expression

```json
{ "$mergeObjects": ["$a", "$b"] }
```

### 📤 Output

```json
{ "x": 3, "y": 2 }
```

---

## 🧱 Ecommerce Example – Combine Product Attributes

### 📌 Pipeline

```json
[
  {
    "$project": {
      "product": "$name",
      "fullSpec": {
        "$mergeObjects": ["$specs.general", "$specs.technical"]
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "name": "Tablet",
  "specs": {
    "general": { "brand": "XTab", "color": "Black" },
    "technical": { "ram": "8GB", "color": "Gray" }
  }
}
```

### 📤 Output

```json
{
  "product": "Tablet",
  "fullSpec": {
    "brand": "XTab",
    "color": "Gray",
    "ram": "8GB"
  }
}
```

---

## 🔧 Common Use Cases

- Merge attribute sets
- Combine multiple partial documents
- Override fields selectively

---

## 🔗 Related Operators

- `$arrayToObject`, `$objectToArray`, `$getField`, `$setField`

---

## 🧠 Notes

- If merging non-documents (e.g., `null`, arrays), behavior may vary
- Null values are ignored unless merged explicitly

---
