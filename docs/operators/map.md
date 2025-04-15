# $map

The `$map` operator applies a transformation expression to each element of an input array.

---

## 📌 Syntax

```json
{
  "$map": {
    "input": <arrayExpression>,
    "as": <variableName>,
    "in": <expression>
  }
}
```

- `input`: The array to iterate
- `as`: The name of the variable for each element
- `in`: The expression to evaluate for each element

---

## ✅ Base Example – Multiply All by 2

### 📥 Input Document

```json
{ "values": [1, 2, 3, 4] }
```

### 📌 Expression

```json
{
  "$map": {
    "input": "$values",
    "as": "num",
    "in": { "$multiply": ["$$num", 2] }
  }
}
```

### 📤 Output

```json
[2, 4, 6, 8]
```

---

## ✅ Base Example – Uppercase Items

### 📥 Input Document

```json
{ "tags": ["sale", "new", "hot"] }
```

### 📌 Expression

```json
{
  "$map": {
    "input": "$tags",
    "as": "tag",
    "in": { "$toUpper": "$$tag" }
  }
}
```

### 📤 Output

```json
["SALE", "NEW", "HOT"]
```

---

## 🧱 Ecommerce Example – Compute Total per Item

### 📌 Pipeline

```json
[
  {
    "$project": {
      "orderId": 1,
      "items": {
        "$map": {
          "input": "$items",
          "as": "item",
          "in": {
            "name": "$$item.name",
            "total": {
              "$multiply": ["$$item.price", "$$item.quantity"]
            }
          }
        }
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "orderId": 123,
  "items": [
    { "name": "Pen", "price": 2, "quantity": 10 },
    { "name": "Notebook", "price": 5, "quantity": 3 }
  ]
}
```

### 📤 Output

```json
{
  "orderId": 123,
  "items": [
    { "name": "Pen", "total": 20 },
    { "name": "Notebook", "total": 15 }
  ]
}
```

---

## 🔧 Common Use Cases

- Transform items in an array
- Compute fields per subdocument
- Prepare output for `$project` or `$group`

---

## 🔗 Related Operators

- `$filter`, `$reduce`, `$concatArrays`, `$arrayElemAt`

---

## 🧠 Notes

- Use `$$` prefix when referencing variables in `$map`.
- Input must be an array; otherwise, returns `null`.

---
