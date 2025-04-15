# $concatArrays

The `$concatArrays` operator merges two or more arrays into a single array.

---

## 📌 Syntax

```json
{ "$concatArrays": [ <array1>, <array2>, ... ] }
```

---

## ✅ Base Example – Merge Lists

### 📥 Input Document

```json
{ "a": [1, 2], "b": [3, 4] }
```

### 📌 Expression

```json
{ "$concatArrays": ["$a", "$b"] }
```

### 📤 Output

```json
[1, 2, 3, 4]
```

---

## ✅ Base Example – Add Single Item

### 📥 Input Document

```json
{ "items": ["Pen", "Book"] }
```

### 📌 Expression

```json
{ "$concatArrays": ["$items", ["Notebook"]] }
```

### 📤 Output

```json
["Pen", "Book", "Notebook"]
```

---

## 🧱 Ecommerce Example – Combine Inventory Locations

### 📌 Pipeline

```json
[
  {
    "$project": {
      "product": "$name",
      "allLocations": {
        "$concatArrays": ["$warehouseA", "$warehouseB"]
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "name": "Speaker",
  "warehouseA": ["W1", "W2"],
  "warehouseB": ["W3"]
}
```

### 📤 Output

```json
{
  "product": "Speaker",
  "allLocations": ["W1", "W2", "W3"]
}
```

---

## 🔧 Common Use Cases

- Merge multiple inventories
- Combine feature lists
- Append new values to arrays

---

## 🔗 Related Operators

- `$arrayElemAt`, `$slice`, `$map`, `$reduce`, `$literal`

---

## 🧠 Notes

- Works only on arrays; input expressions must evaluate to arrays.
- Null or missing inputs are treated as `null` and return `null`.

---
