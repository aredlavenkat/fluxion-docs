# $exp

Raises Euler’s number `e` to the power of the supplied numeric expression.

## Syntax

```json
{ "$exp": <numberExpression> }
```

## Example

### Input

```json
{ "growthRate": 1.2 }
```

### Stage

```json
{ "$project": { "continuousGrowth": { "$exp": "$growthRate" } } }
```

### Output

```json
{ "continuousGrowth": 3.3201169227365472 }
```
