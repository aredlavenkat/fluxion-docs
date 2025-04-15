# $objectToArray

The `$objectToArray` operator converts a document (object) into an array of key-value pair documents.

---

## 📌 Syntax

```json
{ "$objectToArray": <documentExpression> }
```

Each entry in the output array has the form:

```json
{ "k": <key>, "v": <value> }
```

---

## ✅ Base Example – Convert Object to Array

### 📥 Input Document

```json
{ "data": { "x": 1, "y": 2 } }
```

### 📌 Expression

```json
{ "$objectToArray": "$data" }
```

### 📤 Output

```json
[
  { "k": "x", "v": 1 },
  { "k": "y", "v": 2 }
]
```

---

## 🧱 Ecommerce Example – Flatten Dynamic Properties

### 📌 Pipeline

```json
[
  {
    "$project": {
      "product": "$name",
      "attributes": {
        "$objectToArray": "$customAttributes"
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "name": "Sneakers",
  "customAttributes": {
    "size": "10",
    "color": "Red",
    "style": "Casual"
  }
}
```

### 📤 Output

```json
{
  "product": "Sneakers",
  "attributes": [
    { "k": "size", "v": "10" },
    { "k": "color", "v": "Red" },
    { "k": "style", "v": "Casual" }
  ]
}
```

---

## 🔧 Common Use Cases

- Transform object keys into documents
- Use with `$map` and `$filter` on object properties
- Enable custom aggregation logic for dynamic keys

---

## 🔗 Related Operators

- `$arrayToObject`, `$getField`, `$setField`, `$mergeObjects`

---

## 🧠 Notes

- Input must be an object; returns error on arrays or null
- Often paired with `$map` or `$reduce` for key-based logic

---
