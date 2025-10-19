# $unset

Removes specified fields from documents.

---

## Syntax

```json
{ "$unset": "<fieldName>" }
```

```json
{ "$unset": ["<field1>", "<field2>", ...] }
```

- Accepts either a single string or an array of field names.
- To remove nested fields, specify the path (for example `"shipping.trackingNumber"`).

---

## ✅ Basic Example

### 📌 Stage

```json
{
  "$unset": "internal"
}
```

### 📥 Input

```json
{
  "product": "Phone",
  "internal": true
}
```

### 📤 Output

```json
{
  "product": "Phone"
}
```

---

## 🧱 Deep Nested Pipeline Usage (Ecommerce)

```json
[
  {
    "$unset": [
      "internal",
      "archived"
    ]
  }
]
```

### 📥 Input Document

```json
{
  "name": "Shoes",
  "archived": false,
  "internal": true
}
```

### 📤 Output Documents

```json
{
  "name": "Shoes"
}
```

---

## ➕ Supported Accumulators

None for this stage

---

## 🔧 Common Operators

None

---
