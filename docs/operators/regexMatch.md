# $regexMatch

The `$regexMatch` operator evaluates whether a string input matches a regular expression pattern.

---

## 📌 Syntax

```json
{
  "$regexMatch": {
    "input": <string>,
    "regex": <pattern>,
    "options": <string>
  }
}
```

- `input`: The string to test.
- `regex`: The pattern to match.
- `options`: Optional flags like `"i"` (case-insensitive), `"m"` (multiline), etc.

---

## ✅ Base Example 1 – Match Case-Insensitive Word

### 📥 Input Document

```json
{ "name": "Wireless Mouse" }
```

### 📌 Expression

```json
{
  "$regexMatch": {
    "input": "$name",
    "regex": "mouse",
    "options": "i"
  }
}
```

### 📤 Output

```json
true
```

---

## ✅ Base Example 2 – Validate Product ID Format

### 📥 Input Document

```json
{ "productId": "ABC-1234" }
```

### 📌 Expression

```json
{
  "$regexMatch": {
    "input": "$productId",
    "regex": "^[A-Z]{3}-\d{4}$"
  }
}
```

### 📤 Output

```json
true
```

---

## 🧱 Ecommerce Example – Check Brand Label Format

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "brandValid": {
        "$regexMatch": {
          "input": "$items.brand",
          "regex": "^[A-Z]{4,6}$"
        }
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
    { "name": "TV", "brand": "SONY" },
    { "name": "Book", "brand": "bk01" }
  ]
}
```

### 📤 Output

```json
[
  { "brandValid": true, "product": "TV" },
  { "brandValid": false, "product": "Book" }
]
```

---

## 🔧 Common Use Cases

- Format validation (e.g., SKU, ID)
- Keyword searching
- Input sanitization

---

## 🔗 Related Operators

- `$regexFind`, `$regexFindAll`, `$regexMatch`, `$toLower`, `$split`

---

## 🧠 Notes

- Use `\` to escape characters like `\d` in JSON.
- Combine with `$cond` or `$match` for rule evaluation.

---
