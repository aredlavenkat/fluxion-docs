# Rule Engine Glossary

This glossary summarises the primary domain types exposed by `fluxion-rules` (and related helpers in `fluxion-core`). Use it as a quick reference when navigating APIs or designing tooling.

| Type | Purpose | Key fields / data | Typical usage |
| --- | --- | --- | --- |
| `RuleDefinition` | Immutable blueprint of a rule: it packages the metadata, salience, condition pipeline, actions, and hooks that define behaviour. | `id`, `name`, `description`, `salience`, `RuleCondition`, `List<RuleAction>`, rule-level `metadata`, `List<RuleHook>`. | Construct with the fluent builder or DSL; rules are sorted by salience within a `RuleSet`. |
| `RuleCondition` | Aggregation pipeline that decides whether a rule passes for a document. | Ordered `List<Stage>` (Fluxion core stages). | Created via `RuleCondition.pipeline(...)` when parsing DSL or building rules programmatically. |
| `RuleSet` | Immutable bundle of rules plus shared hooks/metadata. | `id`, `name`, `version`, sorted `List<RuleDefinition>`, rule-set `metadata`, `List<RuleSetHook>`. | Acts as the deployable artifact the `RuleEngine` evaluates. |
| `RuleAction` | Functional interface executed after a rule passes (during `execute`). | Receives `RuleExecutionContext`; can mutate context, shared attributes, emit side effects. | Register manually (`RuleActionRegistry.register`) or auto-discover via the SPI. |
| `RuleHook` | Per-rule lifecycle callbacks around evaluation/action execution. | Optional overrides for `beforeEvaluation(...)` and `afterActions(...)`. | Instrument or enrich individual rules without altering the engine. |
| `RuleSetHook` | Hooks that wrap evaluation of the entire rule set for a document. | `beforeRules(...)`, `afterRules(...)` (with access to passes). | Implement shared enrichment, auditing, telemetry across rules. |
| `RuleEngine` | Stateless orchestrator that runs documents through a `RuleSet`. | Delegates to `RulePipelineExecutor`, returns `RuleEvaluationResult`. | Use `evaluate` for read-only checks, `execute` to run actions; supports debug traces. |
| `RulePass` | Indicates a rule whose condition pipeline succeeded for the current document. | Access to the originating `RuleDefinition`, `RuleExecutionContext`, and queued `RuleAction`s. | Returned from `RuleEvaluationResult.passes()`; call `executeActions()` in post-processing. |
| `RuleEvaluationResult` | Aggregated outcome of evaluating one document. | `document()`, `passes()`, `sharedAttributes()`, `ruleContexts()`. | Inspect passes, per-rule contexts, debug traces, and shared state after evaluation. |
| `RuleExecutionContext` | Mutable per-rule context populated during evaluation. | Original `Document`, shared attributes map, rule metadata, transformed documents, `StageMetrics`, optional debug traces. | Accessible by actions, hooks, tests, and tooling. |
| `RuleDebugStageTrace` | Rule-facing wrapper over core debug traces showing stage-by-stage effects. | Stage index/operator, stage spec snapshot, input/output documents, captured exception (if any). | Available via `RuleExecutionContext.debugTrace()` when debug mode is enabled. |
| `RuleDslParser` | Converts JSON DSL payloads into validated `RuleSet`s. | Uses Jackson; emits lint warnings with context. | Call `parse` (strict) or `parseWithLints` (non-throwing) in authoring tools and CI pipelines. |
| `RuleParseResult` | Container for `parseWithLints` output. | Optional `RuleSet` plus ordered `List<RuleLint>`. | Check `hasLints()` to surface structured feedback before deployment. |
| `RuleLint` / `RuleLintType` | Structured representation of static validation issues. | Lint type (`MISSING_STAGE`, `UNSUPPORTED_OPERATOR`, `DUPLICATED_SALIENCE`), descriptive message, rule name, context map. | Display to rule authors; used by `RuleValidator` to enforce correctness. |
| `RuleValidator` | Strict validator invoked by builders. | Throws `RuleValidationException` if any lint remains. | Guarantees programmatic rule construction matches DSL validation rules. |
| `RulePipelineExecutor` | Rule-level adapter around the shared `PipelineExecutor`. | Executes a rule’s pipeline, records `StageMetrics`, captures debug traces. | Used internally by `RuleEngine`; rarely consumed directly. |
| `RulePipelineResult` | Value object capturing pipeline output. | Immutable list of resulting `Document`s and corresponding `StageMetrics`. | Fed back into `RuleExecutionContext` for transformed documents and metrics. |
| `RuleActionRegistry` | Global action registry. | Register, resolve, clear, reload actions. | Populate at application start-up or via ServiceLoader contributors. |
| `RuleHookRegistry` | Registry for named rule and rule-set hooks. | Register, resolve, clear, reload convenience methods. | Enables `addHookByName` in builders so hooks can be referenced declaratively. |
| `RuleActionContributor` / `RuleHookContributor` | ServiceLoader SPIs for reusable actions/hooks. | Return maps of names → implementations. | Package contributors and `META-INF/services` descriptors to auto-discover behaviours. |
| `PipelineExecutor` (core) | Shared stage execution engine used across Fluxion. | Executes `List<Stage>` pipelines, collects `StageMetrics`, supports debug tracing. | Rule engine delegates here; other components (streaming, batch) can reuse it. |
| `StageMetrics` | Aggregated performance metrics per stage. | Invocation counts, documents in/out, total duration, queue sizes (latest & max). | Exposed via `RuleExecutionContext.stageMetrics()` and exported through OpenTelemetry. |

> Need another term? Let us know—this glossary is meant to evolve alongside the rule engine.
