# Executor Test Suite – Scenario Guide

Friendly breakdown of the unit tests in `fluxion-core/src/test/java/.../PipelineExecutorTest.java`
covering `$match`, `$project`, system variables, and computed fields.

---

## How to run the tests

```bash
mvn -pl fluxion-core test -Dtest=PipelineExecutorTest
```

Or run the entire module:

```bash
mvn -pl fluxion-core -am test
```

---

## Scenario matrix

| Test | Purpose | Key stages/operators |
| --- | --- | --- |
| `test_simple_match_pipeline` | Filter documents where `value >= 20`. | `$match`, comparison operators |
| `test_project_include_fields` | Return only `name` and `age`. | `$project` (inclusion) |
| `test_project_exclude_fields` | Remove `city` field. | `$project` (exclusion) |
| `test_project_computed_fields` | Add `total` via `$multiply`. | `$project`, `$multiply` |
| `test_project_exclude_id` | Remove `_id` while keeping `name`. | `$project`, `_id` handling |
| `test_project_nested_fields` | Project nested subfields (`user.profile.first_name`). | `$project`, dotted paths |

Each scenario below lists the input, pipeline, and expected output so you can
replicate the behaviour in your own tests or services.

---

### 1. `test_simple_match_pipeline`

```json
Input:
[
  { "value": 10 },
  { "value": 20 },
  { "value": 30 }
]

Pipeline:
[
  { "$match": { "value": { "$gte": 20 } } }
]

Output:
[
  { "value": 20 },
  { "value": 30 }
]
```

**Tip:** `$match` supports the full MongoDB query syntax, including compound
conditions (`$and`, `$or`).

---

### 2. `test_project_include_fields`

```json
Input:
[
  { "name": "Alice", "age": 30, "city": "New York" }
]

Pipeline:
[
  { "$project": { "name": 1, "age": 1 } }
]

Output:
[
  { "name": "Alice", "age": 30 }
]
```

**Tip:** When using inclusion, `_id` is present unless explicitly excluded.

---

### 3. `test_project_exclude_fields`

```json
Input:
[
  { "name": "Bob", "age": 25, "city": "Los Angeles" }
]

Pipeline:
[
  { "$project": { "city": 0 } }
]

Output:
[
  { "name": "Bob", "age": 25 }
]
```

**Tip:** Exclusion-style projections cannot mix inclusion (other than `_id`).

---

### 4. `test_project_computed_fields`

```json
Input:
[
  { "item": "A", "price": 10, "qty": 2 }
]

Pipeline:
[
  {
    "$project": {
      "item": 1,
      "total": { "$multiply": ["$price", "$qty"] }
    }
  }
]

Output:
[
  { "item": "A", "total": 20 }
]
```

**Tip:** Combine `$multiply` with `$add`/`$divide` for discount or tax
calculations.

---

### 5. `test_project_exclude_id`

```json
Input:
[
  { "_id": 123, "name": "Charlie", "city": "Miami" }
]

Pipeline:
[
  { "$project": { "_id": 0, "name": 1 } }
]

Output:
[
  { "name": "Charlie" }
]
```

**Tip:** This is a common pattern when returning documents over REST APIs.

---

### 6. `test_project_nested_fields`

```json
Input:
[
  {
    "user": {
      "profile": {
        "first_name": "Dana",
        "last_name": "Smith"
      }
    },
    "age": 40
  }
]

Pipeline:
[
  { "$project": { "user.profile.first_name": 1, "age": 1, "_id": 0 } }
]

Output:
[
  {
    "user": { "profile": { "first_name": "Dana" } },
    "age": 40
  }
]
```

**Tip:** Use dotted paths to include/exclude nested fields. Combine with
`$map`/`$filter` when the nesting involves arrays.

---

Feel free to extend the suite with additional cases (renamed fields, computed
aliases, system variable usage). The structure above keeps everything explicit
for humans and assistants alike.

### 📌 Pipeline

```json
[
  { "$project": { "user.profile.first_name": 1, "age": 1 } }
]
```

### 📤 Output

```json
[
  {
    "user": { "profile": { "first_name": "Dana" } },
    "age": 40
  }
]
```

