# $toString

Converts a value to its string representation.

## Syntax

```json
{ "$toString": <expression> }
```

## Example

### Input

```json
{ "orderId": 12345 }
```

### Stage

```json
{ "$project": { "orderId": { "$toString": "$orderId" } } }
```

### Output

```json
{ "orderId": "12345" }
```
