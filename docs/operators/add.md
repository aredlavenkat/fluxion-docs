# $add

The `$add` operator adds numbers together or concatenates dates and numeric expressions.

---

## 📌 Syntax

```json
{ "$add": [ <expression1>, <expression2>, ... ] }
```

Each expression should resolve to a number or date.

---

## ✅ Base Example – Add Two Fields

### 📥 Input Document

```json
{ "price": 100, "tax": 20 }
```

### 📌 Expression

```json
{ "$add": ["$price", "$tax"] }
```

### 📤 Output

```json
120
```

---

## 🧱 Deep Nested Example – Compute Total for Each Item in an Order

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "product": "$items.name",
      "totalCost": { "$add": ["$items.price", "$items.tax"] }
    }
  }
]
```

### 📥 Input Document

```json
{
  "orderId": 1,
  "items": [
    { "name": "Laptop", "price": 1000, "tax": 130 },
    { "name": "Mouse", "price": 50, "tax": 5 }
  ]
}
```

### 📤 Output Documents

```json
[
  { "product": "Laptop", "totalCost": 1130 },
  { "product": "Mouse", "totalCost": 55 }
]
```

---

## 🔧 Common Use Cases

- Calculating final prices
- Summing quantities
- Adding durations to timestamps

---

## 🔗 Related

- `$subtract`, `$multiply`, `$divide` – arithmetic family
- `$project`, `$group` – stages where `$add` is commonly used

---
