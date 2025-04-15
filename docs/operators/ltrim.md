# $ltrim

The `$ltrim` operator removes specified characters from the **beginning (left)** of a string.

---

## 📌 Syntax

```json
{
  "$ltrim": {
    "input": <expression>,
    "chars": <charsToTrim>
  }
}
```

---

## ✅ Base Example 1 – Trim Leading Whitespace

### 📥 Input Document

```json
{ "title": "   Notebook" }
```

### 📌 Expression

```json
{
  "$ltrim": { "input": "$title" }
}
```

### 📤 Output

```json
"Notebook"
```

---

## ✅ Base Example 2 – Trim Leading Zeros from ID

### 📥 Input Document

```json
{ "id": "000123" }
```

### 📌 Expression

```json
{
  "$ltrim": {
    "input": "$id",
    "chars": "0"
  }
}
```

### 📤 Output

```json
"123"
```

---

## 🧱 Ecommerce Example – Clean Item Codes

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "code": {
        "$ltrim": {
          "input": "$items.code",
          "chars": "X"
        }
      },
      "name": "$items.name"
    }
  }
]
```

### 📥 Input Document

```json
{
  "items": [
    { "code": "XXA100", "name": "T-shirt" },
    { "code": "XXB200", "name": "Bag" }
  ]
}
```

### 📤 Output

```json
[
  { "code": "A100", "name": "T-shirt" },
  { "code": "B200", "name": "Bag" }
]
```

---

## 🔧 Common Use Cases

- Trim leading symbols, padding characters
- Normalize document IDs or SKUs
- Clean up data during import

---

## 🔗 Related Operators

- `$rtrim`, `$trim`, `$toLower`, `$substr`

---

## 🧠 Notes

- If `chars` is not specified, whitespace is removed.
- For both sides, use `$trim`.

---
