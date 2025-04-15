# $zip

The `$zip` operator **merges multiple arrays** element-wise into a single array of arrays.

---

## 📌 Syntax

```json
{
  "$zip": {
    "inputs": [ <array1>, <array2>, ... ],
    "useLongestLength": <bool>,  // optional
    "defaults": [ <default1>, <default2>, ... ] // optional
  }
}
```

- `inputs`: The arrays to zip together
- `useLongestLength`: If `true`, extends shorter arrays with `null` or `defaults`
- `defaults`: Values to use when an array runs out of elements

---

## ✅ Base Example – Merge Name and Age Lists

### 📥 Input Document

```json
{
  "names": ["Alice", "Bob", "Charlie"],
  "ages": [30, 25, 35]
}
```

### 📌 Expression

```json
{
  "$zip": {
    "inputs": ["$names", "$ages"]
  }
}
```

### 📤 Output

```json
[["Alice", 30], ["Bob", 25], ["Charlie", 35]]
```

---

## ✅ With Unequal Length Arrays and Defaults

### 📥 Input Document

```json
{
  "a": [1, 2],
  "b": ["x"]
}
```

### 📌 Expression

```json
{
  "$zip": {
    "inputs": ["$a", "$b"],
    "useLongestLength": true,
    "defaults": [null, "NA"]
  }
}
```

### 📤 Output

```json
[[1, "x"], [2, "NA"]]
```

---

## 🧱 Ecommerce Example – Merge Features and Availability

### 📌 Pipeline

```json
[
  { "$unwind": "$items" },
  {
    "$project": {
      "product": "$items.name",
      "featuresZipped": {
        "$zip": {
          "inputs": ["$items.features", "$items.available"],
          "useLongestLength": true,
          "defaults": ["Unknown", false]
        }
      }
    }
  }
]
```

### 📥 Input Document

```json
{
  "items": [
    {
      "name": "Table",
      "features": ["Wood", "Foldable"],
      "available": [true]
    }
  ]
}
```

### 📤 Output

```json
[
  {
    "product": "Table",
    "featuresZipped": [["Wood", true], ["Foldable", false]]
  }
]
```

---

## 🔧 Common Use Cases

- Merge parallel arrays for display
- Create structured tabular pairs
- Fill missing values dynamically

---

## 🔗 Related Operators

- `$map`, `$concatArrays`, `$reduce`, `$arrayElemAt`

---

## 🧠 Notes

- All arrays must be of equal length unless `useLongestLength` is `true`.
- Combine with `$map` to format zipped output.

---
