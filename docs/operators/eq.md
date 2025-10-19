# $eq

Returns `true` when the two expressions evaluate to the same value.

## Syntax

```json
{ "$eq": [ <expression1>, <expression2> ] }
```

## Example

### Input

```json
{ "plan": "premium" }
```

### Stage

```json
{ "$project": { "isPremium": { "$eq": ["$plan", "premium"] } } }
```

### Output

```json
{ "isPremium": true }
```
