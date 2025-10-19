# $limit

Limits the number of documents passed to the next stage.

---

## Syntax

```json
{ "$limit": <positiveInteger> }
```

The integer caps how many documents are emitted downstream.

---

## ✅ Basic Example

### 📌 Stage

```json
{
  "$limit": 2
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
    "a": 1
  },
  {
    "a": 2
  }
]
```

---

## 🧱 Deep Nested Pipeline Usage (Ecommerce)

```json
[
  {
    "$match": {
      "status": "active"
    }
  },
  {
    "$limit": 1
  }
]
```

### 📥 Input Document

```json
[
  {
    "status": "active"
  },
  {
    "status": "inactive"
  }
]
```

### 📤 Output Documents

```json
[
  {
    "status": "active"
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
