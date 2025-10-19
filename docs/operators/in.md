# $in

Checks whether a value exists in an array.

## Syntax

```json
{ "$in": [ <expression>, <arrayExpression> ] }
```

## Example

### Input

```json
{ "tags": ["beta", "feature-flag", "ios"] }
```

### Stage

```json
{ "$project": { "isIos": { "$in": ["ios", "$tags"] } } }
```

### Output

```json
{ "isIos": true }
```
