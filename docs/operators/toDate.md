# $toDate

Converts a value to a BSON `Date`. Strings are parsed using ISO-8601 formats unless options specify otherwise.

## Syntax

```json
{ "$toDate": <expression> }
```

## Example

### Input

```json
{ "createdAt": "2025-02-01T12:30:00Z" }
```

### Stage

```json
{ "$project": { "createdAt": { "$toDate": "$createdAt" } } }
```

### Output

```json
{ "createdAt": { "$date": "2025-02-01T12:30:00Z" } }
```