---

## ✅ test_project_renamed_fields_with_expression

### 📖 Description

Uses computed aliases for fields using `$project`.

### 📥 Input Document

```json
[
  { "name": "Eve", "visits": 5 }
]
```

### 📌 Pipeline

```json
[
  {
    "$project": {
      "username": "$name",
      "visit_count": "$visits"
    }
  }
]
```

### 📤 Output

```json
[
  { "username": "Eve", "visit_count": 5 }
]
```

---

## ✅ test_replace_with_simple_field

### 📖 Description

Replaces full document with a subdocument field using `$replaceWith`.

### 📥 Input Document

```json
[
  {
    "product": "Widget",
    "details": {
      "name": "Widget A",
      "price": 100
    }
  }
]
```

### 📌 Pipeline

```json
[
  { "$replaceWith": "$details" }
]
```

### 📤 Output

```json
[
  { "name": "Widget A", "price": 100 }
]
```

---

## ✅ test_replace_with_merge_objects_executor_style

### 📖 Description

Merges conditional object into the root using `$mergeObjects` and `$cond`.

### 📥 Input Document

```json
[
  { "product": "Widget", "price": 400 }
]
```

### 📌 Pipeline

```json
[
  {
    "$replaceWith": {
      "$mergeObjects": [
        { "type": "standard" },
        {
          "$cond": {
            "if": { "$gte": ["$price", 500] },
            "then": { "type": "premium" },
            "else": {}
          }
        },
        "$$ROOT"
      ]
    }
  }
]
```

### 📤 Output

```json
[
  { "type": "standard", "product": "Widget", "price": 400 }
]
```

---

## ✅ test_root_variable_deep_nested_merge_cond

### 📖 Description

Demonstrates conditional field injection and full merge with root document.

### 📥 Input Document

```json
[
  { "product": "Gadget", "price": 450 }
]
```

### 📌 Pipeline

```json
[
  {
    "$replaceWith": {
      "$mergeObjects": [
        { "default": "basic" },
        {
          "$cond": {
            "if": { "$gte": ["$price", 500] },
            "then": { "premiumField": true },
            "else": {}
          }
        },
        "$$ROOT"
      ]
    }
  }
]
```

### 📤 Output

```json
[
  { "default": "basic", "product": "Gadget", "price": 450 }
]
```

---


---

## ✅ test_now_variable_deep_nested

### 📖 Description

Adds `createdAt` using the system variable `$$NOW` and copies `user`.

### 📥 Input Document

```json
[
  { "user": "Alice" }
]
```

### 📌 Pipeline

```json
[
  {
    "$addFields": {
      "audit": {
        "createdAt": "$$NOW",
        "createdBy": "$user"
      }
    }
  }
]
```

### 📤 Output

```json
[
  {
    "user": "Alice",
    "audit": {
      "createdAt": "<timestamp>",
      "createdBy": "Alice"
    }
  }
]
```

---

## ✅ test_remove_variable_nested_project

### 📖 Description

Uses `$$REMOVE` to eliminate a field dynamically during `$project`.

### 📥 Input Document

```json
[
  {
    "name": "Alice",
    "privateField": "secret",
    "publicField": "info"
  }
]
```

### 📌 Pipeline

```json
[
  {
    "$project": {
      "publicField": 1,
      "privateField": "$$REMOVE",
      "profile": {
        "name": "$name"
      }
    }
  }
]
```

### 📤 Output

```json
[
  {
    "publicField": "info",
    "profile": {
      "name": "Alice"
    }
  }
]
```

---

## ✅ test_cluster_time_variable_nested

### 📖 Description

Adds cluster time using `$$CLUSTER_TIME`.

### 📥 Input Document

```json
[
  { "device": "IoT Sensor" }
]
```

### 📌 Pipeline

```json
[
  {
    "$addFields": {
      "metadata": {
        "clusterCapturedAt": "$$CLUSTER_TIME",
        "deviceName": "$device"
      }
    }
  }
]
```

### 📤 Output

