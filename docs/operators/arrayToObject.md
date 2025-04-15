# $arrayToObject

The `$arrayToObject` operator transforms an array of key-value documents into a single document (object).

---

## 📌 Syntax

```json
{ "$arrayToObject": <arrayExpression> }
```

Each array element must be a document with `"k"` (key) and `"v"` (value) fields.

---

## ✅ Base Example – Convert Array to Object

### 📥 Input Document

```json
{
  "pairs": [
    { "k": "a", "v": 1 },
    { "k": "b", "v": 2 }
  ]
}
```

### 📌 Expression

```json
{ "$arrayToObject": "$pairs" }
```

### 📤 Output

```json
{ "a": 1, "b": 2 }
```

---

## 🧱 Ecommerce Example – Reconstruct Dynamic Attributes

### 📌 Pipeline

```json
[
  {
    "$project": {
      "product": "$name",
      "properties": {
        "$arrayToObject": "$attributes"
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "name": "Shoes",
  "attributes": [
    { "k": "color", "v": "Black" },
    { "k": "size", "v": "42" }
  ]
}
```

### 📤 Output

```json
{
  "product": "Shoes",
  "properties": {
    "color": "Black",
    "size": "42"
  }
}
```

---

## 🔧 Common Use Cases

- Reconstruct flattened key-value arrays
- Transform map-reduce style output
- Enable object-style access to filtered key-value results

---

## 🔗 Related Operators

- `$objectToArray`, `$map`, `$mergeObjects`, `$getField`, `$setField`

---

## 🧠 Notes

- Input array must be in `[{k, v}]` format
- Duplicate keys will overwrite earlier values

---
