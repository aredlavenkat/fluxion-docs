# $fill

`$fill` repairs missing or null fields using forward-fill (LOCF), linear interpolation, or explicit values. Pair it with `$densify` to produce continuous time-series streams before windowing.

---

## 📌 Syntax

```json
{
  "$fill": {
    "partitionBy": <expression>,          // optional
    "sortBy": { "<field>": 1 | -1 },
    "output": {
      "<field>": { "method": "locf" | "linear" },
      "<field>": { "value": <expression> }
    }
  }
}
```

- `partitionBy`: Expression evaluated per document; partitions are filled independently.
- `sortBy`: Required ordering within each partition. Only single-field sort is currently supported.
- `output`: Map of target fields to either a `method` (`locf` or `linear`) or a literal `value` expression.

---

## 🛒 Example – LOCF and Linear Interpolation

```json
{
  "$fill": {
    "partitionBy": "$deviceId",
    "sortBy": { "timestamp": 1 },
    "output": {
      "temperature": { "method": "linear" },
      "status": { "method": "locf" },
      "quality": { "value": "unknown" }
    }
  }
}
```

- `temperature` values are linearly interpolated between surrounding non-null readings.
- `status` carries forward the most recent non-null value.
- `quality` defaults to the literal string `"unknown"` whenever the field is missing.

---

### 📥 Input

```json
[
  {
    "deviceId": "A",
    "timestamp": 1,
    "temperature": 20.0,
    "status": "OK"
  },
  {
    "deviceId": "A",
    "timestamp": 2,
    "temperature": null
  },
  {
    "deviceId": "A",
    "timestamp": 3,
    "temperature": 24.0,
    "status": "WARN"
  }
]
```

### 📤 Output

```json
[
  {
    "deviceId": "A",
    "timestamp": 1,
    "temperature": 20.0,
    "status": "OK",
    "quality": "unknown"
  },
  {
    "deviceId": "A",
    "timestamp": 2,
    "temperature": 22.0,
    "status": "OK",
    "quality": "unknown"
  },
  {
    "deviceId": "A",
    "timestamp": 3,
    "temperature": 24.0,
    "status": "WARN",
    "quality": "unknown"
  }
]
```

---

## 💡 Tips

- The stage only overwrites fields that are `null` or missing; existing values are preserved.
- Linear interpolation requires both a previous and next numeric value; leading/trailing nulls remain unchanged.
- Combine with `$densify` to manufacture intermediate timestamps before filling.

---

## 🔗 Related Stages

- [`$densify`](./densify.md)
- [`$setWindowFields`](./setWindowFields.md)
- [`$group`](./group.md)

---
