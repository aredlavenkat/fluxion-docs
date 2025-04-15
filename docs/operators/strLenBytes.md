# $strLenBytes

The `$strLenBytes` operator returns the length of a string **in bytes** (not characters).

---

## 📌 Syntax

```json
{ "$strLenBytes": <expression> }
```

The expression must resolve to a string.

---

## ✅ Base Example 1 – ASCII String

### 📥 Input Document

```json
{ "name": "Laptop" }
```

### 📌 Expression

```json
{ "$strLenBytes": "$name" }
```

### 📤 Output

```json
6
```

---

## ✅ Base Example 2 – Unicode Characters

### 📥 Input Document

```json
{ "emoji": "😊" }
```

### 📌 Expression

```json
{ "$strLenBytes": "$emoji" }
```

### 📤 Output

```json
4
```

> Emoji and many Unicode characters use multiple bytes.

---

## 🧱 Ecommerce Example – Calculate Byte Size of Labels

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "label": "$items.name",
      "byteLength": {
        "$strLenBytes": "$items.name"
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "items": [
    { "name": "Bag" },
    { "name": "नमस्ते" },
    { "name": "Keyboard" }
  ]
}
```

### 📤 Output

```json
[
  { "label": "Bag", "byteLength": 3 },
  { "label": "नमस्ते", "byteLength": 18 },
  { "label": "Keyboard", "byteLength": 8 }
]
```

---

## 🔧 Common Use Cases

- Prepare size-limited exports
- Analyze string size for storage
- Truncate safely with `$substrBytes`

---

## 🔗 Related Operators

- `$strLenCP`, `$substrBytes`, `$toString`, `$split`

---

## 🧠 Notes

- Byte length ≠ character count in UTF-8.
- Use `$strLenCP` for actual character count.

---
