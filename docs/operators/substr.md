# $substr

The `$substr` operator returns a substring of a string, starting at a specified index and with a specified length.

---

## 📌 Syntax

```json
{ "$substr": [ <string>, <start>, <length> ] }
```

- `<string>`: The source string expression
- `<start>`: Zero-based start index
- `<length>`: Number of characters to return

---

## ✅ Base Example 1 – Extract First 3 Letters

### 📥 Input Document

```json
{ "product": "Shoes" }
```

### 📌 Expression

```json
{ "$substr": ["$product", 0, 3] }
```

### 📤 Output

```json
"Sho"
```

---

## ✅ Base Example 2 – Get File Extension

### 📥 Input Document

```json
{ "filename": "invoice2024.pdf" }
```

### 📌 Expression

```json
{ "$substr": ["$filename", 12, 3] }
```

### 📤 Output

```json
"pdf"
```

---

## 🧱 Ecommerce Example – Shorten Brand Prefix from SKU

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "shortSku": { "$substr": ["$items.sku", 0, 5] },
      "product": "$items.name"
    }
  }
]
```

### 📥 Input Document

```json
{
  "items": [
    { "sku": "BRAND12345", "name": "Backpack" },
    { "sku": "DELTA98765", "name": "Laptop" }
  ]
}
```

### 📤 Output

```json
[
  { "shortSku": "BRAND", "product": "Backpack" },
  { "shortSku": "DELTA", "product": "Laptop" }
]
```

---

## 🔧 Common Use Cases

- Extract prefixes, suffixes, or sections
- Parse embedded strings
- Simplify field values for display

---

## 🔗 Related Operators

- `$substrBytes`, `$substrCP`, `$slice`, `$split`, `$indexOfBytes`

---

## 🧠 Notes

- `$substr` is deprecated in favor of `$substrBytes` and `$substrCP` for Unicode correctness.
- Start index out of range returns an empty string.

---
