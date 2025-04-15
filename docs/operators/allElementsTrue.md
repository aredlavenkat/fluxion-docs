# $allElementsTrue

The `$allElementsTrue` operator returns `true` if **every element** in the input array evaluates to a truthy value.

---

## 📌 Syntax

```json
{ "$allElementsTrue": [ <arrayExpression> ] }
```

---

## ✅ Base Example – All True

### 📥 Input Document

```json
{ "results": [1, true, "ok"] }
```

### 📌 Expression

```json
{ "$allElementsTrue": ["$results"] }
```

### 📤 Output

```json
true
```

---

## ✅ Base Example – Contains Falsy

### 📥 Input Document

```json
{ "values": [1, 0, true] }
```

### 📌 Expression

```json
{ "$allElementsTrue": ["$values"] }
```

### 📤 Output

```json
false
```

---

## 🧱 Ecommerce Example – All Features Enabled

### 📌 Pipeline

```json
[
  {
    "$project": {
      "name": 1,
      "allEnabled": {
        "$allElementsTrue": {
          "$map": {
            "input": "$features",
            "as": "f",
            "in": "$$f.enabled"
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
  "name": "Standing Desk",
  "features": [
    { "title": "Electric Height", "enabled": true },
    { "title": "Memory Presets", "enabled": true }
  ]
}
```

### 📤 Output

```json
{
  "name": "Standing Desk",
  "allEnabled": true
}
```

---

## 🔧 Common Use Cases

- Verify that every value meets a condition
- Check that all features/options are enabled
- Chain with `$map`, `$filter`

---

## 🔗 Related Operators

- `$anyElementTrue`, `$reduce`, `$cond`, `$and`

---

## 🧠 Notes

- Returns `true` for an empty array
- Each element is evaluated using JavaScript-style truthiness

---
