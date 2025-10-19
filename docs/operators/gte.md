# $gte

Returns `true` when the first expression evaluates greater than or equal to the second.

## Syntax

```json
{ "$gte": [ <expression1>, <expression2> ] }
```

## Example

### Input

```json
{ "score": 70 }
```

### Stage

```json
{ "$project": { "passed": { "$gte": ["$score", 70] } } }
```

### Output

```json
{ "passed": true }
```
