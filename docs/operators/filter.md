# $filter

The `$filter` operator selects elements from an array that satisfy a condition.

---

## 📌 Syntax

```json
{
  "$filter": {
    "input": <arrayExpression>,
    "as": <variableName>,
    "cond": <booleanExpression>
  }
}
```

- `input`: The array to filter
- `as`: The variable name for each item
- `cond`: The condition that must be `true` to include the item

---

## ✅ Base Example – Filter Even Numbers

### 📥 Input Document

```json
{ "nums": [1, 2, 3, 4, 5, 6] }
```

### 📌 Expression

```json
{
  "$filter": {
    "input": "$nums",
    "as": "num",
    "cond": { "$eq": [{ "$mod": ["$$num", 2] }, 0] }
  }
}
```

### 📤 Output

```json
[2, 4, 6]
```

---

## ✅ Base Example – Remove Empty Strings

### 📥 Input Document

```json
{ "tags": ["", "sale", null, "new", ""] }
```

### 📌 Expression

```json
{
  "$filter": {
    "input": "$tags",
    "as": "tag",
    "cond": { "$and": [{ "$ne": ["$$tag", ""] }, { "$ne": ["$$tag", null] }] }
  }
}
```

### 📤 Output

```json
["sale", "new"]
```

---

## 🧱 Ecommerce Example – Filter High-Value Items

### 📌 Pipeline

```json
[
  {
    "$project": {
      "items": {
        "$filter": {
          "input": "$items",
          "as": "item",
          "cond": { "$gt": ["$$item.price", 100] }
        }
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "items": [
    { "name": "Book", "price": 40 },
    { "name": "Tablet", "price": 300 },
    { "name": "Monitor", "price": 150 }
  ]
}
```

### 📤 Output

```json
{
  "items": [
    { "name": "Tablet", "price": 300 },
    { "name": "Monitor", "price": 150 }
  ]
}
```

---

## 🔧 Common Use Cases

- Select items matching a filter
- Clean arrays of null/empty/invalid entries
- Apply logic to sub-documents or features

---

## 🔗 Related Operators

- `$map`, `$reduce`, `$anyElementTrue`, `$allElementsTrue`

---

## 🧠 Notes

- Use `$$` to access loop variables.
- Useful for filtering ecommerce features, specs, or product tags.

---
