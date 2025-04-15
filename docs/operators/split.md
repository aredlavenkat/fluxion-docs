# $split

The `$split` operator divides a string into an array of substrings based on a specified delimiter.

---

## 📌 Syntax

```json
{ "$split": [ <string>, <delimiter> ] }
```

---

## ✅ Base Example 1 – Split Email Address

### 📥 Input Document

```json
{ "email": "user@example.com" }
```

### 📌 Expression

```json
{ "$split": ["$email", "@"] }
```

### 📤 Output

```json
["user", "example.com"]
```

---

## ✅ Base Example 2 – Break Path

### 📥 Input Document

```json
{ "path": "inventory/electronics/phones" }
```

### 📌 Expression

```json
{ "$split": ["$path", "/"] }
```

### 📤 Output

```json
["inventory", "electronics", "phones"]
```

---

## 🧱 Ecommerce Example – Parse Composite SKU Codes

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "skuParts": {
        "$split": ["$items.sku", "-"]
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
    { "name": "Laptop", "sku": "ELEC-BRND-X123" },
    { "name": "Shoes", "sku": "FASH-BRND-Y456" }
  ]
}
```

### 📤 Output

```json
[
  { "skuParts": ["ELEC", "BRND", "X123"], "product": "Laptop" },
  { "skuParts": ["FASH", "BRND", "Y456"], "product": "Shoes" }
]
```

---

## 🔧 Common Use Cases

- Email/username separation
- Path decomposition
- SKU/code parsing

---

## 🔗 Related Operators

- `$arrayElemAt`, `$indexOfArray`, `$substr`, `$split`, `$map`

---

## 🧠 Notes

- If the delimiter is not found, the entire string is returned in a single-element array.
- Returns an empty array when splitting an empty string.

---
