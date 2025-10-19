# $floor

Rounds a numeric expression down to the nearest integer.

## Syntax

```json
{ "$floor": <numberExpression> }
```

## Example

### Input

```json
{ "measurement": 7.98 }
```

### Stage

```json
{ "$project": { "roundedDown": { "$floor": "$measurement" } } }
```

### Output

```json
{ "roundedDown": 7 }
```
