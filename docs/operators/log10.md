# $log10

Computes the base-10 logarithm of a numeric expression.

## Syntax

```json
{ "$log10": <numberExpression> }
```

## Example

### Input

```json
{ "latencyMs": 250 }
```

### Stage

```json
{ "$project": { "logLatency": { "$log10": "$latencyMs" } } }
```

### Output

```json
{ "logLatency": 2.3979400086720375 }
```
