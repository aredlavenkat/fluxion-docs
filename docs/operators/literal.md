# $literal

The `$literal` operator returns a **constant value** without interpreting it as a field path or expression.

---

## 📌 Syntax

```json
{ "$literal": <value> }
```

- `<value>` can be a string, number, object, array, or other raw constant.

---

## ✅ Base Example – Return a Constant String

### 📥 Input Document

```json
{ "name": "Alice" }
```

### 📌 Expression

```json
{ "$literal": "Hello World" }
```

### 📤 Output

```json
"Hello World"
```

---

## ✅ Base Example – Return Constant Object

### 📥 Input Document

```json
{}
```

### 📌 Expression

```json
{
  "$literal": { "x": 1, "y": 2 }
}
```

### 📤 Output

```json
{ "x": 1, "y": 2 }
```

---

## 🧱 Ecommerce Example – Inject Static Tax Field

### 📌 Pipeline

```json
[
  {
    "$project": {
      "product": "$name",
      "taxRate": { "$literal": 0.15 }
    }
  }
]
```

### 📥 Input Document

```json
{ "name": "Smartphone", "price": 999 }
```

### 📤 Output

```json
{ "product": "Smartphone", "taxRate": 0.15 }
```

---

## 🔧 Common Use Cases

- Force a static value in dynamic documents
- Prevent evaluation of field paths or expressions
- Embed objects or arrays literally

---

## 🔗 Related Operators

- `$const` (MongoDB alternative), `$toString`, `$type`, `$mergeObjects`

---

## 🧠 Notes

- Useful inside `$project`, `$group`, `$addFields`
- Does not evaluate inner values — used exactly as written

---
