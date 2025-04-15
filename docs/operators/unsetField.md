# $unsetField

The `$unsetField` operator **removes a field** from a document dynamically.

---

## 📌 Syntax

```json
{
  "$unsetField": {
    "field": <fieldName>,
    "input": <document>
  }
}
```

- `field`: The field name to remove (string or expression)
- `input`: The document to modify

---

## ✅ Base Example – Remove Field

### 📥 Input Document

```json
{ "a": 1, "b": 2 }
```

### 📌 Expression

```json
{
  "$unsetField": {
    "field": "b",
    "input": "$$ROOT"
  }
}
```

### 📤 Output

```json
{ "a": 1 }
```

---

## ✅ Remove Dynamic Field

### 📥 Input Document

```json
{ "fields": { "x": 1, "y": 2 }, "removeKey": "y" }
```

### 📌 Expression

```json
{
  "$unsetField": {
    "field": "$removeKey",
    "input": "$fields"
  }
}
```

### 📤 Output

```json
{ "x": 1 }
```

---

## 🧱 Ecommerce Example – Remove Obsolete Attribute

### 📌 Pipeline

```json
[
  {
    "$project": {
      "product": "$name",
      "cleanAttributes": {
        "$unsetField": {
          "field": "legacyCode",
          "input": "$attributes"
        }
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "name": "Backpack",
  "attributes": {
    "color": "Blue",
    "legacyCode": "DEPRECATED"
  }
}
```

### 📤 Output

```json
{
  "product": "Backpack",
  "cleanAttributes": {
    "color": "Blue"
  }
}
```

---

## 🔧 Common Use Cases

- Clean up deprecated or legacy fields
- Sanitize sensitive data
- Apply dynamic field deletions

---

## 🔗 Related Operators

- `$setField`, `$getField`, `$objectToArray`, `$project`, `$unset`

---

## 🧠 Notes

- Returns a new document with the field removed
- Dynamic field names supported

---
