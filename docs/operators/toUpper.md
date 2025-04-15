# $toUpper

The `$toUpper` operator converts a string to all uppercase letters.

---

## 📌 Syntax

```json
{ "$toUpper": <expression> }
```

The expression must resolve to a string.

---

## ✅ Base Example 1 – Uppercase City Name

### 📥 Input Document

```json
{ "city": "toronto" }
```

### 📌 Expression

```json
{ "$toUpper": "$city" }
```

### 📤 Output

```json
"TORONTO"
```

---

## ✅ Base Example 2 – Uppercase Username for Display

### 📥 Input Document

```json
{ "username": "janedoe" }
```

### 📌 Expression

```json
{ "$toUpper": "$username" }
```

### 📤 Output

```json
"JANEDOE"
```

---

## 🧱 Ecommerce Example – Uppercase Category Labels

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "categoryUpper": { "$toUpper": "$items.category" },
      "product": "$items.name"
    }
  }
]
```

### 📥 Input Document

```json
{
  "items": [
    { "name": "Shoes", "category": "fashion" },
    { "name": "Tablet", "category": "electronics" }
  ]
}
```

### 📤 Output

```json
[
  { "categoryUpper": "FASHION", "product": "Shoes" },
  { "categoryUpper": "ELECTRONICS", "product": "Tablet" }
]
```

---

## 🔧 Common Use Cases

- Case normalization
- Visual formatting for UI
- Standardizing categories and labels

---

## 🔗 Related Operators

- `$toLower`, `$trim`, `$concat`, `$substr`, `$toString`

---

## 🧠 Notes

- Input must be a string.
- Combine with `$ifNull` or `$cond` to avoid null errors.

---
