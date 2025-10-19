# $not

Returns the logical negation of the supplied expression.

## Syntax

```json
{ "$not": [ <expression> ] }
```

## Example

### Input

```json
{ "isDeleted": false }
```

### Stage

```json
{ "$project": { "isActive": { "$not": ["$isDeleted"] } } }
```

### Output

```json
{ "isActive": true }
```
