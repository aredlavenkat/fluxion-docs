# $last

The `$last` operator returns the **last element** in an array or the **last document** in a group.

---

## 📌 Syntax (Aggregation Accumulator)

```json
{ "$last": <expression> }
```

- Used in `$group`, `$reduce`, `$map`, or array contexts.

---

## ✅ Base Example – Array Last Element

### 📥 Input Document

```json
{ "values": [5, 10, 15] }
```

### 📌 Expression

```json
{ "$last": "$values" }
```

### 📤 Output

```json
15
```

---

## 🧱 Ecommerce Example – Last Item in Cart by Time

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  { "$sort": { "items.addedAt": 1 } },
  {
    "$group": {
      "_id": "$orderId",
      "lastItem": { "$last": "$items.name" }
    }
  }
]
```

### 📥 Input Documents

```json
[
  {
    "orderId": 202,
    "items": [
      { "name": "Pen", "addedAt": "2024-01-01" },
      { "name": "Notebook", "addedAt": "2024-01-03" }
    ]
  }
]
```

### 📤 Output

```json
[
  {
    "_id": 202,
    "lastItem": "Notebook"
  }
]
```

---

## 🔧 Common Use Cases

- Return last item in array
- Use with sorted groups
- Useful in `$group`, `$reduce`, or windowed analysis

---

## 🔗 Related Operators

- `$first`, `$min`, `$max`, `$reduce`, `$arrayElemAt`

---

## 🧠 Notes

- In `$group`, the result depends on the order of documents.
- On arrays, behaves like accessing the last index.

---
