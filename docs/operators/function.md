# $function

The `$function` operator lets you define and execute **custom logic** using JavaScript-like syntax within a pipeline.

---

## 📌 Syntax

```json
{
  "$function": {
    "body": <functionCodeString>,
    "args": [ <arg1>, <arg2>, ... ],
    "lang": "js"
  }
}
```

- `body`: JavaScript function code as a string
- `args`: Array of arguments passed to the function
- `lang`: Must be `"js"` (SrotaX-safe context)

---

## ✅ Base Example – Sum Two Fields

### 📥 Input Document

```json
{ "a": 2, "b": 3 }
```

### 📌 Expression

```json
{
  "$function": {
    "body": "function(a, b) { return a + b; }",
    "args": ["$a", "$b"],
    "lang": "js"
  }
}
```

### 📤 Output

```json
5
```

---

## 🧱 Ecommerce Example – Calculate Dynamic Discount

### 📌 Pipeline

```json
[
  {
    "$project": {
      "product": "$name",
      "discountedPrice": {
        "$function": {
          "body": "function(p, d) {
            return Math.round(p - (p * d));
          }",
          "args": ["$price", "$discount"],
          "lang": "js"
        }
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "name": "Speaker",
  "price": 200,
  "discount": 0.15
}
```

### 📤 Output

```json
{
  "product": "Speaker",
  "discountedPrice": 170
}
```

---

## 🔧 Common Use Cases

- Advanced conditional logic
- Complex business formulas
- Processing values beyond built-in operators

---

## 🔗 Related Operators

- `$cond`, `$switch`, `$reduce`, `$map`

---

## 🧠 Notes

- SrotaX executes `$function` securely and deterministically.
- Avoid side effects; treat as pure functions.

---
