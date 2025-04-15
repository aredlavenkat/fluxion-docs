# $trim

The `$trim` operator removes specified characters from the beginning and end of a string.

---

## 📌 Syntax

```json
{
  "$trim": {
    "input": <expression>,
    "chars": <charsToTrim>
  }
}
```

- `input`: The string to trim
- `chars`: The characters to remove (optional, defaults to whitespace)

---

## ✅ Base Example 1 – Trim Whitespace

### 📥 Input Document

```json
{ "username": "  alice123  " }
```

### 📌 Expression

```json
{
  "$trim": { "input": "$username" }
}
```

### 📤 Output

```json
"alice123"
```

---

## ✅ Base Example 2 – Trim Slashes from Path

### 📥 Input Document

```json
{ "path": "/products/shoes/" }
```

### 📌 Expression

```json
{
  "$trim": {
    "input": "$path",
    "chars": "/"
  }
}
```

### 📤 Output

```json
"products/shoes"
```

---

## 🧱 Ecommerce Example – Clean Category Labels

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "cleanCategory": {
        "$trim": {
          "input": "$items.category",
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
    { "name": "Sneakers", "category": "--footwear--" },
    { "name": "Watch", "category": "--accessories--" }
  ]
}
```

### 📤 Output

```json
[
  { "cleanCategory": "footwear", "product": "Sneakers" },
  { "cleanCategory": "accessories", "product": "Watch" }
]
```

---

## 🔧 Common Use Cases

- Strip extra characters (slashes, dashes, spaces)
- Normalize labels and IDs
- Clean user input

---

## 🔗 Related Operators

- `$ltrim`, `$rtrim`, `$toLower`, `$toUpper`

---

## 🧠 Notes

- If `chars` is omitted, whitespace is trimmed by default.
- Useful before comparisons and filtering.

---
