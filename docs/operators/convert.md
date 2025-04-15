# $convert

The `$convert` operator explicitly converts a value to a specified type, optionally handling `onNull` and `onError` fallbacks.

---

## 📌 Syntax

```json
{
  "$convert": {
    "input": <expression>,
    "to": <type>,
    "onError": <fallback>,     // optional
    "onNull": <fallback>       // optional
  }
}
```

- `input`: The value to convert
- `to`: Target type (`string`, `int`, `double`, `bool`, `date`, `objectId`)
- `onError`: Value to return if conversion fails
- `onNull`: Value to return if input is null or missing

---

## ✅ Base Example – Convert String to Integer

### 📥 Input Document

```json
{ "price": "100" }
```

### 📌 Expression

```json
{
  "$convert": {
    "input": "$price",
    "to": "int"
  }
}
```

### 📤 Output

```json
100
```

---

## ✅ With `onNull` and `onError`

### 📥 Input Document

```json
{ "qty": null }
```

### 📌 Expression

```json
{
  "$convert": {
    "input": "$qty",
    "to": "int",
    "onNull": 0,
    "onError": -1
  }
}
```

### 📤 Output

```json
0
```

---

## 🧱 Ecommerce Example – Convert Product Launch Date

### 📌 Pipeline

```json
[
  {
    "$project": {
      "name": 1,
      "launchDate": {
        "$convert": {
          "input": "$releaseDateStr",
          "to": "date",
          "onError": "$$CLUSTER_TIME"
        }
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "name": "Tablet",
  "releaseDateStr": "2024-03-01"
}
```

### 📤 Output

```json
{
  "name": "Tablet",
  "launchDate": "2024-03-01T00:00:00Z"
}
```

---

## 🔧 Common Use Cases

- Format validation and casting
- Prevent failures during transformation
- Normalize mixed-type inputs

---

## 🔗 Related Operators

- `$toString`, `$type`, `$literal`, `$ifNull`

---

## 🧠 Notes

- Conversion errors trigger `onError`, not pipeline failure
- Use `$type` to debug values before converting

---
