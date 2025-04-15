# Basic Gallery

This gallery showcases advanced use cases, deep-nested stages, accumulators, and expression operators for Fluxion's aggregation engine.

---

## 🧩 Nested $facet + $bucket + $group + $project

```json
[
  { "$match": { "orderDate": { "$gte": "2024-01-01" } } },
  { "$unwind": "$items" },
  { "$facet": {
    "byCategory": [
      { "$group": {
          "_id": "$items.category",
          "totalSales": { "$sum": "$items.price" },
          "averagePrice": { "$avg": "$items.price" },
          "count": { "$sum": 1 }
      }},
      { "$project": {
          "category": "$_id",
          "totalSales": 1,
          "averagePrice": 1,
          "count": 1,
          "_id": 0
      }}
    ],
    "priceBuckets": [
      { "$bucket": {
          "groupBy": "$items.price",
          "boundaries": [0, 100, 500, 1000],
          "default": "Other",
          "output": {
            "itemCount": { "$sum": 1 },
            "avgPrice": { "$avg": "$items.price" }
          }
      }}
    ]
  }}
]
```

---

## 🧠 All Major Accumulators in $group

```json
[
  {
    "$group": {
      "_id": "$category",
      "total": { "$sum": "$amount" },
      "avg": { "$avg": "$amount" },
      "min": { "$min": "$amount" },
      "max": { "$max": "$amount" },
      "first": { "$first": "$amount" },
      "last": { "$last": "$amount" },
      "uniqueBrands": { "$addToSet": "$brand" },
      "allItems": { "$push": "$item" }
    }
  }
]
```

---

## 🔧 Operator Showcase

Each example uses a single operator.

### $map

```json
{
  "$map": {
    "input": "$items",
    "as": "item",
    "in": { "total": { "$multiply": ["$$item.price", "$$item.qty"] } }
  }
}
```

### $filter

```json
{
  "$filter": {
    "input": "$items",
    "as": "item",
    "cond": { "$gt": ["$$item.qty", 1] }
  }
}
```

### $reduce

```json
{
  "$reduce": {
    "input": "$items",
    "initialValue": 0,
    "in": { "$add": ["$$value", "$$this.price"] }
  }
}
```

### $cond

```json
{
  "$cond": {
    "if": { "$gte": ["$price", 1000] },
    "then": "expensive",
    "else": "cheap"
  }
}
```

### $switch

```json
{
  "$switch": {
    "branches": [
      { "case": { "$lt": ["$price", 100] }, "then": "low" },
      { "case": { "$lt": ["$price", 500] }, "then": "mid" }
    ],
    "default": "high"
  }
}
```

---
