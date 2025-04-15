# $divide

The `$divide` operator divides one number by another.

---

## 📌 Syntax

```json
{ "$divide": [ <numerator>, <denominator> ] }
```

Both arguments must resolve to numeric values. Division by zero will result in an error or `null`.

---

## ✅ Base Example 1 – Simple Division

### 📥 Input Document

```json
{ "total": 100, "parts": 4 }
```

### 📌 Expression

```json
{ "$divide": ["$total", "$parts"] }
```

### 📤 Output

```json
25
```

---

## ✅ Base Example 2 – Convert Cents to Dollars

### 📥 Input Document

```json
{ "amountInCents": 1250 }
```

### 📌 Expression

```json
{ "$divide": ["$amountInCents", 100] }
```

### 📤 Output

```json
12.5
```

---

## 🧱 Ecommerce Example – Compute Unit Price per Item

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "product": "$items.name",
      "unitPrice": {
        "$divide": ["$items.totalPrice", "$items.quantity"]
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "orderId": 2002,
  "items": [
    { "name": "Chair", "totalPrice": 300, "quantity": 3 },
    { "name": "Desk", "totalPrice": 500, "quantity": 1 }
  ]
}
```

### 📤 Output

```json
[
  { "product": "Chair", "unitPrice": 100 },
  { "product": "Desk", "unitPrice": 500 }
]
```

---

## 🧱 Ecommerce Example – Discount Percent

### 📌 Expression

```json
{ "$divide": ["$discount", "$price"] }
```

Used to compute the proportion of discount relative to the price.

### 📥 Input Document

```json
{ "price": 250, "discount": 25 }
```

### 📤 Output

```json
0.1
```

---

## 🔧 Common Use Cases

- Price per unit
- Percentage calculations
- Normalization

---

## 🔗 Related Operators

- `$add`, `$subtract`, `$multiply`, `$mod`
- `$cond`, `$round`, `$project`

---

## 🧠 Notes

- If denominator is `0`, the result may be `null` or cause an error.
- Always validate for zero if dynamic input is used.

---
