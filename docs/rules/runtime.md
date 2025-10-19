# Runtime Execution Flow

The rule engine executes rule sets in three phases:

1. **Rule-set hooks (before)** – `RuleSetHook.beforeRules` runs once per input document. Use it to populate shared attributes or perform pre-processing.
2. **Rule evaluation loop** – for each rule (sorted by salience):
   - A `RuleExecutionContext` is created with the source document, shared attributes, and metadata.
   - Optional `RuleHook.beforeEvaluation` callbacks fire.
   - The rule's pipeline is executed by `RulePipelineExecutor`, which delegates to the core `PipelineExecutor`.
   - If the pipeline yields documents, a `RulePass` is produced, containing the rule, context, and associated actions.
3. **Rule-set hooks (after)** – `RuleSetHook.afterRules` observes the aggregate passes and shared attributes for the document.

Finally, when `RuleEngine.execute` is used instead of `evaluate`, the engine iterates over passes and runs each action's `execute` method, followed by `RuleHook.afterActions` callbacks. `RuleEngine.evaluate` stops before actions run, making it suitable for read-only evaluation or "test this rule" flows.

## RuleExecutionContext

The context exposes:

- `document()` – original input document.
- `sharedAttributes()` – mutable map shared across rules in the same rule set.
- `attributes()` – rule-specific mutable map for actions and hooks.
- `transformedDocuments()` – output documents returned by the rule pipeline.
- `stageMetrics()` – aggregated pipeline metrics from the most recent execution.
- Optional debug trace (see below).

## Debug tracing

Call `RuleEngine.evaluate(..., true)` or `RuleEngine.execute(..., true)` to enable debug mode. When enabled, the underlying core `PipelineExecutor` captures per-stage snapshots and each `RuleExecutionContext` exposes them as a list of `RuleDebugStageTrace` entries (`ai.fluxion.rules.debug` package) containing:

- Stage index and operator name.
- Stage specification snapshot.
- Input and output document snapshots for that stage.
- Any exception thrown by the stage handler.

This trace is available via `context.debugTrace()` and is invaluable for building rule debugging tools. Each trace entry also exposes `filtered()` / `transformed()` booleans you can use to highlight the stage that changed execution. Because the snapshots come directly from the shared core `PipelineExecutor`, the same mechanism can power future engines (e.g. streaming) without additional work.

## Actions and hooks at runtime

- **Actions** run after the pipeline passes when `execute` is called. They can mutate the context (mutations are visible to downstream hooks) or trigger side effects.
- **Rule hooks** execute before and after actions, allowing you to audit or transform results.
- **Rule set hooks** run once per document and receive the list of passes, making them ideal for shared enrichment or aggregation.

The runtime enforces immutability where appropriate—rule definitions, rule sets, and metadata are copied defensively—while contexts remain mutable to support orchestrating state. Avoid caching the context outside the evaluation call; treat it as request-scoped.

## Threading model

- `RuleEngine` is stateless and thread-safe. Share a single instance across requests.
- `RuleExecutionContext` is created per rule evaluation and must not be reused.
- `RuleActionRegistry` / `RuleHookRegistry` are static singletons; register contributions during application bootstrap before evaluating any rules.

## Error handling

- If a stage throws an exception, the engine records it on the relevant `RuleDebugStageTrace` (when debugging) and propagates the exception.
- Actions should surface domain errors by throwing runtime exceptions; upstream services can translate them into HTTP/gRPC responses.
- Hooks should avoid throwing where possible—log and continue to keep rule evaluation deterministic.
