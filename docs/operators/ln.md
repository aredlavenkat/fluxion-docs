# $ln

Computes the natural logarithm (base `e`) of a numeric expression.

## Syntax

```json
{ "$ln": <numberExpression> }
```

## Example

### Input

```json
{ "value": 54.6 }
```

### Stage

```json
{ "$project": { "naturalLog": { "$ln": "$value" } } }
```

### Output

```json
{ "naturalLog": 3.999093 }
```
