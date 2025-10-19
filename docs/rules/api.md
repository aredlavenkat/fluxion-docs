# API Reference

This page summarises the primary Java types involved in Fluxion's rule engine. It is not a replacement for Javadoc, but it captures the relationships and most frequently used methods so tooling and documentation generators can cross-link them.

## Domain model

| Type | Purpose | Key methods |
| --- | --- | --- |
| `RuleDefinition` | Immutable description of a single rule (pipeline + actions + metadata). | `name()`, `salience()`, `condition()`, `actions()`, `hooks()` |
| `RuleDefinition.Builder` | Fluent builder that validates on `build()`. | `salience(int)`, `condition(RuleCondition)`, `addAction(RuleAction)`, `addHook(RuleHook)`, `addHookByName(String)` |
| `RuleCondition` | Wrapper around a list of `Stage` objects. | `RuleCondition.pipeline(List<Stage>)`, `pipeline()` |
| `RuleSet` | Immutable collection of rules plus shared metadata and hooks. | `rules()`, `hooks()`, `metadata()` |
| `RuleSet.Builder` | Fluent builder for rule sets. | `addRule(RuleDefinition)`, `addHook(RuleSetHook)`, `addHookByName(String)` |

## Runtime types

| Type | Purpose | Key methods |
| --- | --- | --- |
| `RuleEngine` | Evaluates documents against a rule set. | `evaluate(List<Document>, RuleSet)`, `evaluate(..., boolean debug)`, `execute(...)` |
| `RuleEvaluationResult` | Outcome for a single document. | `document()`, `passes()`, `sharedAttributes()`, `ruleContexts()` |
| `RulePass` | Indicates a rule whose condition pipeline passed. | `rule()`, `context()`, `actions()` |
| `RuleExecutionContext` | Mutable per-rule context that actions and hooks can use. | `attributes()`, `putAttribute(String, Object)`, `sharedAttributes()`, `transformedDocuments()`, `debugTrace()` |
| `RuleDebugStageTrace` (`ai.fluxion.rules.debug`) | Debug snapshot for a single stage. | `index()`, `operator()`, `inputs()`, `outputs()`, `filtered()`, `transformed()`, `error()` |
| `RulePipelineExecutor` | Internal adapter between rules and the core `PipelineExecutor`. | `execute(RuleDefinition, RuleExecutionContext)` |

## Registry & SPI types

| Type | Purpose |
| --- | --- |
| `RuleActionRegistry` | Global registry of `RuleAction` instances. Supports manual registration (`register`, `unregister`) and automatic discovery via `RuleActionContributor` SPI. |
| `RuleHookRegistry` | Resolves named `RuleHook` and `RuleSetHook` instances, including SPI-contributed hooks. |
| `StageRegistry` | Core registry of aggregation stages. Extended via `StageHandlerContributor` to publish custom operators. |
| `RuleActionContributor` | SPI interface for publishing action implementations via ServiceLoader. |
| `RuleHookContributor` | SPI interface for rule/rule-set hooks. |
| `StageHandlerContributor` | SPI interface for new aggregation stages. |

## Parser & validation classes

| Type | Purpose | Notes |
| --- | --- | --- |
| `RuleDslParser` | Converts JSON DSL into `RuleSet`. | Use `parseWithLints` for non-throwing validation. |
| `RuleParseResult` | Holds parsed rule set plus lint list. | `hasRuleSet()`, `ruleSet()`, `hasLints()`, `lints()` |
| `RuleLintCollector` | Static lint discovery for rule definitions and DSL payloads. | `collect(RuleDefinition)`, `collectFromDsl(JsonNode)` |
| `RuleLint` | Structured lint message. | `type()`, `message()`, `ruleName()`, `context()` |
| `RuleLintType` | Enum of lint categories (`MISSING_STAGE`, `UNSUPPORTED_OPERATOR`, `DUPLICATED_SALIENCE`). |  |
| `RuleValidator` | Strict validation invoked during builder `build()`. | Throws `RuleValidationException` on failure. |

## Actions, hooks, and context interfaces

| Type | Purpose |
| --- | --- |
| `RuleAction` | Functional interface invoked during `execute`. Receives a `RuleExecutionContext`. |
| `RuleHook` | Offers `beforeEvaluation` and `afterActions` hooks per rule evaluation. Default methods make implementation optional. |
| `RuleSetHook` | Offers `beforeRules` and `afterRules` hooks for the entire rule set per document. |

## Utilities & support

- `RulePipelineAdapter` converts a `RuleDefinition` into the list of `Stage` objects required by the core pipeline executor.
- `RulePipelineResult` encapsulates pipeline outputs plus `StageMetrics` so the context can expose them later.
- `RuleTester` (legacy harness) has been deprecated in favour of direct `RuleEngine` usage; see [Testing & Debugging](testing.md).

## Package overview

```
ai.fluxion.rules.actions        // Action registry and contributors
ai.fluxion.rules.domain         // RuleDefinition, RuleSet, builders
ai.fluxion.rules.dsl            // JSON parser and lint integration
ai.fluxion.rules.engine         // Runtime execution, contexts, hooks
ai.fluxion.rules.spi            // ServiceLoader extension points
ai.fluxion.rules.validation     // RuleValidator, lint collector
```

Use this map to configure IDE auto-completion, documentation cross-links, or VS Code snippet generators.
