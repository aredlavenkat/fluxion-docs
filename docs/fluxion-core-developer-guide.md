# Fluxion Core Developer Guide

## Overview

Fluxion Core powers the aggregation engine that executes MongoDB-compatible pipelines inside the Fluxion platform. This document gives Java developers a concise, always-up-to-date view of the built-in operators and stages, plus guidance on extending the engine with custom functionality. Use it alongside the module source (`fluxion-core/src/main/java/ai/fluxion/core`) when integrating or contributing operators and stages.

## Aggregation Operators

All operators are registered through `OperatorRegistry` and follow MongoDB semantics unless noted. Usage examples assume they appear inside an aggregation expression (e.g., `$project`, `$addFields`, `$group`).

### Comparison & Logical

| Operator | Description | Example usage |
| --- | --- | --- |
| `$eq` | Strict equality comparison | `{"$eq": ["$status", "ACTIVE"]}` |
| `$ne` | Inequality comparison | `{"$ne": ["$category", "archived"]}` |
| `$gt` | Greater-than comparison | `{"$gt": ["$score", 80]}` |
| `$gte` | Greater-than-or-equal comparison | `{"$gte": ["$inventory", 10]}` |
| `$lt` | Less-than comparison | `{"$lt": ["$latencyMs", 5]}` |
| `$lte` | Less-than-or-equal comparison | `{"$lte": ["$priority", 3]}` |
| `$in` | Value containment test | `{"$in": ["$region", ["us-east", "us-west"]]}` |
| `$nin` | Negated containment test | `{"$nin": ["$role", ["internal", "deprecated"]]}` |
| `$cmp` | Three-way comparison returning -1/0/1 | `{"$cmp": ["$version", "$latestVersion"]}` |
| `$and` | Logical AND across expressions | `{"$and": [{"$gt": ["$score", 70]}, {"$lt": ["$score", 90]}]}` |
| `$or` | Logical OR across expressions | `{"$or": [{"$eq": ["$tier", "gold"]}, {"$eq": ["$tier", "platinum"]}]}` |
| `$not` | Logical negation | `{"$not": [{"$in": ["$status", ["disabled", "deleted"]]}]}` |
| `$nor` | Logical NOR (none match) | `{"$nor": [{"$eq": ["$state", "PENDING"]}, {"$eq": ["$state", "FAILED"]}]}` |

### Math & Numeric

| Operator | Description | Example usage |
| --- | --- | --- |
| `$add` | Adds numbers or concatenates dates and durations | `{"$add": ["$price", "$tax"]}` |
| `$subtract` | Subtracts numbers or dates | `{"$subtract": ["$endTime", "$startTime"]}` |
| `$multiply` | Multiplies numbers | `{"$multiply": ["$quantity", "$unitPrice"]}` |
| `$divide` | Divides numbers | `{"$divide": ["$distanceMeters", "$durationSec"]}` |
| `$mod` | Remainder of division | `{"$mod": ["$shard", 4]}` |
| `$abs` | Absolute value | `{"$abs": ["$delta"]}` |
| `$ceil` | Round upward | `{"$ceil": ["$cpuUsage"]}` |
| `$floor` | Round downward | `{"$floor": ["$cpuUsage"]}` |
| `$trunc` | Truncate toward zero | `{"$trunc": ["$rating", 1]}` |
| `$round` | Bankers rounding with optional place | `{"$round": ["$score", 2]}` |
| `$exp` | Euler's number power | `{"$exp": ["$growthRate"]}` |
| `$ln` | Natural logarithm | `{"$ln": ["$hits"]}` |
| `$log10` | Base-10 logarithm | `{"$log10": ["$pageViews"]}` |
| `$log` | Logarithm with custom base | `{"$log": ["$value", 10]}` |
| `$bitAnd` | Bitwise AND | `{"$bitAnd": [ "$permissions", 0x04 ]}` |
| `$bitOr` | Bitwise OR | `{"$bitOr": [ "$flags", 0x02 ]}` |
| `$bitXor` | Bitwise XOR | `{"$bitXor": [ "$featureMask", 0x10 ]}` |
| `$pow` | Power function | `{"$pow": ["$variance", 0.5]}` |
| `$sqrt` | Square root | `{"$sqrt": ["$variance"]}` |
| `$sin` | Sine (radians) | `{"$sin": ["$angleRad"]}` |
| `$cos` | Cosine (radians) | `{"$cos": ["$angleRad"]}` |
| `$tan` | Tangent (radians) | `{"$tan": ["$angleRad"]}` |
| `$asin` | Arcsine | `{"$asin": ["$ratio"]}` |
| `$acos` | Arccosine | `{"$acos": ["$ratio"]}` |
| `$atan` | Arctangent | `{"$atan": ["$ratio"]}` |
| `$atan2` | Arctangent with two args | `{"$atan2": ["$y", "$x"]}` |
| `$sinh` | Hyperbolic sine | `{"$sinh": ["$angle"]}` |
| `$cosh` | Hyperbolic cosine | `{"$cosh": ["$angle"]}` |
| `$tanh` | Hyperbolic tangent | `{"$tanh": ["$angle"]}` |
| `$asinh` | Inverse hyperbolic sine | `{"$asinh": ["$ratio"]}` |
| `$acosh` | Inverse hyperbolic cosine | `{"$acosh": ["$ratio"]}` |
| `$atanh` | Inverse hyperbolic tangent | `{"$atanh": ["$ratio"]}` |
| `$degreesToRadians` | Convert degrees to radians | `{"$degreesToRadians": ["$bearingDeg"]}` |
| `$radiansToDegrees` | Convert radians to degrees | `{"$radiansToDegrees": ["$bearingRad"]}` |
| `$min` | Minimum of arguments | `{"$min": ["$minLatency", "$observedLatency"]}` |
| `$max` | Maximum of arguments | `{"$max": ["$peak", "$current"]}` |
| `$avg` | Average of arguments | `{"$avg": ["$temperatureMorning", "$temperatureEvening"]}` |
| `$sum` | Sum of arguments | `{"$sum": "$readings"}` |
| `$minDistance` | Minimum geo distance (flat or spherical) | `{"$minDistance": {"input": "$origin", "points": "$warehouseCoords"}}` |

