# $sum

Adds together numeric values and returns their total.

## Syntax

```json
{ "$sum": <arrayExpression> }
```

## Example

### Input

```json
{ "lineItems": [29.99, 14.5, 9.0] }
```

### Stage

```json
{ "$project": { "orderTotal": { "$sum": "$lineItems" } } }
```

### Output

```json
{ "orderTotal": 53.49 }
```
