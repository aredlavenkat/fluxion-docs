# $type

The `$type` operator returns a **string that identifies the BSON type** of the input value.

---

## 📌 Syntax

```json
{ "$type": <expression> }
```

Returns values such as `"string"`, `"int"`, `"double"`, `"date"`, `"bool"`, `"object"`, `"array"`, etc.

---

## ✅ Base Example – Detect Type

### 📥 Input Document

```json
{ "price": 99.99 }
```

### 📌 Expression

```json
{ "$type": "$price" }
```

### 📤 Output

```json
"double"
```

---

## ✅ Type of Nested Field

### 📥 Input Document

```json
{ "metadata": { "tags": ["new", "hot"] } }
```

### 📌 Expression

```json
{ "$type": "$metadata.tags" }
```

### 📤 Output

```json
"array"
```

---

## 🧱 Ecommerce Example – Validate Price Type Before Conversion

### 📌 Pipeline

```json
[
  {
    "$project": {
      "product": "$name",
      "priceType": { "$type": "$price" },
      "price": 1
    }
  }
]
```

### 📥 Input Documents

```json
[
  { "name": "Phone", "price": "699" },
  { "name": "Laptop", "price": 1499.99 }
]
```

### 📤 Output

```json
[
  { "product": "Phone", "priceType": "string", "price": "699" },
  { "product": "Laptop", "priceType": "double", "price": 1499.99 }
]
```

---

## 🔧 Common Use Cases

- Debug complex fields before transformation
- Conditional conversion or validation
- Auditing unknown schemas

---

## 🔗 Related Operators

- `$convert`, `$ifNull`, `$cond`, `$literal`

---

## 🧠 Notes

- Useful when working with inconsistent or dynamic data sources
- Helps avoid runtime conversion errors

---
