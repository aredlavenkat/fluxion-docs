# Rule Engine Overview

Fluxion's rule engine lets you layer decision logic on top of aggregation pipelines. A rule wraps an ordered list of stages, attaches metadata and salience (priority), and optionally triggers actions or hooks when the pipeline passes. This section documents everything you need to author, validate, extend, and run rules—whether you are building tooling, integrating the runtime, or designing new SPIs.

**Key capabilities**

- *Pipeline native*: reuse the same aggregation operators and stages already available in Fluxion.
- *Priority control*: rules are sorted in descending salience so the most important decisions run first.
- *Rich context*: rule and rule-set metadata flow into evaluation results and can be surfaced to downstream systems.
- *Hook + action model*: pre/post evaluation hooks and pluggable actions allow enrichment, side-effects, and auditing.
- *Static validation*: missing stages, unsupported operators, and duplicate salience values are detected before runtime.
- *Debuggable execution*: optional debug tracing captures stage-by-stage inputs/outputs for any rule run.
- *Extensibility*: ServiceLoader-based SPIs let you publish custom stages, actions, and hooks from external modules.

### How the rule engine fits the platform

```
┌────────────┐    documents     ┌────────────┐    passes/actions     ┌────────────┐
│ Connectors │ ───────────────▶ │ RuleEngine │ ────────────────────▶ │ Downstream │
└────────────┘  (Fluxion Docs)  └─────▲──────┘    (hooks/actions)    └─────▲──────┘
                                       │                               │
                                       │ rules + DSL                   │ shared attrs
                              ┌────────┴────────┐                      │
                              │ Rule DSL / API │◀─────────────────────┘
                              └────────────────┘
```

- **Connectors** ingest data from Kafka/HTTP/etc.
- **RuleEngine** evaluates documents against rule sets built from the DSL or Java builders.
- **Downstream systems** receive passes, transformed documents, and action side-effects.

If you need the full stack view, the [Platform Architecture overview](../platform/overview.md)
explains how the Rule Engine layers above Fluxion Core, Connect, and Enrich.

### Reading guide

| Section | Use it when |
| --- | --- |
| [Quick Start](quickstart.md) | You want to try the engine end-to-end with working code samples. |
| [Authoring Rule Sets](authoring.md) | You are designing the DSL payloads or using the builder APIs. |
| [DSL Reference](dsl-reference.md) | You need a field-by-field schema for validation or tooling. |
| [Runtime Execution](runtime.md) | You are integrating the `RuleEngine` inside a service. |
| [API Reference](api.md) | You need a catalogue of domain types (`RuleDefinition`, `RuleSet`, etc.). |
| [Validation & Linting](validation.md) | You are building authoring tooling or CI checks. |
| [Testing & Debugging](testing.md) | You want reliable regression tests and troubleshooting guidance. |
| [Extensions & SPIs](extensions.md) | You plan to publish custom stages/actions/hooks. |
| [Best Practices](best-practices.md) | You want recommendations for salience, naming, and structure. |
| [Lifecycle & Governance](lifecycle.md) | You manage versions, approvals, and deployments. |
| [Integration Guide](integration.md) | You embed the engine into microservices or jobs. |
| [Tooling & IDE Support](tooling.md) | You are building editors or VS Code extensions. |
| [Examples](examples.md) | You learn best from copy/paste friendly walkthroughs. |

Use the navigation to dive into the sections that match your goal. Every page includes cross references so a reader—or an assistant—can understand the framework holistically.