### String Processing

| Operator | Description | Example usage |
| --- | --- | --- |
| `$concat` | Concatenate strings | `{"$concat": ["$firstName", " ", "$lastName"]}` |
| `$toLower` | Lowercase string | `{"$toLower": ["$email"]}` |
| `$toUpper` | Uppercase string | `{"$toUpper": ["$country"]}` |
| `$trim` | Trim substring from both ends | `{"$trim": {"input": "$path", "chars": "/"}}` |
| `$ltrim` | Trim from start | `{"$ltrim": {"input": "$path", "chars": "/"}}` |
| `$rtrim` | Trim from end | `{"$rtrim": {"input": "$path", "chars": "/"}}` |
| `$substr` | Substring by code units | `{"$substr": ["$id", 0, 6]}` |
| `$substrBytes` | Substring by bytes | `{"$substrBytes": ["$binHex", 0, 4]}` |
| `$substrCP` | Substring by code points | `{"$substrCP": ["$emojiString", 0, 2]}` |
| `$strLenBytes` | String length in bytes | `{"$strLenBytes": ["$raw"]}` |
| `$strLenCP` | String length in code points | `{"$strLenCP": ["$emojiString"]}` |
| `$split` | Split string into array | `{"$split": ["$tags", ","]}` |
| `$indexOfBytes` | Byte offset of substring | `{"$indexOfBytes": ["$url", "://"]}` |
| `$indexOfCP` | Code point offset | `{"$indexOfCP": ["$title", "★"]}` |
| `$strcasecmp` | Case-insensitive compare | `{"$strcasecmp": ["$locale", "en_US"]}` |
| `$regexMatch` | Boolean regex test | `{"$regexMatch": {"input": "$sku", "regex": "^SKU-"}}` |
| `$regexFind` | First regex match info | `{"$regexFind": {"input": "$body", "regex": "(?<code>\\d{6})"}}` |
| `$regexFindAll` | All regex matches info | `{"$regexFindAll": {"input": "$body", "regex": "\\b[A-Z]{3}\\b"}}` |
| `$replaceOne` | Replace first match | `{"$replaceOne": {"input": "$path", "find": "/tmp", "replacement": "/var"}}` |
| `$replaceAll` | Replace all matches | `{"$replaceAll": {"input": "$slug", "find": "_", "replacement": "-"}}` |
| `$startsWith` | Prefix test | `{"$startsWith": {"input": "$name", "prefix": "svc-"}}` |
| `$endsWith` | Suffix test | `{"$endsWith": {"input": "$name", "suffix": ".log"}}` |

### Conditional & Control Flow

