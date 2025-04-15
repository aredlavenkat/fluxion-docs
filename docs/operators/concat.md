# $concat

The `$concat` operator joins strings together into a single string.

---

## 📌 Syntax

```json
{ "$concat": [ <expression1>, <expression2>, ... ] }
```

All expressions must resolve to strings. If any operand resolves to `null`, the result is `null`.

---

## ✅ Base Example 1 – Full Name

### 📥 Input Document

```json
{ "firstName": "Jane", "lastName": "Doe" }
```

### 📌 Expression

```json
{ "$concat": ["$firstName", " ", "$lastName"] }
```

### 📤 Output

```json
"Jane Doe"
```

---

## ✅ Base Example 2 – File Path Builder

### 📥 Input Document

```json
{ "folder": "invoices", "file": "2025.pdf" }
```

### 📌 Expression

```json
{ "$concat": ["/data/", "$folder", "/", "$file"] }
```

### 📤 Output

```json
"/data/invoices/2025.pdf"
```

---

## 🧱 Ecommerce Example – Generate SKU Code

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "sku": {
        "$concat": [
          "$items.category", "-",
          "$items.brand", "-",
          "$items.productId"
        ]
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "items": [
    {
      "productId": "X123",
      "category": "ELEC",
      "brand": "SNY"
    },
    {
      "productId": "Y987",
      "category": "BOOK",
      "brand": "PNH"
    }
  ]
}
```

### 📤 Output

```json
[
  { "sku": "ELEC-SNY-X123" },
  { "sku": "BOOK-PNH-Y987" }
]
```

---

## 🔧 Common Use Cases

- Formatting names and addresses
- Constructing SKUs or file paths
- HTML or CSV generation

---

## 🔗 Related Operators

- `$toString`, `$substr`, `$toLower`, `$trim`
- `$project`, `$set`, `$map`

---

## 🧠 Notes

- Ensure all values are strings or cast them using `$toString`.
- Use `$cond` or `$ifNull` to avoid null operands.

---
