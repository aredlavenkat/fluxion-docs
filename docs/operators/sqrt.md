# $sqrt

Computes the square root of a numeric expression.

## Syntax

```json
{ "$sqrt": <numberExpression> }
```

## Example

### Input

```json
{ "variance": 9 }
```

### Stage

```json
{ "$project": { "stdDev": { "$sqrt": "$variance" } } }
```

### Output

```json
{ "stdDev": 3 }
```
