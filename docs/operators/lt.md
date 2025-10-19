# $lt

Returns `true` when the first expression evaluates less than the second.

## Syntax

```json
{ "$lt": [ <expression1>, <expression2> ] }
```

## Example

### Input

```json
{ "temperature": 12 }
```

### Stage

```json
{ "$project": { "belowFreezing": { "$lt": ["$temperature", 0] } } }
```

### Output

```json
{ "belowFreezing": false }
```
