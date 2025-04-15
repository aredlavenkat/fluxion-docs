# $switch

The `$switch` operator allows multiple conditional branches, similar to a traditional `switch-case` statement.

---

## 📌 Syntax

```json
{
  "$switch": {
    "branches": [
      { "case": <expression>, "then": <result> },
      ...
    ],
    "default": <defaultResult>
  }
}
```

---

## ✅ Base Example – Grade Evaluation

### 📥 Input Document

```json
{ "score": 85 }
```

### 📌 Expression

```json
{
  "$switch": {
    "branches": [
      { "case": { "$gte": ["$score", 90] }, "then": "A" },
      { "case": { "$gte": ["$score", 80] }, "then": "B" },
      { "case": { "$gte": ["$score", 70] }, "then": "C" }
    ],
    "default": "F"
  }
}
```

### 📤 Output

```json
"B"
```

---

## 🧱 Ecommerce Example – Price Bracket Label

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "product": "$items.name",
      "priceBracket": {
        "$switch": {
          "branches": [
            { "case": { "$lte": ["$items.price", 50] }, "then": "Budget" },
            { "case": { "$lte": ["$items.price", 200] }, "then": "Mid-Range" },
            { "case": { "$lte": ["$items.price", 1000] }, "then": "Premium" }
          ],
          "default": "Luxury"
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
    { "name": "Notebook", "price": 30 },
    { "name": "Tablet", "price": 180 },
    { "name": "Smartphone", "price": 999 },
    { "name": "Laptop", "price": 1200 }
  ]
}
```

### 📤 Output

```json
[
  { "product": "Notebook", "priceBracket": "Budget" },
  { "product": "Tablet", "priceBracket": "Mid-Range" },
  { "product": "Smartphone", "priceBracket": "Premium" },
  { "product": "Laptop", "priceBracket": "Luxury" }
]
```

---

## 🔧 Common Use Cases

- Multi-condition branching
- Label assignment based on ranges
- Fallback logic for exceptions

---

## 🔗 Related Operators

- `$cond`, `$ifNull`, `$or`, `$and`, `$eq`, `$gt`, `$lt`

---

## 🧠 Notes

- Conditions are evaluated in order; the first match is returned.
- If no conditions match, the `default` is used.

---
