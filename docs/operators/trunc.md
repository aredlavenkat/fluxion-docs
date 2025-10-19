# $trunc

Truncates a number toward zero to a specified precision (defaults to whole numbers).

## Syntax

```json
{ "$trunc": [ <numberExpression>, <placeExpression?> ] }
```

- Omit `place` to drop everything after the decimal.
- Use a positive place to keep decimal digits, or a negative place to truncate to tens, hundreds, etc.

## Example

### Input

```json
{ "exchangeRate": 74.987 }
```

### Stage

```json
{ "$project": { "roundedDown": { "$trunc": ["$exchangeRate", 2] } } }
```

### Output

```json
{ "roundedDown": 74.98 }
```
