# 📘 Usage Guide

This guide walks you through using Fluxion pipelines for common aggregation tasks.

---

## ✅ 1. Creating a Pipeline

A pipeline is an array of stages:

```json
[
  { "$match": { "status": "active" } },
  { "$group": { "_id": "$category", "total": { "$sum": "$amount" } } }
]
```

---

## ✅ 2. Running a Pipeline

You can run it via the `MongoPipelineExecutor`:

```python
from aggregator.executor import MongoPipelineExecutor

executor = MongoPipelineExecutor()
result = executor.execute(documents, pipeline)
```

---

## ✅ 3. Example with Variables

Fluxion supports variables like `$$ROOT`, `$$CURRENT`, `$$NOW`:

```json
{
  "$addFields": {
    "createdAt": "$$NOW",
    "copy": "$$ROOT"
  }
}
```

---

## ✅ 4. Advanced Nesting

You can nest operators like:

```json
{
  "$addFields": {
    "final_score": {
      "$reduce": {
        "input": "$scores",
        "initialValue": 0,
        "in": { "$add": ["$$value", "$$this"] }
      }
    }
  }
}
```

---

For more examples, see the [Examples Gallery](examples/exampleSet1.md).
