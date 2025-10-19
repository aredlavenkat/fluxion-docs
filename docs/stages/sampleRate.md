# $sampleRate

Emits each document with the given probability. Useful for lightweight sampling or throttling without shuffling the pipeline.

---

## Syntax

```json
{ "$sampleRate": <rate> }
```

```json
{ "$sampleRate": { "rate": <rate>, "seed": <number> } }
```

- `rate` must be between `0` and `1`. A document is kept when a random draw is less than or equal to the rate.
- `seed` (optional) makes the sampling deterministic, which is handy for tests.

---

## Example

### Input

```json
[
  { "eventId": 1 },
  { "eventId": 2 },
  { "eventId": 3 },
  { "eventId": 4 }
]
```

### Stage

```json
{ "$sampleRate": { "rate": 0.5, "seed": 42 } }
```

### Output

```json
[
  { "eventId": 2 },
  { "eventId": 3 },
  { "eventId": 4 }
]
```

With the provided seed, events 2, 3, and 4 are retained. Without a seed, the sample varies across executions.

---

## Tips

- Apply `$match` or `$project` before sampling to reduce payload size.
- Use after `$sort` if you need sampling from a specific ordering of documents.
