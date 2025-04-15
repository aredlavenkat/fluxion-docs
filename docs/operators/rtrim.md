# $rtrim

The `$rtrim` operator removes specified characters from the **end (right)** of a string.

---

## 📌 Syntax

```json
{
  "$rtrim": {
    "input": <expression>,
    "chars": <charsToTrim>
  }
}
```

- `input`: The string to trim from the right
- `chars`: Characters to remove (defaults to whitespace)

---

## ✅ Base Example 1 – Trim Trailing Whitespace

### 📥 Input Document

```json
{ "code": "SKU100   " }
```

### 📌 Expression

```json
{ "$rtrim": { "input": "$code" } }
```

### 📤 Output

```json
"SKU100"
```

---

## ✅ Base Example 2 – Trim Dashes from End

### 📥 Input Document

```json
{ "slug": "tshirt---" }
```

### 📌 Expression

```json
{
  "$rtrim": {
    "input": "$slug",
    "chars": "-"
  }
}
```

### 📤 Output

```json
"tshirt"
```

---

## 🧱 Ecommerce Example – Trim Codes for Export

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "cleanCode": {
        "$rtrim": {
          "input": "$items.sku",
          "chars": "-"
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
    { "sku": "ABC123---", "name": "Bag" },
    { "sku": "XYZ999-", "name": "Shoes" }
  ]
}
```

### 📤 Output

```json
[
  { "cleanCode": "ABC123", "product": "Bag" },
  { "cleanCode": "XYZ999", "product": "Shoes" }
]
```

---

## 🔧 Common Use Cases

- Remove unwanted trailing symbols
- Clean up formatting before exporting
- Strip padding or delimiters from strings

---

## 🔗 Related Operators

- `$ltrim`, `$trim`, `$toLower`, `$substr`

---

## 🧠 Notes

- If `chars` is omitted, whitespace is removed.
- Use with `$ltrim` or `$trim` to handle full sides.

---
