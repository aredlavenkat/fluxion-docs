# $replaceWith

The `$replaceWith` stage is an alias for `$replaceRoot` but supports **any expression**, not just a field path.

---

## 📌 Syntax

```json
{ "$replaceWith": <expression> }
```

---

## ✅ Base Example – Replace With Embedded Customer

```json
{ "$replaceWith": "$customer" }
```

---

## 🧱 Deep Nested Example – Merge Order ID with Customer Info

### 📌 Stage

```json
{
  "$replaceWith": {
    "$mergeObjects": ["$customer", { "orderId": "$orderId" }]
  }
}
```

### 📥 Input

```json
{
  "orderId": 55,
  "customer": {
    "name": "Bob",
    "email": "bob@example.com"
  }
}
```

### 📤 Output

```json
{
  "name": "Bob",
  "email": "bob@example.com",
  "orderId": 55
}
```

---

## 🔧 Common Operators

- `$mergeObjects`, `$map`, `$reduce`, `$cond`

---

## 🔗 Related

- `$replaceRoot`, `$addFields`, `$project`

---
