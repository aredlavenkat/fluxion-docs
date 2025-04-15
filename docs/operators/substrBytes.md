# $substrBytes

The `$substrBytes` operator returns a substring from a string, measured in **bytes**, starting at a specified index with a specified byte length.

---

## 📌 Syntax

```json
{ "$substrBytes": [ <string>, <startByte>, <byteLength> ] }
```

- `<string>`: The input string
- `<startByte>`: Byte offset (starting from 0)
- `<byteLength>`: Number of bytes to return

---

## ✅ Base Example 1 – Extract First 3 Bytes (ASCII safe)

### 📥 Input Document

```json
{ "code": "ABC123" }
```

### 📌 Expression

```json
{ "$substrBytes": ["$code", 0, 3] }
```

### 📤 Output

```json
"ABC"
```

---

## ✅ Base Example 2 – Byte Truncation of File Name

### 📥 Input Document

```json
{ "filename": "product.csv" }
```

### 📌 Expression

```json
{ "$substrBytes": ["$filename", 0, 7] }
```

### 📤 Output

```json
"product"
```

---

## 🧱 Ecommerce Example – Shorten Product Codes for Export

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "shortCode": {
        "$substrBytes": ["$items.sku", 0, 5]
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
    { "sku": "CODE99999", "name": "Bag" },
    { "sku": "TOOL88888", "name": "Drill" }
  ]
}
```

### 📤 Output

```json
[
  { "shortCode": "CODE9", "product": "Bag" },
  { "shortCode": "TOOL8", "product": "Drill" }
]
```

---

## 🔧 Common Use Cases

- Export byte-limited strings
- Handling legacy encodings (e.g., ASCII)
- Efficient slicing in fixed-byte-width systems

---

## 🔗 Related Operators

- `$substr`, `$substrCP`, `$slice`, `$split`, `$indexOfBytes`

---

## 🧠 Notes

- Works best with ASCII or single-byte encodings.
- May truncate multi-byte Unicode characters incorrectly — for Unicode-safe use `$substrCP`.

---
