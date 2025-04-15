# $getField

The `$getField` operator retrieves the **value of a specified field** from a document, supporting both static and dynamic keys.

---

## 📌 Syntax

```json
{
  "$getField": {
    "field": <fieldName>,
    "input": <document>
  }
}
```

- `field`: Name of the field to access (string or expression)
- `input`: The document to read from (optional — defaults to current document)

---

## ✅ Base Example – Static Field Name

### 📥 Input Document

```json
{ "a": 10, "b": 20 }
```

### 📌 Expression

```json
{
  "$getField": {
    "field": "a"
  }
}
```

### 📤 Output

```json
10
```

---

## ✅ Dynamic Field from Variable

### 📥 Input Document

```json
{ "stats": { "score": 88 }, "target": "score" }
```

### 📌 Expression

```json
{
  "$getField": {
    "field": "$target",
    "input": "$stats"
  }
}
```

### 📤 Output

```json
88
```

---

## 🧱 Ecommerce Example – Access Variant Detail by Dynamic Key

### 📌 Pipeline

```json
[
  {
    "$project": {
      "variantColor": {
        "$getField": {
          "field": "$selectedColor",
          "input": "$variants"
        }
      },
      "product": "$name"
    }
  }
]
```

### 📥 Input Document

```json
{
  "name": "Backpack",
  "selectedColor": "blue",
  "variants": {
    "red": { "stock": 5 },
    "blue": { "stock": 12 }
  }
}
```

### 📤 Output

```json
{
  "product": "Backpack",
  "variantColor": { "stock": 12 }
}
```

---

## 🔧 Common Use Cases

- Dynamically access fields
- Fetch variable-key metadata
- Pair with `$map` or `$objectToArray` for dynamic access

---

## 🔗 Related Operators

- `$setField`, `$objectToArray`, `$getField`, `$mergeObjects`, `$literal`

---

## 🧠 Notes

- Avoids `$[dot]` notation for dynamic fields
- Returns `null` if the field does not exist

---