```json
[
  {
    "device": "IoT Sensor",
    "metadata": {
      "clusterCapturedAt": "<cluster_time>",
      "deviceName": "IoT Sensor"
    }
  }
]
```

---

## ✅ test_expression_remove_variable

### 📖 Description

Removes `status` field using `$$REMOVE` during `$addFields`.

### 📥 Input Document

```json
[
  { "name": "Charlie", "status": "active" }
]
```

### 📌 Pipeline

```json
[
  {
    "$addFields": {
      "status": "$$REMOVE"
    }
  }
]
```

### 📤 Output

```json
[
  { "name": "Charlie" }
]
```

---

## ✅ test_expression_now_variable

### 📖 Description

Projects current timestamp using `$$NOW`.

### 📥 Input Document

```json
[
  { "user": "Bob" }
]
```

### 📌 Pipeline

```json
[
  { "$project": { "createdAt": "$$NOW" } }
]
```

### 📤 Output

```json
[
  { "createdAt": "<timestamp>" }
]
```

---


---

## ✅ test_function_double_price

### 📖 Description

Doubles the price using `$function` operator with a custom lambda.

### 📥 Input Document

```json
[
  { "price": 100 }
]
```

### 📌 Pipeline

```json
[
  {
    "$addFields": {
      "doublePrice": {
        "$function": {
          "body": "lambda price: price * 2",
          "args": ["$price"],
          "lang": "js"
        }
      }
    }
  }
]
```

### 📤 Output

```json
[
  { "price": 100, "doublePrice": 200 }
]
```

---

## ✅ test_function_add_two_fields

### 📖 Description

Adds two fields together using a custom lambda function.

### 📥 Input Document

```json
[
  { "a": 5, "b": 7 }
]
```

### 📌 Pipeline

```json
[
  {
    "$addFields": {
      "sumAB": {
        "$function": {
          "body": "lambda a, b: a + b",
          "args": ["$a", "$b"],
          "lang": "js"
        }
      }
    }
  }
]
```

### 📤 Output

```json
[
  { "a": 5, "b": 7, "sumAB": 12 }
]
```

---

## ✅ test_function_pass_fail_status

### 📖 Description

Evaluates whether a score passes or fails based on threshold logic.

### 📥 Input Document

```json
[
  { "score": 45 },
  { "score": 75 }
]
```

### 📌 Pipeline

```json
[
  {
    "$addFields": {
      "status": {
        "$function": {
          "body": "lambda score: 'pass' if score >= 50 else 'fail'",
          "args": ["$score"],
          "lang": "js"
        }
      }
    }
  }
]
```

### 📤 Output

```json
[
  { "score": 45, "status": "fail" },
  { "score": 75, "status": "pass" }
]
```

---

## ✅ test_function_complex_logic

### 📖 Description

Applies tiered pricing logic based on product category.

### 📥 Input Document

```json
[
  { "category": "A", "price": 100 },
  { "category": "B", "price": 200 }
]
```

### 📌 Pipeline

```json
[
  {
    "$addFields": {
      "adjustedPrice": {
        "$function": {
          "body": "lambda category, price: price * 0.9 if category == 'A' else price * 1.1",
          "args": ["$category", "$price"],
          "lang": "js"
        }
      }
    }
  }
]
```

### 📤 Output

```json
[
  { "category": "A", "price": 100, "adjustedPrice": 90.0 },
  { "category": "B", "price": 200, "adjustedPrice": 220.0 }
]
```

---


---

## ✅ test_expr_inside_match

### 📖 Description

Uses `$expr` inside `$match` to dynamically filter documents based on field values.

### 📥 Input Document

```json
[
  { "_id": 1, "price": 100 },
  { "_id": 2, "price": 600 },
  { "_id": 3, "price": 400 }
]
```

### 📌 Pipeline

```json
[
  {
    "$match": {
      "$expr": { "$gt": ["$price", 500] }
    }
  }
]
```

### 📤 Output

```json
[
  { "_id": 2, "price": 600 }
]
```

---

## ✅ test_extra_logic_pipeline

### 📖 Description

Classifies IoT sensor risk level using `$switch`, `$and`, `$or`, and `$lt` conditions.

### 📥 Input Document

