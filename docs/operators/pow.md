# $pow

Raises a number to the specified exponent.

## Syntax

```json
{ "$pow": [ <baseExpression>, <exponentExpression> ] }
```

## Example

### Input

```json
{ "radius": 3 }
```

### Stage

```json
{ "$project": { "areaFactor": { "$pow": ["$radius", 2] } } }
```

### Output

```json
{ "areaFactor": 9 }
```
