# Rule Engine Overview

SrotaX’s rule engine lets you wrap aggregation pipelines in declarative rules.
Each rule evaluates a document, tracks salience (priority), executes actions, and
exposes hooks/state you can integrate into services or workflow engines.

---

## 1. Prerequisites

| Requirement | Notes |
| --- | --- |
| SrotaX modules | `fluxion-core`, `fluxion-rules`; add `fluxion-connect` if you need `$httpCall`/`$sqlQuery`. |
| Runtime | JVM service/worker where you evaluate rules. |
| DSL / JSON | Rule-set definitions (JSON DSL or builder API). |
| Persistence (optional) | Wherever you store rule sets (Git, DB, config service). |

---

## 2. Core components

| Component | Purpose |
| --- | --- |
| `RuleDefinition` | Declares stages, salience, actions, metadata, and hooks for one rule. |
| `RuleSet` | Ordered collection of rules plus shared metadata and hooks. |
| `RuleEngine` | Evaluates documents against a rule set, returning passes and shared attributes. |
| `RuleAction` | Custom business logic triggered when a rule passes. |
| `RuleHook` / `RuleSetHook` | Pre/post evaluation callbacks for enrichment or auditing. |
| `RuleValidator` / `RuleLintCollector` | Static analysis to catch issues before runtime. |

### Platform context

```
┌────────────┐    documents     ┌────────────┐    passes/actions     ┌────────────┐
│ Connectors │ ───────────────▶ │ RuleEngine │ ────────────────────▶ │ Downstream │
└────────────┘                  └─────▲──────┘                       └─────▲──────┘
                                      │                                  │
                                      │ Rule DSL / API                   │ Shared attrs
                                      └──────────────────────────────────┘
```

For a full stack view (Core ↔ Connect ↔ Engines with enrichment inside Connect), see the
[Platform Architecture overview](../platform/overview.md).

---

## 3. Evaluation flow

1. **Document ingestion** – Connectors or application code supply a `Document`.
2. **Rule iteration** – Rules are sorted by salience (highest first). Missing
   stages or unsupported operators are rejected during validation.
3. **Pipeline execution** – Each rule runs its SrotaX stages via `RulePipelineExecutor`.
4. **Shared state update** – Rules can read/write `sharedAttributes` across the
   entire rule set.
5. **Actions & hooks** – On pass, actions run and hooks fire (`before/after`).
6. **Result assembly** – `RuleEvaluationResult` captures passes, shared state,
   debug traces, and stage metrics.

---

## 4. Capabilities at a glance

- **Pipeline native** – Reuse the same stages/operators as streaming pipelines.
- **Salience & priority** – Control the order in which rules execute.
- **Shared context** – Pass data between rules via `sharedAttributes`.
- **Hooks & actions** – Extend with ServiceLoader-based SPIs for side effects
  (HTTP calls, notifications, auditing, etc.).
- **Debug tracing** – Enable stage-by-stage traces for troubleshooting.
- **Static linting** – `RuleValidator` and `RuleLintCollector` catch issues
  before runtime (missing stages, duplicate salience, unsupported operators).

---

## 5. Integration checklist

| Task | Reference |
| --- | --- |
| Load rule sets | `fluxion-rules/src/main/java/.../RuleSetLoader` or custom code. |
| Validate rules | `RuleValidator.validateRule(...)`, `RuleLintCollector.collect(...)`. |
| Evaluate | `RuleEngine.evaluate(...)` or `RuleEngine.execute(...)` (with actions). |
| Inspect results | `RuleEvaluationResult` (passes, shared attributes, debug trace). |
| Extend actions/hooks | Implement `RuleAction`, `RuleHook`, or `RuleSetHook` via ServiceLoader. |

---

## 6. Reading guide

| Section | Use it when |
| --- | --- |
| [Quick Start](quickstart.md) | Try the engine with runnable samples. |
| [Authoring Rule Sets](authoring.md) | Design JSON DSL or builder-based rule definitions. |
| [DSL Reference](dsl-reference.md) | Need field-by-field schema. |
| [Runtime Execution](runtime.md) | Embed `RuleEngine` in a service or worker. |
| [API Reference](api.md) | Explore domain types (`RuleDefinition`, `RuleSet`, etc.). |
| [Validation & Linting](validation.md) | Build authoring tooling or CI checks. |
| [Testing & Debugging](testing.md) | Add regression tests and debug tracing. |
| [Extensions & SPIs](extensions.md) | Publish custom stages/actions/hooks. |
| [Best Practices](best-practices.md) | Naming, salience, and structure tips. |
| [Lifecycle & Governance](lifecycle.md) | Manage versions, approvals, deployments. |
| [Integration Guide](integration.md) | Wire rules into jobs or microservices. |
| [Tooling & IDE Support](tooling.md) | IDE/editor integration notes. |
| [Examples](examples.md) | Copy/paste walkthroughs. |

---

## 7. Useful source files

| Path | Why it matters |
| --- | --- |
| `fluxion-rules/src/main/java/.../RuleEngine.java` | Core evaluation logic. |
| `fluxion-rules/src/main/java/.../RuleDefinition.java` | Builder + metadata for rules. |
| `fluxion-rules/src/main/java/.../RuleSet.java` | Salience ordering, hooks, metadata storage. |
| `fluxion-rules/src/main/java/.../RuleValidator.java` | Validation entry point. |
| `fluxion-rules/src/test/java/...` | Sample rule JSON, unit tests, linting examples. |

Use these references when implementing rule authoring tools, validation
pipelines, or runtime integrations.
