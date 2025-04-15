# $indexOfArray

The `$indexOfArray` operator returns the **index of the first occurrence** of a value in an array.

---

## 📌 Syntax

```json
{ "$indexOfArray": [ <arrayExpression>, <searchValue>, <start>, <end> ] }
```

- `arrayExpression`: The array to search
- `searchValue`: The value to search for
- `start`: (optional) Index to start searching from
- `end`: (optional) Index to end searching (exclusive)

---

## ✅ Base Example – Find Index of Value

### 📥 Input Document

```json
{ "tags": ["sale", "new", "hot"] }
```

### 📌 Expression

```json
{ "$indexOfArray": ["$tags", "new"] }
```

### 📤 Output

```json
1
```

---

## ✅ Base Example – Value Not Found

### 📥 Input Document

```json
{ "tags": ["sale", "hot"] }
```

### 📌 Expression

```json
{ "$indexOfArray": ["$tags", "exclusive"] }
```

### 📤 Output

```json
-1
```

---

## 🧱 Ecommerce Example – Position of Top-Selling Product

### 📌 Pipeline

```json
[
  {
    "$project": {
      "productRank": {
        "$indexOfArray": ["$topProducts", "SKU-1002"]
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "topProducts": ["SKU-1001", "SKU-1002", "SKU-1003"]
}
```

### 📤 Output

```json
{ "productRank": 1 }
```

---

## 🔧 Common Use Cases

- Determine item positions
- Ranking systems
- Validation of list order

---

## 🔗 Related Operators

- `$arrayElemAt`, `$slice`, `$filter`, `$indexOfBytes`, `$indexOfCP`

---

## 🧠 Notes

- Returns `-1` if not found.
- Use with `$cond` to handle missing values gracefully.

---
