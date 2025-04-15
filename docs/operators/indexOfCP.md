# $indexOfCP

The `$indexOfCP` operator returns the **character (code point)** index of the first occurrence of a substring in a string.

---

## 📌 Syntax

```json
{ "$indexOfCP": [ <string>, <search>, <start>, <end> ] }
```

- `string`: The string to search
- `search`: The substring to find
- `start` (optional): Index to start searching (default 0)
- `end` (optional): Index to stop searching (exclusive)

---

## ✅ Base Example 1 – Unicode-Safe Search

### 📥 Input Document

```json
{ "text": "नमस्ते दुनिया" }
```

### 📌 Expression

```json
{ "$indexOfCP": ["$text", "दुनिया"] }
```

### 📤 Output

```json
7
```

---

## ✅ Base Example 2 – ASCII Search

### 📥 Input Document

```json
{ "message": "Welcome to Canada" }
```

### 📌 Expression

```json
{ "$indexOfCP": ["$message", "Canada"] }
```

### 📤 Output

```json
11
```

---

## 🧱 Ecommerce Example – Check for Prefix in SKU Labels

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "prefixIndex": {
        "$indexOfCP": ["$items.sku", "PRE"]
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
    { "sku": "PRE-X-001", "name": "Cable" },
    { "sku": "X-PRE-002", "name": "Plug" },
    { "sku": "XYZ-003", "name": "Battery" }
  ]
}
```

### 📤 Output

```json
[
  { "prefixIndex": 0, "product": "Cable" },
  { "prefixIndex": 2, "product": "Plug" },
  { "prefixIndex": -1, "product": "Battery" }
]
```

---

## 🔧 Common Use Cases

- Locate substrings safely in multilingual data
- Index slicing or truncation
- Filtering records with presence check

---

## 🔗 Related Operators

- `$indexOfBytes`, `$substrCP`, `$split`, `$strLenCP`

---

## 🧠 Notes

- Use `$indexOfCP` when string may include multibyte characters (e.g. emojis, Hindi, Chinese).
- Returns `-1` if not found.

---
