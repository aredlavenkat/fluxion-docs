# $cond

The `$cond` operator is a conditional (if–then–else) operator.

It evaluates a condition and returns one of two expressions depending on whether the condition is true or false.

---

## 📌 Syntax

### ✅ Object Syntax

```json
{
  "$cond": {
    "if": <condition>,
    "then": <expression-if-true>,
    "else": <expression-if-false>
  }
}
```

### ✅ Array Syntax

```json
{ "$cond": [ <condition>, <then>, <else> ] }
```

---

## ✅ Base Example 1 – Membership Status

### 📥 Input Document

```json
{ "isMember": true }
```

### 📌 Expression

```json
{
  "$cond": {
    "if": "$isMember",
    "then": "Discounted",
    "else": "Regular"
  }
}
```

### 📤 Output

```json
"Discounted"
```

---

## ✅ Base Example 2 – Array Syntax

### 📥 Input Document

```json
{ "score": 65 }
```

### 📌 Expression

```json
{ "$cond": [ { "$gte": ["$score", 70] }, "Pass", "Fail" ] }
```

### 📤 Output

```json
"Fail"
```

---

## 🧱 Ecommerce Example – Shipping Fee Waiver

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "name": "$items.name",
      "shippingFee": {
        "$cond": {
          "if": { "$gte": ["$items.price", 100] },
          "then": 0,
          "else": 10
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
    { "name": "Monitor", "price": 150 },
    { "name": "Cable", "price": 20 }
  ]
}
```

### 📤 Output

```json
[
  { "name": "Monitor", "shippingFee": 0 },
  { "name": "Cable", "shippingFee": 10 }
]
```

---

## 🔧 Common Use Cases

- Apply different logic based on price, quantity, status
- Select values conditionally inside `$project`, `$group`, `$map`
- Use inside `$reduce`, `$switch`, `$filter`

---

## 🔗 Related Operators

- `$ifNull`, `$switch`, `$eq`, `$gt`, `$lt`, `$gte`, `$lte`

---

## 🧠 Notes

- Flexible control structure in aggregation pipelines
- Both array and object syntax are supported by MongoDB (and Fluxion)

---
