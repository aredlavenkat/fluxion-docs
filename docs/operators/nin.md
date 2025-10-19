# $nin

Checks whether a value does **not** exist in an array.

## Syntax

```json
{ "$nin": [ <expression>, <arrayExpression> ] }
```

## Example

### Input

```json
{ "roles": ["viewer", "auditor"] }
```

### Stage

```json
{ "$project": { "isAdmin": { "$nin": ["admin", "$roles"] } } }
```

### Output

```json
{ "isAdmin": true }
```
