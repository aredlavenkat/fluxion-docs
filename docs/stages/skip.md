# $skip

Skips over a specified number of documents.

---

## ✅ Basic Example

### 📌 Stage

```json
{
  "$skip": 1
}
```

### 📥 Input

```json
[
  {
    "a": 1
  },
  {
    "a": 2
  },
  {
    "a": 3
  }
]
```

### 📤 Output

```json
[
  {
    "a": 2
  },
  {
    "a": 3
  }
]
```

---

## 🧱 Deep Nested Pipeline Usage (Ecommerce)

```json
[
  {
    "$sort": {
      "price": 1
    }
  },
  {
    "$skip": 2
  }
]
```

### 📥 Input Document

```json
[
  {
    "price": 10
  },
  {
    "price": 30
  },
  {
    "price": 20
  }
]
```

### 📤 Output Documents

```json
[
  {
    "price": 30
  }
]
```

---

## ➕ Supported Accumulators

None for this stage

---

## 🔧 Common Operators

None

---
