# $indexOfBytes

The `$indexOfBytes` operator returns the byte index of the first occurrence of a substring in a string.

---

## 📌 Syntax

```json
{ "$indexOfBytes": [ <string>, <search>, <start>, <end> ] }
```

- `string`: The string to search
- `search`: Substring to find
- `start` (optional): Start index (inclusive)
- `end` (optional): End index (exclusive)

---

## ✅ Base Example 1 – Basic Search

### 📥 Input Document

```json
{ "message": "hello world" }
```

### 📌 Expression

```json
{ "$indexOfBytes": ["$message", "world"] }
```

### 📤 Output

```json
6
```

---

## ✅ Base Example 2 – Not Found

### 📥 Input Document

```json
{ "message": "hello world" }
```

### 📌 Expression

```json
{ "$indexOfBytes": ["$message", "planet"] }
```

### 📤 Output

```json
-1
```

---

## 🧱 Ecommerce Example – Check for Brand Code in SKU

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "brandIndex": {
        "$indexOfBytes": ["$items.sku", "BRND"]
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
    { "sku": "ELEC-BRND-X123", "name": "TV" },
    { "sku": "FASH-Y789", "name": "Watch" }
  ]
}
```

### 📤 Output

```json
[
  { "brandIndex": 5, "product": "TV" },
  { "brandIndex": -1, "product": "Watch" }
]
```

---

## 🔧 Common Use Cases

- Find substring location
- Validate presence or order of fields
- Index-based manipulation

---

## 🔗 Related Operators

- `$indexOfCP`, `$split`, `$substrBytes`, `$arrayElemAt`

---

## 🧠 Notes

- Operates on byte positions (not Unicode-safe).
- Use `$indexOfCP` for multi-byte character safety.

---
