# $avg

Calculates the arithmetic mean of the supplied numeric values.

## Syntax

```json
{ "$avg": <arrayExpression> }
```

## Example

### Input

```json
{ "scores": [90, 85, 95] }
```

### Stage

```json
{ "$project": { "averageScore": { "$avg": "$scores" } } }
```

### Output

```json
{ "averageScore": 90 }
```
