# $first

The `$first` operator returns the **first element** in an array or the **first document** in a group.

---

## 📌 Syntax (Aggregation Accumulator)

```json
{ "$first": <expression> }
```

- Used inside `$group`, `$setWindowFields`, or in array expressions like `$map`, `$reduce`.

---

## ✅ Base Example – Array First Element

### 📥 Input Document

```json
{ "values": [10, 20, 30] }
```

### 📌 Expression

```json
{ "$first": "$values" }
```

### 📤 Output

```json
10
```

---

## 🧱 Ecommerce Example – Get First Ordered Item

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  { "$sort": { "items.addedAt": 1 } },
  {
    "$group": {
      "_id": "$orderId",
      "firstItem": { "$first": "$items.name" }
    }
  }
]
```

### 📥 Input Documents

```json
[
  {
    "orderId": 101,
    "items": [
      { "name": "Laptop", "addedAt": "2024-01-01" },
      { "name": "Mouse", "addedAt": "2024-01-02" }
    ]
  }
]
```

### 📤 Output

```json
[
  {
    "_id": 101,
    "firstItem": "Laptop"
  }
]
```

---

## 🔧 Common Use Cases

- Get the first record in a group
- Pick the first element of an array
- Use in `$group`, `$project`, `$reduce`

---

## 🔗 Related Operators

- `$last`, `$min`, `$max`, `$arrayElemAt`, `$reduce`

---

## 🧠 Notes

- In `$group`, `$first` takes the value from the **first document** in the group — order matters!
- In arrays, it behaves like `arr[0]`.

---
