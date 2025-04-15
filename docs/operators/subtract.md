# $subtract

The `$subtract` operator subtracts one number (or date) from another.

---

## 📌 Syntax

```json
{ "$subtract": [ <expression1>, <expression2> ] }
```

`<expression1>` is the value to subtract from.  
`<expression2>` is the value to subtract.

---

## ✅ Base Example – Subtract Discount from Price

### 📥 Input Document

```json
{ "price": 100, "discount": 25 }
```

### 📌 Expression

```json
{ "$subtract": ["$price", "$discount"] }
```

### 📤 Output

```json
75
```

---

## 🧱 Deep Nested Example – Compute Remaining Balance per Item

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "product": "$items.name",
      "due": {
        "$subtract": ["$items.total", "$items.paid"]
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "orderId": 2,
  "items": [
    { "name": "Monitor", "total": 200, "paid": 150 },
    { "name": "Keyboard", "total": 100, "paid": 100 }
  ]
}
```

### 📤 Output Documents

```json
[
  { "product": "Monitor", "due": 50 },
  { "product": "Keyboard", "due": 0 }
]
```

---

## 🔧 Common Use Cases

- Calculating remaining balances
- Date differences (with ISODate types)
- Price reductions and change tracking

---

## 🔗 Related

- `$add`, `$multiply`, `$divide`
- `$project`, `$group`, `$set`

---
