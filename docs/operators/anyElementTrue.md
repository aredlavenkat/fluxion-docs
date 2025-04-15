# $anyElementTrue

The `$anyElementTrue` operator returns `true` if **any element** of the input array evaluates to a truthy value.

---

## 📌 Syntax

```json
{ "$anyElementTrue": [ <arrayExpression> ] }
```

---

## ✅ Base Example – Check If Any Truthy

### 📥 Input Document

```json
{ "flags": [false, false, true, false] }
```

### 📌 Expression

```json
{ "$anyElementTrue": ["$flags"] }
```

### 📤 Output

```json
true
```

---

## ✅ Base Example – All Falsy

### 📥 Input Document

```json
{ "checks": [false, 0, null] }
```

### 📌 Expression

```json
{ "$anyElementTrue": ["$checks"] }
```

### 📤 Output

```json
false
```

---

## 🧱 Ecommerce Example – Check for Available Features

### 📌 Pipeline

```json
[
  {
    "$project": {
      "hasFeature": {
        "$anyElementTrue": {
          "$map": {
            "input": "$features",
            "as": "f",
            "in": "$$f.enabled"
          }
        }
      },
      "name": 1
    }
  }
]
```

### 📥 Input Document

```json
{
  "name": "Chair",
  "features": [
    { "title": "Padded Seat", "enabled": false },
    { "title": "Adjustable Height", "enabled": true }
  ]
}
```

### 📤 Output

```json
{
  "name": "Chair",
  "hasFeature": true
}
```

---

## 🔧 Common Use Cases

- Detect at least one match
- Validate presence of flags or statuses
- Use after `$map` or `$filter`

---

## 🔗 Related Operators

- `$allElementsTrue`, `$map`, `$reduce`, `$cond`, `$or`

---

## 🧠 Notes

- Returns `false` for an empty array.
- Converts each element to boolean using JavaScript-like truthiness.

---
