# $reduce

The `$reduce` operator applies an expression to each element in an array and **accumulates** the result.

---

## 📌 Syntax

```json
{
  "$reduce": {
    "input": <arrayExpression>,
    "initialValue": <expression>,
    "in": <accumulatorExpression>
  }
}
```

- `input`: The array to iterate
- `initialValue`: The starting value of the accumulator
- `in`: Expression that updates the accumulator using `$$value` and `$$this`

---

## ✅ Base Example – Sum Values

### 📥 Input Document

```json
{ "nums": [1, 2, 3, 4] }
```

### 📌 Expression

```json
{
  "$reduce": {
    "input": "$nums",
    "initialValue": 0,
    "in": { "$add": ["$$value", "$$this"] }
  }
}
```

### 📤 Output

```json
10
```

---

## ✅ Base Example – Concatenate Strings

### 📥 Input Document

```json
{ "words": ["hello", "world"] }
```

### 📌 Expression

```json
{
  "$reduce": {
    "input": "$words",
    "initialValue": "",
    "in": { "$concat": ["$$value", " ", "$$this"] }
  }
}
```

### 📤 Output

```json
" hello world"
```

---

## 🧱 Ecommerce Example – Total Price of All Items

### 📌 Pipeline

```json
[
  {
    "$project": {
      "total": {
        "$reduce": {
          "input": "$items",
          "initialValue": 0,
          "in": {
            "$add": ["$$value", { "$multiply": ["$$this.price", "$$this.quantity"] }]
          }
        }
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "items": [
    { "name": "Book", "price": 20, "quantity": 2 },
    { "name": "Bag", "price": 50, "quantity": 1 }
  ]
}
```

### 📤 Output

```json
{ "total": 90 }
```

---

## 🔧 Common Use Cases

- Compute totals or nested accumulations
- String aggregation
- Custom transformations

---

## 🔗 Related Operators

- `$map`, `$filter`, `$concat`, `$sum`, `$avg`

---

## 🧠 Notes

- `$$value` refers to the accumulator
- `$$this` refers to the current array element

---
