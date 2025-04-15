# $ifNull

The `$ifNull` operator returns the **first non-null** expression from the provided list of two expressions.

---

## 📌 Syntax

```json
{ "$ifNull": [ <expression1>, <fallback> ] }
```

---

## ✅ Base Example 1 – Provide Default Value

### 📥 Input Document

```json
{ "nickname": null }
```

### 📌 Expression

```json
{ "$ifNull": ["$nickname", "Guest"] }
```

### 📤 Output

```json
"Guest"
```

---

## ✅ Base Example 2 – Use Alternative Field

### 📥 Input Document

```json
{ "username": "alice" }
```

### 📌 Expression

```json
{ "$ifNull": ["$nickname", "$username"] }
```

### 📤 Output

```json
"alice"
```

---

## 🧱 Ecommerce Example – Show Discount Label

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "product": "$items.name",
      "discountLabel": {
        "$ifNull": ["$items.discountLabel", "Standard"]
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "items": [
    { "name": "Shoes", "discountLabel": "Spring Sale" },
    { "name": "Socks" }
  ]
}
```

### 📤 Output

```json
[
  { "product": "Shoes", "discountLabel": "Spring Sale" },
  { "product": "Socks", "discountLabel": "Standard" }
]
```

---

## 🔧 Common Use Cases

- Provide fallback/default values
- Avoid nulls in views or exports
- Chain with `$cond` or `$switch`

---

## 🔗 Related Operators

- `$cond`, `$switch`, `$coalesce`, `$or`

---

## 🧠 Notes

- Returns `expression1` if it's not null or missing
- Otherwise returns `fallback`

---
