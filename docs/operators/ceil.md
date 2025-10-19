# $ceil

Rounds a numeric expression up to the nearest integer.

## Syntax

```json
{ "$ceil": <numberExpression> }
```

## Example

### Input

```json
{ "measurement": 12.01 }
```

### Stage

```json
{ "$project": { "roundedUp": { "$ceil": "$measurement" } } }
```

### Output

```json
{ "roundedUp": 13 }
```
