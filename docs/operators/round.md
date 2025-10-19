# $round

Rounds a number to the nearest integer or to a specified decimal place.

## Syntax

```json
{ "$round": [ <numberExpression>, <placeExpression?> ] }
```

- Omit the second argument to round to the nearest integer.
- Supply a positive `place` to round to decimal places, or a negative value to round to tens, hundreds, etc.

## Example

### Input

```json
{ "score": 87.456 }
```

### Stage

```json
{ "$project": { "scoreRounded": { "$round": ["$score", 1] } } }
```

### Output

```json
{ "scoreRounded": 87.5 }
```