| Operator | Description | Example usage |
| --- | --- | --- |
| `$cond` | If/then/else branching | `{"$cond": [{"$gt": ["$score", 90]}, "A", "B"]}` |
| `$ifNull` | Fallback when null/undefined | `{"$ifNull": ["$nickname", "$firstName"]}` |
| `$switch` | Multi-branch evaluation | `{"$switch": {"branches": [{"case": {"$eq": ["$tier", "gold"]}, "then": 0.9}], "default": 1}}` |
| `$coalesce` | First non-null argument | `{"$coalesce": ["$preferred", "$fallback", "unknown"]}` |
| `$let` | Introduce scoped variables | `{"$let": {"vars": {"total": {"$sum": "$items"}}, "in": {"$gt": ["$$total", 5]}}}` |

### Array & Collection

| Operator | Description | Example usage |
| --- | --- | --- |
| `$map` | Transform array elements | `{"$map": {"input": "$items", "as": "item", "in": {"$toUpper": "$$item"}}}` |
| `$filter` | Filter array elements | `{"$filter": {"input": "$items", "as": "item", "cond": {"$gt": ["$$item.price", 10]}}}` |
| `$reduce` | Fold array to a single value | `{"$reduce": {"input": "$items", "initialValue": 0, "in": {"$add": ["$$value", "$$this.qty"]}}}` |
| `$range` | Generate integer range | `{"$range": [0, {"$size": "$items"}]}` |
| `$size` | Array length | `{"$size": "$items"}` |
| `$isArray` | Test for array type | `{"$isArray": "$items"}` |
| `$arrayElemAt` | Element at index | `{"$arrayElemAt": ["$tags", 0]}` |
| `$slice` | Sub-array by range | `{"$slice": ["$tags", 3]}` |
| `$indexOfArray` | Index of element | `{"$indexOfArray": ["$tags", "new"]}` |
| `$concatArrays` | Merge arrays | `{"$concatArrays": ["$primaryTags", "$secondaryTags"]}` |
| `$reverseArray` | Reverse array order | `{"$reverseArray": "$events"}` |
| `$first` | First array element | `{"$first": "$events"}` |
| `$last` | Last array element | `{"$last": "$events"}` |
| `$zip` | Zip multiple arrays | `{"$zip": {"inputs": ["$skus", "$counts"]}}` |

### Type & Utility

| Operator | Description | Example usage |
| --- | --- | --- |
| `$literal` | Return value without parsing | `{"$literal": {"$thisIsNotParsed": true}}` |
| `$toLong` | Convert to 64-bit integer | `{"$toLong": "$sequenceNumber"}` |
| `$toInt` | Convert to 32-bit integer | `{"$toInt": "$count"}` |
| `$toDouble` | Convert to double | `{"$toDouble": "$score"}` |
| `$toDecimal` | Convert to decimal | `{"$toDecimal": "$amount"}` |
| `$toDate` | Convert to date | `{"$toDate": "$timestamp"}` |
| `$toBool` | Convert to boolean | `{"$toBool": "$flag"}` |
| `$toString` | Convert to string | `{"$toString": "$_id"}` |
| `$toObjectId` | Convert to ObjectId | `{"$toObjectId": "$hexId"}` |
| `$type` | BSON type name | `{"$type": "$payload"}` |
| `$convert` | Generalized conversion with onError/onNull | `{"$convert": {"input": "$maybeNumber", "to": "int", "onError": 0}}` |
| `$isNumber` | Numeric test | `{"$isNumber": "$maybeNumber"}` |
| `$binarySize` | BSON binary size | `{"$binarySize": "$file"}` |
| `$bsonSize` | Size of document in bytes | `{"$bsonSize": "$$ROOT"}` |

### Object Manipulation

| Operator | Description | Example usage |
| --- | --- | --- |
| `$mergeObjects` | Merge documents left-to-right | `{"$mergeObjects": ["$defaults", "$overrides"]}` |
| `$objectToArray` | Convert document to key/value array | `{"$objectToArray": "$metadata"}` |
| `$arrayToObject` | Convert array of pairs into document | `{"$arrayToObject": "$kvPairs"}` |
| `$getField` | Read dynamic field | `{"$getField": {"field": "$fieldName", "input": "$metrics"}}` |
| `$setField` | Set dynamic field | `{"$setField": {"field": "status", "input": "$$ROOT", "value": "ACTIVE"}}` |
| `$unsetField` | Remove dynamic field | `{"$unsetField": {"field": "internalOnly", "input": "$document"}}` |

