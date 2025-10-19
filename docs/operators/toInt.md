# $toInt

Converts a value to a 32-bit integer, truncating toward zero.

## Syntax

```json
{ "$toInt": <expression> }
```

## Example

### Input

```json
{ "quantity": "15" }
```

### Stage

```json
{ "$project": { "quantity": { "$toInt": "$quantity" } } }
```

### Output

```json
{ "quantity": 15 }
```
