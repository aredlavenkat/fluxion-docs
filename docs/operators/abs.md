# $abs

Returns the absolute value of a numeric expression.

## Syntax

```json
{ "$abs": <numberExpression> }
```

## Example

### Input

```json
{ "delta": -42 }
```

### Stage

```json
{ "$project": { "deltaAbs": { "$abs": "$delta" } } }
```

### Output

```json
{ "deltaAbs": 42 }
```