### Set Operators

| Operator | Description | Example usage |
| --- | --- | --- |
| `$setEquals` | True when sets contain same elements | `{"$setEquals": ["$tagsA", "$tagsB"]}` |
| `$setIntersection` | Intersection of sets | `{"$setIntersection": ["$tagsA", "$tagsB"]}` |
| `$setUnion` | Union of sets | `{"$setUnion": ["$tagsA", "$tagsB"]}` |
| `$setDifference` | Elements in first not in second | `{"$setDifference": ["$tagsA", "$tagsB"]}` |
| `$setIsSubset` | Subset test | `{"$setIsSubset": ["$childTags", "$parentTags"]}` |
| `$setIsSuperset` | Superset test | `{"$setIsSuperset": ["$parentTags", "$childTags"]}` |
| `$anyElementTrue` | Any truthy | `{"$anyElementTrue": "$flags"}` |
| `$allElementsTrue` | All truthy | `{"$allElementsTrue": "$flags"}` |

### Date & Time

| Operator | Description | Example usage |
| --- | --- | --- |
| `$year` | Extract ISO year | `{"$year": "$ts"}` |
| `$month` | Extract month number | `{"$month": "$ts"}` |
| `$dayOfMonth` | Day within month | `{"$dayOfMonth": "$ts"}` |
| `$dayOfWeek` | Day of week (1-7) | `{"$dayOfWeek": "$ts"}` |
| `$dayOfYear` | Day within year | `{"$dayOfYear": "$ts"}` |
| `$hour` | Hour of day | `{"$hour": "$ts"}` |
| `$minute` | Minute of hour | `{"$minute": "$ts"}` |
| `$second` | Second of minute | `{"$second": "$ts"}` |
| `$millisecond` | Millisecond component | `{"$millisecond": "$ts"}` |
| `$dateAdd` | Add units to date | `{"$dateAdd": {"startDate": "$ts", "unit": "hour", "amount": 4}}` |
| `$dateSubtract` | Subtract units from date | `{"$dateSubtract": {"startDate": "$ts", "unit": "minute", "amount": 30}}` |
| `$dateDiff` | Difference between dates | `{"$dateDiff": {"startDate": "$start", "endDate": "$end", "unit": "day"}}` |
| `$isoWeek` | ISO week number | `{"$isoWeek": "$ts"}` |
| `$isoDayOfWeek` | ISO day (1-7) | `{"$isoDayOfWeek": "$ts"}` |
| `$isoWeekYear` | ISO week-based year | `{"$isoWeekYear": "$ts"}` |
| `$dateTrunc` | Truncate to unit | `{"$dateTrunc": {"date": "$ts", "unit": "hour"}}` |
| `$dateToString` | Format date string | `{"$dateToString": {"date": "$ts", "format": "%Y-%m-%d"}}` |
| `$dateFromString` | Parse date from string | `{"$dateFromString": {"dateString": "$isoDate"}}` |
| `$dateFromParts` | Build date from parts | `{"$dateFromParts": {"year": 2024, "month": 1, "day": 1}}` |
| `$dateToParts` | Break date into parts | `{"$dateToParts": {"date": "$ts", "iso8601": true}}` |
| `$tsSecond` | Extract BSON Timestamp seconds | `{"$tsSecond": "$tsValue"}` |
| `$tsIncrement` | Extract BSON Timestamp increment | `{"$tsIncrement": "$tsValue"}` |
| `$week` | Week of year | `{"$week": "$ts"}` |

### Special Execution

| Operator | Description | Example usage |
| --- | --- | --- |
| `$function` | Invoke JavaScript-style function provided via `body`, `args`, and optional `lang` | `{"$function": {"body": "function(x){ return x * 2; }", "args": ["$value"], "lang": "js"}}` |
| `$expr` | Evaluate aggregation expression in query context | `{"$expr": {"$gt": ["$score", 90]}}` |

## Aggregation Stages

Stages coordinate documents flowing through the pipeline and are registered via `StageRegistry`. Aliases (`$set` → `$addFields`, `$replaceWith` → `$replaceRoot`) share the same implementation.

### `$match`
- Filters documents using query expressions.
- Example:
```json
{ "$match": { "status": "ACTIVE", "score": { "$gte": 80 } } }
```

### `$project`
- Reshapes documents by including/excluding fields and adding expressions.
- Example:
```json
{ "$project": { "_id": 0, "name": 1, "score": 1, "isHighValue": { "$gt": ["$score", 90] } } }
```

