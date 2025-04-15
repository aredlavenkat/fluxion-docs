# $toLower

The `$toLower` operator converts a string to all lowercase letters.

---

## 📌 Syntax

```json
{ "$toLower": <expression> }
```

The expression must resolve to a string.

---

## ✅ Base Example 1 – Lowercase Email

### 📥 Input Document

```json
{ "email": "USER@EXAMPLE.COM" }
```

### 📌 Expression

```json
{ "$toLower": "$email" }
```

### 📤 Output

```json
"user@example.com"
```

---

## ✅ Base Example 2 – City Name Normalization

### 📥 Input Document

```json
{ "city": "Toronto" }
```

### 📌 Expression

```json
{ "$toLower": "$city" }
```

### 📤 Output

```json
"toronto"
```

---

## 🧱 Ecommerce Example – Normalize Brand Name

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "brand_normalized": {
        "$toLower": "$items.brand"
      },
      "product": "$items.name"
    }
  }
]
```

### 📥 Input Document

```json
{
  "items": [
    { "name": "Laptop", "brand": "HP" },
    { "name": "Keyboard", "brand": "LogiTech" }
  ]
}
```

### 📤 Output

```json
[
  { "brand_normalized": "hp", "product": "Laptop" },
  { "brand_normalized": "logitech", "product": "Keyboard" }
]
```

---

## 🔧 Common Use Cases

- Case-insensitive comparisons
- Normalization for search, filter, and indexing
- String unification in user inputs

---

## 🔗 Related Operators

- `$toUpper`, `$trim`, `$concat`, `$substr`, `$toString`

---

## 🧠 Notes

- If the input is not a string, it will raise an error.
- Use `$ifNull` or `$cond` to handle missing values gracefully.

---
