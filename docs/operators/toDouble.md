# $toDouble

Converts a value to a double-precision floating point number.

## Syntax

```json
{ "$toDouble": <expression> }
```

## Example

### Input

```json
{ "price": "19.75" }
```

### Stage

```json
{ "$project": { "price": { "$toDouble": "$price" } } }
```

### Output

```json
{ "price": 19.75 }
```
