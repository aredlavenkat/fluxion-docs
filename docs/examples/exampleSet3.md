# Deep Pipeline Scenarios

Built from `fluxion-core/src/test/java/.../DeepPipelineTest.java`. Each scenario
demonstrates multi-stage pipelines with nested arrays, reducers, windowing, and
conditional logic.

---

## How to run the scenarios

```bash
mvn -pl fluxion-core test -Dtest=DeepPipelineTest
```

Or run the entire module:

```bash
mvn -pl fluxion-core -am test
```

---

## Scenario summary

| Test | Highlights |
| --- | --- |
| `test_deep_pipeline` | `$reduce`, `$map`, `$match`, `$project` – calculate total payment and total item cost. |
| `test_deep_nested_pipeline` | `$filter`, `$map`, `$reduce`, `$dateAdd`, `$cond` – filter high-quantity items and compute order status. |
| `test_deep_nested_pipeline_two` | `$map`, `$reduce`, `$switch`, `$project` – flatten nested orders and classify status. |
| `test_degree_pipeline` (if present) | `$group`, `$unwind`, `$setWindowFields` – windowed aggregations across nested data. |
| `test_deep_pipeline_with_variables` | Demonstrates `$$ROOT`, `$$CURRENT`, custom variables. |

Below are the two most illustrative scenarios. Clone them when you need complex
transformations or nested-array manipulations.

---

### 1. `test_deep_pipeline`

```json
Input:
[
  {
    "order_id": "A1001",
    "payment": {
      "transactions": [
        {"txn_id": "T1", "amount": 120},
        {"txn_id": "T2", "amount": 30}
      ]
    },
    "items": [
      {"price": 100, "quantity": 2},
      {"price": 50, "quantity": 1}
    ],
    "created_at": "2023-04-15T10:00:00"
  }
]

Pipeline:
[
  {
    "$addFields": {
      "total_payment": {
        "$reduce": {
          "input": "$payment.transactions",
          "initialValue": 0,
          "in": { "$add": ["$$value", "$$this.amount"] }
        }
      },
      "total_item_cost": {
        "$reduce": {
          "input": {
            "$map": {
              "input": "$items",
              "as": "item",
              "in": { "$multiply": ["$$item.price", "$$item.quantity"] }
            }
          },
          "initialValue": 0,
          "in": { "$add": ["$$value", "$$this"] }
        }
      }
    }
  },
  { "$match": { "total_payment": { "$gte": 100 } } },
  { "$project": { "order_id": 1, "total_payment": 1, "total_item_cost": 1, "_id": 0 } }
]

Output:
[
  {
    "order_id": "A1001",
    "total_payment": 150,
    "total_item_cost": 250
  }
]
```

**Key ideas**

- `$reduce` is ideal when summing nested array values.
- Compose `$map` inside `$reduce` to transform data before accumulation.

---

### 2. `test_deep_nested_pipeline`

```json
Input:
[
  {
    "order_id": "ORD123",
    "order_date": "2024-04-01T00:00:00",
    "items": [
      {"item": "Pen", "quantity": 0, "unit_price": 1.5},
      {"item": "Notebook", "quantity": 10, "unit_price": 5.0},
      {"item": "Bag", "quantity": 3, "unit_price": 20.0}
    ]
  }
]

Pipeline:
[
  {
    "$project": {
      "order_id": 1,
      "filtered_items": {
        "$filter": {
          "input": "$items",
          "as": "item",
          "cond": { "$gt": ["$$item.quantity", 1] }
        }
      },
      "total_amount": {
        "$reduce": {
          "input": {
            "$map": {
              "input": {
                "$filter": {
                  "input": "$items",
                  "as": "item",
                  "cond": { "$gt": ["$$item.quantity", 1] }
                }
              },
              "as": "item",
              "in": {
                "$multiply": ["$$item.quantity", "$$item.unit_price"]
              }
            }
          },
          "initialValue": 0,
          "in": { "$add": ["$$value", "$$this"] }
        }
      },
      "expected_delivery": {
        "$dateAdd": {
          "startDate": "$order_date",
          "unit": "day",
          "amount": 7
        }
      }
    }
  },
  {
    "$addFields": {
      "order_status": {
        "$cond": {
          "if": { "$gte": ["$total_amount", 100] },
          "then": "approved",
          "else": "pending"
        }
      }
    }
  }
]

Output:
[
  {
    "order_id": "ORD123",
    "filtered_items": [
      { "item": "Notebook", "quantity": 10, "unit_price": 5.0 },
      { "item": "Bag", "quantity": 3, "unit_price": 20.0 }
    ],
    "total_amount": 170,
    "expected_delivery": "2024-04-08T00:00:00",
    "order_status": "approved"
  }
]
```

**Key ideas**

- Chain `$filter` → `$map` → `$reduce` to process nested arrays.
- `$dateAdd` and `$cond` are handy for scheduling and approval flows.