### `$addFields` / `$set`
- Adds new fields or overwrites existing ones with expression results.
- Example:
```json
{ "$set": { "total": { "$add": ["$subtotal", "$tax"] } } }
```

### `$unset`
- Removes one or more fields.
- Example:
```json
{ "$unset": ["internalNotes", "debugOnly"] }
```

### `$limit`
- Restricts the number of documents passed downstream.
- Example:
```json
{ "$limit": 100 }
```

### `$skip`
- Skips the first N documents.
- Example:
```json
{ "$skip": 200 }
```

### `$sort`
- Orders documents by specified keys.
- Example:
```json
{ "$sort": { "score": -1, "timestamp": 1 } }
```

### `$group`
- Groups documents by an `_id` expression and computes accumulators.
- Example:
```json
{ "$group": { "_id": "$customerId", "total": { "$sum": "$amount" }, "latest": { "$max": "$timestamp" } } }
```

### `$unwind`
- Deconstructs array fields to emit one document per element.
- Example:
```json
{ "$unwind": { "path": "$items", "preserveNullAndEmptyArrays": true } }
```

### `$replaceRoot` / `$replaceWith`
- Promotes a sub-document to the top level.
- Example:
```json
{ "$replaceWith": "$details.profile" }
```

### `$facet`
- Runs multiple sub-pipelines in parallel and outputs all results in one document.
- Example:
```json
{ "$facet": {
    "topScores": [
      { "$sort": { "score": -1 } },
      { "$limit": 5 }
    ],
    "byStatus": [
      { "$group": { "_id": "$status", "count": { "$sum": 1 } } }
    ]
  }
}
```

### `$bucket`
- Groups documents into user-defined boundary ranges and outputs counts/accumulators.
- Example:
```json
{ "$bucket": {
    "groupBy": "$score",
    "boundaries": [0, 50, 75, 90, 100],
    "default": "other",
    "output": { "count": { "$sum": 1 } }
  }
}
```

### `$bucketAuto`
- Automatically creates evenly distributed buckets.
- Example:
```json
{ "$bucketAuto": {
    "groupBy": "$latencyMs",
    "buckets": 4,
    "output": { "avgLatency": { "$avg": "$latencyMs" } }
  }
}
```

### `$sortByCount`
- Groups by an expression and sorts by the count descending.
- Example:
```json
{ "$sortByCount": "$status" }
```

### `$lookup`
- Performs left outer join with another collection or pipeline.
- Example:
```json
{ "$lookup": {
    "from": "orders",
    "localField": "customerId",
    "foreignField": "customerId",
    "as": "orders"
  }
}
```

### `$densify`
- Fills gaps in sequence data by inserting synthetic documents.
- Example:
```json
{ "$densify": {
    "field": "timestamp",
    "range": { "step": 1, "unit": "hour", "bounds": "partition" },
    "partitionByFields": ["deviceId"]
  }
}
```

### `$fill`
- Interpolates or carries forward values for missing fields.
- Example:
```json
{ "$fill": {
    "sortBy": { "timestamp": 1 },
    "partitionBy": "$deviceId",
    "output": { "temperature": { "method": "linear" } }
  }
}
```

### `$setWindowFields`
- Executes window functions over partitions with ordering.
- Example:
```json
{ "$setWindowFields": {
    "partitionBy": "$deviceId",
    "sortBy": { "timestamp": 1 },
    "output": {
      "rollingAvg": {
        "$avg": "$temperature",
        "window": { "range": [-2, 0], "unit": "hour" }
      }
    }
  }
}
```

### `$sampleRate`
- Emits a probabilistic sample of documents based on the given rate.
- Example:
```json
{ "$sampleRate": { "rate": 0.1, "seed": 42 } }
```

### `$replaceWith` (alias noted above)

No additional example beyond `$replaceRoot`.

## Extending Fluxion Core

When adding new functionality, prefer the extension hooks that Fluxion exposes so that downstream applications can discover your contributions automatically.

### Adding a Custom Operator

1. Implement the `Operator` interface, resolving arguments with `ExpressionEvaluator` as needed.
2. Provide an `OperatorContributor` that registers the operator.
3. Register the contributor via Java's `ServiceLoader` by adding the fully qualified class name to `META-INF/services/ai.fluxion.core.expression.OperatorContributor`.

