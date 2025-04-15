# $strcasecmp

The `$strcasecmp` operator performs a **case-insensitive comparison** of two strings and returns:

- `0` if they are equal (ignoring case)
- `1` if the first is greater
- `-1` if the second is greater

---

## 📌 Syntax

```json
{ "$strcasecmp": [ <string1>, <string2> ] }
```

---

## ✅ Base Example 1 – Equal Strings

### 📥 Input Document

```json
{ "input1": "hello", "input2": "HELLO" }
```

### 📌 Expression

```json
{ "$strcasecmp": ["$input1", "$input2"] }
```

### 📤 Output

```json
0
```

---

## ✅ Base Example 2 – Alphabetical Check

### 📥 Input Document

```json
{ "input1": "apple", "input2": "banana" }
```

### 📌 Expression

```json
{ "$strcasecmp": ["$input1", "$input2"] }
```

### 📤 Output

```json
-1
```

---

## 🧱 Ecommerce Example – Compare Category Codes for Sorting

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "isCategoryMatch": {
        "$eq": [
          { "$strcasecmp": ["$items.category", "FASHION"] },
          0
        ]
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
    { "name": "Jacket", "category": "Fashion" },
    { "name": "Speaker", "category": "Electronics" }
  ]
}
```

### 📤 Output

```json
[
  { "isCategoryMatch": true, "name": "Jacket" },
  { "isCategoryMatch": false, "name": "Speaker" }
]
```

---

## 🔧 Common Use Cases

- Case-insensitive comparisons
- Field normalization for filtering
- Use in `$project`, `$match`, or conditional logic

---

## 🔗 Related Operators

- `$eq`, `$toLower`, `$cmp`, `$cond`

---

## 🧠 Notes

- Both arguments must be strings.
- Returns numeric value based on case-insensitive lexicographical comparison.

---
