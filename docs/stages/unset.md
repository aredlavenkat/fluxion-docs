# $unset

Removes specified fields from documents.

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
