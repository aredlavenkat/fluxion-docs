# $and

Returns `true` only when every expression in the array evaluates to a truthy value.

## Syntax

```json
{ "$and": [ <expression1>, <expression2>, ... ] }
```

## Example

### Input

```json
{ "status": "active", "paymentReceived": true }
```

### Stage

```json
{
  "$project": {
    "canFulfill": {
      "$and": [
        { "$eq": ["$status", "active"] },
        "$paymentReceived"
      ]
    }
  }
}
```

### Output

```json
{ "canFulfill": true }
```
