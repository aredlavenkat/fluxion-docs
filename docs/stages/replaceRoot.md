# $replaceRoot

The `$replaceRoot` stage replaces the entire input document with a specified subdocument.

---

## 📌 Syntax

```json
{ "$replaceRoot": { "newRoot": <expression> } }
```

---

## ✅ Base Example – Replace Root with Embedded Customer Object

### 📥 Input Document

```json
{
  "_id": 1,
  "orderId": 22,
  "customer": {
    "name": "Alice",
    "email": "alice@example.com"
  },
  "items": [{ "name": "Book" }]
}
```

### 📌 Stage

```json
{ "$replaceRoot": { "newRoot": "$customer" } }
```

### 📤 Output Document

```json
{
  "name": "Alice",
  "email": "alice@example.com"
}
```

---

## 🔧 Common Operators

- `$mergeObjects`, `$ifNull`, `$project`

---

## 🔗 Related

- `$replaceWith`, `$project`, `$unwind`

---
