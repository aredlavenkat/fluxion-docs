# $ne

Returns `true` when the two expressions evaluate to different values.

## Syntax

```json
{ "$ne": [ <expression1>, <expression2> ] }
```

## Example

### Input

```json
{ "status": "pending" }
```

### Stage

```json
{ "$project": { "isFinal": { "$ne": ["$status", "complete"] } } }
```

### Output

```json
{ "isFinal": true }
```
