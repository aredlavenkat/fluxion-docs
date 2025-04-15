# $substrCP

The `$substrCP` operator returns a substring from a string, measured in **UTF-8 code points (characters)**, starting at a specified character index with a specified number of characters.

---

## 📌 Syntax

```json
{ "$substrCP": [ <string>, <startChar>, <charLength> ] }
```

- `<string>`: The input string
- `<startChar>`: Character index to start from (0-based)
- `<charLength>`: Number of characters (code points) to return

---

## ✅ Base Example 1 – Extract First 4 Characters

### 📥 Input Document

```json
{ "title": "Notebook" }
```

### 📌 Expression

```json
{ "$substrCP": ["$title", 0, 4] }
```

### 📤 Output

```json
"Note"
```

---

## ✅ Base Example 2 – Multilingual Safe Slicing

### 📥 Input Document

```json
{ "label": "नमस्ते" }
```

### 📌 Expression

```json
{ "$substrCP": ["$label", 0, 3] }
```

### 📤 Output

```json
"नमस्"
```

---

## 🧱 Ecommerce Example – Shorten Product Names

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "shortName": {
        "$substrCP": ["$items.name", 0, 6]
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "items": [
    { "name": "Bluetooth Speaker" },
    { "name": "Wireless Keyboard" }
  ]
}
```

### 📤 Output

```json
[
  { "shortName": "Blueto" },
  { "shortName": "Wirele" }
]
```

---

## 🔧 Common Use Cases

- Multilingual-safe substring extraction
- Character-length-limited output
- Text formatting for display

---

## 🔗 Related Operators

- `$substr`, `$substrBytes`, `$split`, `$slice`, `$strLenCP`

---

## 🧠 Notes

- Use `$substrCP` over `$substr` or `$substrBytes` for Unicode strings.
- Avoid cutting emojis or multibyte characters with `$substrBytes`.

---
