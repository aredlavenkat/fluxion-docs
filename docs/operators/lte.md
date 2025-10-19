# $lte

Returns `true` when the first expression evaluates less than or equal to the second.

## Syntax

```json
{ "$lte": [ <expression1>, <expression2> ] }
```

## Example

### Input

```json
{ "inventory": 5 }
```

### Stage

```json
{ "$project": { "lowStock": { "$lte": ["$inventory", 5] } } }
```

### Output

```json
{ "lowStock": true }
```
