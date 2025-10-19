# $gt

Returns `true` when the first expression evaluates greater than the second.

## Syntax

```json
{ "$gt": [ <expression1>, <expression2> ] }
```

## Example

### Input

```json
{ "temperature": 31 }
```

### Stage

```json
{ "$project": { "isHot": { "$gt": ["$temperature", 30] } } }
```

### Output

```json
{ "isHot": true }
```