---

## Extending the suite

- Add tests covering system variables (`$$ROOT`, `$$CURRENT`, `$$REMOVE`).
- Include `$setWindowFields` for moving averages or rank calculations.
- Test negative paths (no matching items, zero transactions) to validate
  default behaviour.

These scenarios are ready-made templates for complex analytical pipelines—drop
them into your services or expand them into broader regression suites.
          {"name": "Phone", "price": 500, "qty": 1},
          {"name": "Charger", "qty": 1}
        ]
      }
    ]
  }
]
```

### 📌 Pipeline

See full implementation in source – uses:
- `$map`
- `$reduce`
- `$switch`
- `$ifNull`

### 📤 Output

```json
[
  {
    "user_id": "U123",
    "orders": [
      { "order_id": "O100", "order_total": 1104, "order_status": "high_value" },
      { "order_id": "O101", "order_total": 504, "order_status": "high_value" }
    ]
  }
]
```


---

## ✅ test_deep_nested_expr_bucket_auto

### 📖 Description

This test:
- Uses `$match` with `$expr` to filter products with price > 100
- Applies `$facet` to run:
  - `$bucketAuto`: Group prices into 2 auto-sized buckets with a count
  - `$group` + `$sort`: Identify top products by sales value

### 📥 Input Document

```json
[
  {"_id": 1, "product": "Laptop", "price": 1200},
  {"_id": 2, "product": "Phone", "price": 800},
  {"_id": 3, "product": "Tablet", "price": 400},
  {"_id": 4, "product": "Mouse", "price": 50},
  {"_id": 5, "product": "Charger", "price": 30}
]
```

### 📌 Pipeline

```json
[
  { "$match": { "$expr": { "$gt": ["$price", 100] } } },
  {
    "$facet": {
      "priceBuckets": [
        {
          "$bucketAuto": {
            "groupBy": "$price",
            "buckets": 2,
            "output": { "count": { "$sum": 1 } }
          }
        }
      ],
      "topProducts": [
        {
          "$group": {
            "_id": "$product",
            "totalSales": { "$sum": "$price" }
          }
        },
        { "$sort": { "totalSales": -1 } }
      ]
    }
  }
]
```

### 📤 Output

```json
[
  {
    "priceBuckets": [
      { "_id": { "min": 400, "max": 800 }, "count": 1 },
      { "_id": { "min": 800, "max": 1200 }, "count": 1 }
    ],
    "topProducts": [
      { "_id": "Laptop", "totalSales": 1200 },
      { "_id": "Phone", "totalSales": 800 },
      { "_id": "Tablet", "totalSales": 400 }
    ]
  }
]
```

---

## ✅ test_change_internal_array_object_structure

### 📖 Description

This test filters nested array `items` based on a sub-array condition:
- Only keeps those with feature `"Anti-Slip Pads"`
- Reshapes inner structure using `$map`, `$filter`, and `$cond`

### 📥 Input Document (abbreviated)

```json
[
  {
    "order_id": "ORD123456",
    "items": [
      {
        "product_id": "PROD001",
        "features": [
          { "feature_name": "Waterproof", "description": "Water-resistant material" }
        ]
      },
      {
        "product_id": "PROD002",
        "features": [
          { "feature_name": "Anti-Slip Pads", "description": "Rubber pads prevent slipping and scratching" }
        ]
      }
    ]
  }
]
```

### 📌 Pipeline

```json
[
  {
    "$set": {
      "items": {
        "$map": {
          "input": {
            "$filter": {
              "input": "$items",
              "as": "item",
              "cond": {
                "$gt": [
                  {
                    "$size": {
                      "$filter": {
                        "input": { "$ifNull": ["$$item.features", []] },
                        "as": "feature",
                        "cond": { "$eq": ["$$feature.feature_name", "Anti-Slip Pads"] }
                      }
                    }
                  },
                  0
                ]
              }
            }
          },
          "as": "item",
          "in": {
            "product_id": "$$item.product_id",
            "features": {
              "$map": {
                "input": {
                  "$filter": {
                    "input": { "$ifNull": ["$$item.features", []] },
                    "as": "feature",
                    "cond": { "$eq": ["$$feature.feature_name", "Anti-Slip Pads"] }
                  }
                },
                "as": "feature",
                "in": {
                  "title": "$$feature.feature_name",
                  "details": "$$feature.description",
                  "enabled": {
                    "$cond": {
                      "if": { "$eq": ["$$feature.feature_name", "Anti-Slip Pads"] },
                      "then": true,
                      "else": false
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  }
]
```

### 📤 Output

```json
[
  {
    "order_id": "ORD123456",
    "items": [
      {
        "product_id": "PROD002",
        "features": [
          {
            "title": "Anti-Slip Pads",
            "details": "Rubber pads prevent slipping and scratching",
            "enabled": true
          }
        ]
      }
    ]
  }
]
```

---
