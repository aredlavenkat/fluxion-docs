# 🧠 Glossary

---

## 📦 Key Concepts

**Pipeline**  
A series of stages executed in order on a document stream.

**Stage**  
A pipeline step (e.g., `$match`, `$group`, `$project`) that transforms documents.

**Operator**  
An expression keyword like `$sum`, `$map`, `$add`, used inside stages.

**Accumulator**  
Operators like `$sum`, `$avg`, `$min`, `$max` used within `$group`.

**System Variables**  
- `$$ROOT`: The entire document.
- `$$CURRENT`: The current level of the document.
- `$$NOW`: Current timestamp.
- `$$REMOVE`: Removes a field from projection.
- `$$CLUSTER_TIME`: Simulated logical timestamp.

**Executor**  
The Python engine that executes the pipeline logic.

**Expression**  
Any logic within fields: computations, filters, conditions.

---

## 🧱 Supported Stages (40+)

- `$addFields`
- `$bucket`
- `$bucketAuto`
- `$collStats` (coming soon)
- `$count`
- `$currentOp` (not supported)
- `$facet`
- `$geoNear` (coming soon)
- `$group`
- `$indexStats` (coming soon)
- `$limit`
- `$listSessions` (not supported)
- `$lookup`
- `$match`
- `$merge` (planned)
- `$out` (planned)
- `$planCacheStats` (planned)
- `$project`
- `$redact`
- `$replaceRoot`
- `$replaceWith`
- `$sample`
- `$set`
- `$skip`
- `$sort`
- `$sortByCount`
- `$unwind`
- `$function`
- `$expr`
- `$densify` (coming soon)
- `$fill` (coming soon)
- `$graphLookup` (coming soon)
- `$unionWith` (planned)
- `$documents`
- `$search` (planned)
- `$vectorSearch` (experimental)

---

## 🧮 Expression Operators (150+)

### 📏 Comparison
`$eq`, `$ne`, `$gt`, `$gte`, `$lt`, `$lte`, `$in`, `$nin`, `$cmp`

### 🔘 Boolean
`$and`, `$or`, `$not`, `$nor`

### ➕ Arithmetic
`$add`, `$subtract`, `$multiply`, `$divide`, `$mod`, `$abs`, `$ceil`, `$floor`, `$round`, `$sqrt`, `$trunc`

### 🔤 String
`$concat`, `$toLower`, `$toUpper`, `$trim`, `$ltrim`, `$rtrim`, `$substr`, `$substrBytes`, `$substrCP`, `$strLenBytes`, `$strLenCP`, `$split`, `$indexOfBytes`, `$indexOfCP`, `$strcasecmp`, `$regexMatch`

### 🎲 Conditional
`$cond`, `$ifNull`, `$switch`, `$default`

### 📚 Array
`$map`, `$filter`, `$reduce`, `$anyElementTrue`, `$allElementsTrue`, `$size`, `$isArray`, `$arrayElemAt`, `$slice`, `$indexOfArray`, `$concatArrays`, `$reverseArray`, `$first`, `$last`, `$min`, `$max`, `$zip`

### 🧱 Object
`$mergeObjects`, `$objectToArray`, `$arrayToObject`, `$getField`, `$setField`, `$unsetField`

### 🔁 Type Conversion
`$convert`, `$toString`, `$toInt`, `$toDouble`, `$toDecimal`, `$toBool`, `$type`

### 🧠 Variables & Literals
`$$ROOT`, `$$CURRENT`, `$$NOW`, `$$REMOVE`, `$literal`

### 🧬 Custom & Function
`$function`, `$accumulator`, `$let`, `$dateAdd`, `$dateSubtract`

---

Use the [Operators](../operators/) and [Stages](../stages/) sections for in-depth syntax and examples.