```json
{
  "device_id": "sensor-500",
  "temperature": 42,
  "humidity": 82,
  "battery": 18,
  "device_status": "online",
  "signal_strength": -95,
  "location": "ZoneA",
  "uptime_hours": 400
}
```

### 📌 Pipeline

```json
[
  {
    "$addFields": {
      "risk_level": {
        "$switch": {
          "branches": [
            {
              "case": {
                "$and": [
                  { "$gte": ["$temperature", 40] },
                  { "$gte": ["$humidity", 80] },
                  { "$lt": ["$battery", 20] }
                ]
              },
              "then": "Critical"
            },
            {
              "case": {
                "$or": [
                  { "$lt": ["$signal_strength", -90] },
                  { "$eq": ["$device_status", "offline"] }
                ]
              },
              "then": "Warning"
            },
            {
              "case": {
                "$and": [
                  { "$gt": ["$uptime_hours", 300] },
                  { "$eq": ["$location", "ZoneA"] }
                ]
              },
              "then": "Maintenance Due"
            }
          ],
          "default": "Normal"
        }
      }
    }
  },
  {
    "$project": {
      "device_id": 1,
      "risk_level": 1,
      "battery": 1,
      "signal_strength": 1,
      "uptime_hours": 1
    }
  }
]
```

### 📤 Output

```json
[
  {
    "device_id": "sensor-500",
    "risk_level": "Critical",
    "battery": 18,
    "signal_strength": -95,
    "uptime_hours": 400
  }
]
```

---

## ✅ test_extra_deep_pipeline

### 📖 Description

This test:
- Computes total payment using `$reduce`
- Extracts last tracking event for each shipment using `$arrayElemAt` and `$size`
- Projects only essential order and customer fields

### 📥 Input Document

```json
{
  "order_id": "A1001",
  "customer": {
    "name": "Alice Wonderland",
    "loyalty": "gold"
  },
  "shipments": [
    {
      "shipment_id": "S1",
      "tracking_events": [
        { "status": "picked_up", "timestamp": "2023-01-10T10:00:00" },
        { "status": "in_transit", "timestamp": "2023-01-11T12:00:00" }
      ]
    },
    {
      "shipment_id": "S2",
      "tracking_events": [
        { "status": "picked_up", "timestamp": "2023-01-12T09:00:00" },
        { "status": "delivered", "timestamp": "2023-01-14T16:00:00" }
      ]
    }
  ],
  "payment": {
    "transactions": [
      { "txn_id": "T1", "amount": 120 },
      { "txn_id": "T2", "amount": 30 }
    ]
  }
}
```

### 📌 Pipeline

```json
[
  {
    "$addFields": {
      "total_payment_amount": {
        "$reduce": {
          "input": "$payment.transactions",
          "initialValue": 0,
          "in": { "$add": ["$$value", "$$this.amount"] }
        }
      }
    }
  },
  {
    "$addFields": {
      "last_status_per_shipment": {
        "$map": {
          "input": "$shipments",
          "as": "shipment",
          "in": {
            "shipment_id": "$$shipment.shipment_id",
            "last_event": {
              "$arrayElemAt": [
                "$$shipment.tracking_events",
                { "$subtract": [ { "$size": "$$shipment.tracking_events" }, 1 ] }
              ]
            }
          }
        }
      }
    }
  },
  {
    "$project": {
      "order_id": 1,
      "customer_name": "$customer.name",
      "customer_loyalty": "$customer.loyalty",
      "total_payment_amount": 1,
      "last_status_per_shipment": 1
    }
  }
]
```

### 📤 Output

```json
[
  {
    "order_id": "A1001",
    "customer_name": "Alice Wonderland",
    "customer_loyalty": "gold",
    "total_payment_amount": 150,
    "last_status_per_shipment": [
      {
        "shipment_id": "S1",
        "last_event": {
          "status": "in_transit",
          "timestamp": "2023-01-11T12:00:00"
        }
      },
      {
        "shipment_id": "S2",
        "last_event": {
          "status": "delivered",
          "timestamp": "2023-01-14T16:00:00"
        }
      }
    ]
  }
]
```

---