```java
package com.acme.fluxion.ops;

import ai.fluxion.core.expression.ExpressionEvaluator;
import ai.fluxion.core.expression.Operator;
import ai.fluxion.core.expression.OperatorContributor;
import ai.fluxion.core.expression.OperatorRegistry;
import ai.fluxion.core.model.Document;

import java.util.Map;

public final class PercentileOperator implements Operator {
    @Override
    public Object apply(Object expr, Document doc, Map<String, Object> vars, ExpressionEvaluator evaluator) {
        Map<String, Object> spec = (Map<String, Object>) expr;
        Number value = (Number) evaluator.resolve(spec.get("value"), doc, vars);
        Number percentile = (Number) spec.getOrDefault("percentile", 0.5d);
        return value.doubleValue() * percentile.doubleValue();
    }

    public static final class Contributor implements OperatorContributor {
        @Override
        public void contribute(OperatorRegistry registry) {
            registry.register("$percentile", new PercentileOperator());
        }
    }
}
```

`src/main/resources/META-INF/services/ai.fluxion.core.expression.OperatorContributor`:

```
com.acme.fluxion.ops.PercentileOperator$Contributor
```

During tests you can also register the operator imperatively:

```java
OperatorRegistry.getInstance().register("$percentile", new PercentileOperator());
```

### Nested Pipelines with `$subPipeline`

Use `$subPipeline` when you want to invoke child pipelines (registered or inline) inside a parent pipeline.

Sequential example (default):

```json
{
  "$subPipeline": [
    "cleanse@1",
    { "pipeline": [ { "$addFields": { "processed": true } } ] },
    "post-process"
  ]
}
```

- Each entry in the array is executed in order. A string references a registered pipeline (`name@version`).
- Inline child pipelines can be provided via `{ "pipeline": [ ... ] }`.

Parallel example:

```json
{
  "$subPipeline": [
    {
      "parallel": [ "enrich-geo", "enrich-fraud" ],
      "merge": "facet"
    }
  ]
}
```

- `parallel` takes an array of child refs or inline pipelines. Each branch can also include overrides (`ref`, `pipeline`, `globals`, `input`, `onError`).
- `merge` supports `concat`, `zip`, or `facet` to combine branch outputs.

### Adding a Custom Stage

1. Implement `StageHandler`, handling validation and document transformation inside `apply`.
2. Optionally create a `StageHandlerContributor` to register multiple stages.
3. Register the contributor with `META-INF/services/ai.fluxion.core.engine.spi.StageHandlerContributor`.

```java
package com.acme.fluxion.stages;

import ai.fluxion.core.aggregation.stages.StageHandler;
import ai.fluxion.core.engine.spi.StageHandlerContributor;
import ai.fluxion.core.model.Document;

import java.util.List;
import java.util.Map;

public final class MaskStageHandler implements StageHandler {
    @Override
    public List<Document> apply(List<Document> input, Object spec, Map<String, Object> vars) {
        Map<String, Object> config = (Map<String, Object>) spec;
        String field = (String) config.get("field");
        Object replacement = config.getOrDefault("replacement", "***");
        for (Document document : input) {
            if (document.containsKey(field)) {
                document.put(field, replacement);
            }
        }
        return input;
    }

    public static final class Contributor implements StageHandlerContributor {
        @Override
        public Map<String, StageHandler> stageHandlers() {
            return Map.of("$mask", new MaskStageHandler());
        }
    }
}
```

`src/main/resources/META-INF/services/ai.fluxion.core.engine.spi.StageHandlerContributor`:

```
com.acme.fluxion.stages.MaskStageHandler$Contributor
```

Need metrics or state-store access? Override `process` to receive the `StageExecutionContext`, then delegate to `StageHandler.super.process(...)` once finished.

### Testing and Validation Tips

- Favor unit tests under `fluxion-core/src/test/java` that exercise both happy-path and edge cases matching MongoDB semantics.
- Use `ExpressionEvaluator` in tests to execute raw operator expressions without constructing full pipelines.
- Pipeline-level tests can reuse `StageRegistry.getInstance().register(...)` to bind experimental handlers without ServiceLoader wiring.

## See Also

- `fluxion-core/src/main/java/ai/fluxion/core/expression`: operator implementations.
- `fluxion-core/src/main/java/ai/fluxion/core/aggregation/stages`: stage handlers and execution utilities.
- `Fluxion_Mongo_Operator_Support_Report.md`: compatibility matrix (kept in sync with this document).
