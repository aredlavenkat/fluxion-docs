# $setField

The `$setField` operator allows **adding, updating, or replacing a field** in a document.

---

## 📌 Syntax

```json
{
  "$setField": {
    "field": <fieldName>,
    "input": <document>,
    "value": <newValue>
  }
}
```

- `field`: The field to set
- `input`: The input document
- `value`: The new value to assign

---

## ✅ Base Example – Set Field Value

### 📥 Input Document

```json
{ "a": 1 }
```

### 📌 Expression

```json
{
  "$setField": {
    "field": "b",
    "input": "$$ROOT",
    "value": 2
  }
}
```

### 📤 Output

```json
{ "a": 1, "b": 2 }
```

---

## ✅ Overwrite Existing Field

### 📥 Input Document

```json
{ "a": 10 }
```

### 📌 Expression

```json
{
  "$setField": {
    "field": "a",
    "input": "$$ROOT",
    "value": 99
  }
}
```

### 📤 Output

```json
{ "a": 99 }
```

---

## 🧱 Ecommerce Example – Add Rating to Product Attribute

### 📌 Pipeline

```json
[
  {
    "$project": {
      "product": "$name",
      "updatedAttributes": {
        "$setField": {
          "field": "rating",
          "input": "$attributes",
          "value": 4.8
        }
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "name": "Watch",
  "attributes": {
    "color": "Black",
    "style": "Sport"
  }
}
```

### 📤 Output

```json
{
  "product": "Watch",
  "updatedAttributes": {
    "color": "Black",
    "style": "Sport",
    "rating": 4.8
  }
}
```

---

## 🔧 Common Use Cases

- Modify nested subdocuments
- Add calculated fields
- Create documents with dynamic key-value pairs

---

## 🔗 Related Operators

- `$getField`, `$mergeObjects`, `$unsetField`, `$objectToArray`

---

## 🧠 Notes

- Allows modifying deeply nested fields with full control
- Supports dynamic field names using expressions

---
