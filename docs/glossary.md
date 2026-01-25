# 🧠 Glossary

---

## 📦 Key Concepts

**Pipeline**  
A series of stages executed in order for each document. In SrotaX Core the focus is currently on single-document transformations (fan-out writes are on the roadmap).

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
`PipelineExecutor` – the Java entry point that evaluates stages and expressions.

**Expression**  
Any logic within fields: computations, filters, conditions.

---

## 🧱 Stage & Operator Catalogues

See the [Stages](../stages/index.md) and [Operators](../operators/index.md) references for the up-to-date catalogues, syntax, and examples. Stages such as `$merge`, `$out`, `$search`, and `$vectorSearch` are reserved but **not supported** in SrotaX Core yet.
