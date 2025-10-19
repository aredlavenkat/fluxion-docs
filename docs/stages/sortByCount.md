# $sortByCount

Groups documents by the supplied expression, counts each group, and sorts the results by count in descending order.

---

## Syntax

```json
{ "$sortByCount": <expression> }
```

- The expression can be a field path (`"$status"`), system variable, or computed expression.
- The output documents contain `_id` (group key) and `count`.

---

## Example

### Input

```json
[
  { "status": "OPEN" },
  { "status": "OPEN" },
  { "status": "CLOSED" },
  { "status": "FAILED" },
  { "status": "OPEN" }
]
```

### Stage

```json
{ "$sortByCount": "$status" }
```

### Output

```json
[
  { "_id": "OPEN", "count": 3 },
  { "_id": "CLOSED", "count": 1 },
  { "_id": "FAILED", "count": 1 }
]
```

---

## Tips

- When you already have grouped data, consider `$group` + `$sort` for more control over additional fields.
- Combine with `$match` beforehand to limit the documents being counted.
