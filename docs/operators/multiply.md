# $multiply

The `$multiply` operator multiplies numbers together.

---

## 📌 Syntax

```json
{ "$multiply": [ <expression1>, <expression2>, ... ] }
```

Each expression must resolve to a numeric value.

---

## ✅ Base Example – Multiply Price by Quantity

### 📥 Input Document

```json
{ "price": 20, "qty": 3 }
```

### 📌 Expression

```json
{ "$multiply": ["$price", "$qty"] }
```

### 📤 Output

```json
60
```

---

## 🧱 Deep Nested Example – Compute Line Totals in Order

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "product": "$items.name",
      "lineTotal": {
        "$multiply": ["$items.price", "$items.quantity"]
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "orderId": 1001,
  "items": [
    { "name": "Pen", "price": 2, "quantity": 5 },
    { "name": "Notebook", "price": 3, "quantity": 2 }
  ]
}
```

### 📤 Output Documents

```json
[
  { "product": "Pen", "lineTotal": 10 },
  { "product": "Notebook", "lineTotal": 6 }
]
```

---

## 🔧 Common Use Cases

- Calculating total cost (`price * qty`)
- Tax/value scaling
- Aggregated computations in `$group`

---

## 🔗 Related

- `$add`, `$subtract`, `$divide`, `$mod`
- `$group`, `$project`, `$set`

---
