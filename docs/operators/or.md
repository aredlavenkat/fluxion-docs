# $or

Returns `true` when any expression in the array evaluates to a truthy value.

## Syntax

```json
{ "$or": [ <expression1>, <expression2>, ... ] }
```

## Example

### Input

```json
{ "status": "pending", "manualOverride": true }
```

### Stage

```json
{
  "$project": {
    "shouldProcess": {
      "$or": [
        { "$eq": ["$status", "ready"] },
        "$manualOverride"
      ]
    }
  }
}
```

### Output

```json
{ "shouldProcess": true }
```
