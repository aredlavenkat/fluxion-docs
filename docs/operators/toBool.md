# $toBool

Converts a value to a boolean using MongoDB’s truthiness rules.

## Syntax

```json
{ "$toBool": <expression> }
```

## Example

### Input

```json
{ "flag": "true" }
```

### Stage

```json
{ "$project": { "asBoolean": { "$toBool": "$flag" } } }
```

### Output

```json
{ "asBoolean": true }
```
