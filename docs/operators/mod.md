# $mod

The `$mod` operator returns the remainder of the division of one number by another.

---

## 📌 Syntax

```json
{ "$mod": [ <dividend>, <divisor> ] }
```

Both values must resolve to numbers. Division by zero returns an error.

---

## ✅ Base Example 1 – Modulo with Even/Odd Check

### 📥 Input Document

```json
{ "orderId": 101 }
```

### 📌 Expression

```json
{ "$mod": ["$orderId", 2] }
```

### 📤 Output

```json
1
```

> Output `0` means even, `1` means odd.

---

## ✅ Base Example 2 – Minutes Past the Hour

### 📥 Input Document

```json
{ "elapsedSeconds": 3670 }
```

### 📌 Expression

```json
{ "$mod": ["$elapsedSeconds", 3600] }
```

### 📤 Output

```json
70
```

---

## 🧱 Ecommerce Example – Discount Every 3rd Item

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "name": "$items.name",
      "isEligibleForDiscount": {
        "$eq": [{ "$mod": ["$items.index", 3] }, 0]
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "items": [
    { "name": "Item A", "index": 0 },
    { "name": "Item B", "index": 1 },
    { "name": "Item C", "index": 2 },
    { "name": "Item D", "index": 3 }
  ]
}
```

### 📤 Output

```json
[
  { "name": "Item A", "isEligibleForDiscount": true },
  { "name": "Item B", "isEligibleForDiscount": false },
  { "name": "Item C", "isEligibleForDiscount": false },
  { "name": "Item D", "isEligibleForDiscount": true }
]
```

---

## 🔧 Common Use Cases

- Even/odd classification
- Recurring logic (every N items)
- Time breakdowns (minutes/seconds)

---

## 🔗 Related Operators

- `$divide`, `$add`, `$subtract`, `$cond`
- `$eq`, `$project`, `$switch`

---

## 🧠 Notes

- Dividing by zero will return an error.
- Common in coupon, rule, and scheduling systems.

---
