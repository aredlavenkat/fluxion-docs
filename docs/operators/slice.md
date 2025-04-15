# $slice

The `$slice` operator returns a **subset of an array**. You can specify the number of elements to return, and optionally, the starting position.

---

## 📌 Syntax

```json
{ "$slice": [ <arrayExpression>, <n> ] }
```

or

```json
{ "$slice": [ <arrayExpression>, <start>, <n> ] }
```

- `arrayExpression`: The array to slice.
- `n`: The number of elements to return.
- `start`: (optional) The index to start slicing from.

---

## ✅ Base Example 1 – First 2 Elements

### 📥 Input Document

```json
{ "values": [10, 20, 30, 40] }
```

### 📌 Expression

```json
{ "$slice": ["$values", 2] }
```

### 📤 Output

```json
[10, 20]
```

---

## ✅ Base Example 2 – Middle Slice

### 📥 Input Document

```json
{ "values": [10, 20, 30, 40, 50] }
```

### 📌 Expression

```json
{ "$slice": ["$values", 1, 3] }
```

### 📤 Output

```json
[20, 30, 40]
```

---

## 🧱 Ecommerce Example – Get First 2 Features

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "product": "$items.name",
      "topFeatures": {
        "$slice": ["$items.features", 2]
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "items": [
    {
      "name": "Smartwatch",
      "features": ["Bluetooth", "Heart Rate", "GPS", "Waterproof"]
    }
  ]
}
```

### 📤 Output

```json
[
  {
    "product": "Smartwatch",
    "topFeatures": ["Bluetooth", "Heart Rate"]
  }
]
```

---

## 🔧 Common Use Cases

- Truncate arrays
- Return previews
- Extract top-N items

---

## 🔗 Related Operators

- `$arrayElemAt`, `$filter`, `$reduce`, `$map`, `$indexOfArray`

---

## 🧠 Notes

- Negative `n` removes elements from the end.
- Use with `$sort` to return top records before slicing.

---
