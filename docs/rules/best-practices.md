# Best Practices

These guidelines reflect lessons learned from running the rule engine in production. Following them results in predictable behaviour, clear authoring workflows, and maintainable rule libraries.

## Design principles

### 1. Keep salience ranges meaningful

- Reserve broad ranges for major categories (e.g. `1000-1999` for blocking rules, `500-999` for warnings, `0-499` for enrichment).
- Avoid reusing salience values—duplicate salience now triggers validation errors, but a deliberate scheme makes conflicts rare.

### 2. Name rules for intent

- Use verbs and nouns (`"Flag high value order"`, `"Allow loyalty member"`).
- Match IDs and metadata to your change-management system (`"order.flag.highValue"`).
- Include owner team in rule-set metadata (`"owner": "payments-risk"`).

### 3. Keep pipelines short and testable

- Prefer multiple small stages over one complex `$expr`. It improves readability and aids debug traces.
- If you need complex transformations, perform them upstream using Fluxion Enrich or dedicated services.

### 4. Treat actions as idempotent

- Actions may run again if a service retries after partial failure. Ensure side effects (notifications, database writes) can handle duplicates.
- Use context attributes to guard repeated work when necessary.

### 5. Centralise shared data in `sharedAttributes`

- Rule-set hooks can pre-compute data, such as customer segments, and store them in `sharedAttributes` for downstream rules.
- Document the keys you use so actions/hooks written by other teams can rely on them.

## Authoring workflow

1. Prototype the rule locally using the [Quick Start](quickstart.md) steps.
2. Commit the DSL file alongside unit tests that cover positive and negative cases.
3. During review, include lint output (`parseWithLints`) and example passes (see [Testing & Debugging](testing.md)).
4. Publish rule-set metadata describing owner, escalation path, and version.

## Actions and hooks

- Register actions at application bootstrap and avoid removing them at runtime. If an action name disappears, parsing will fail.
- For SPI-provided actions/hooks, package the contributor classes and service descriptors in the same jar to simplify deployment.
- Use hooks for cross-cutting concerns (audit, telemetry) and keep actions focused on decision outcomes.

## Debugging tips

- Enable debug mode only when diagnosing an issue; truncate traces before logging in production to reduce sensitive data exposure.
- When a rule fails unexpectedly, inspect `RuleExecutionContext.stageMetrics()` to see the pipeline throughput.
- Use `RuleDebugStageTrace.filtered()` to quickly spot the stage that removed a document.

## Performance considerations

- Rule evaluation is synchronous; if you run expensive actions, offload them to asynchronous queues.
- Cache rule sets in memory; rebuilding them per request wastes CPU and re-triggers validation.
- Use `RuleActionRegistry.reload()` in tests, not production. Production services should register actions explicitly on startup.

## Documentation hygiene

- Link each rule to runbooks or domain docs via metadata (e.g. `"runbookUrl"`).
- Maintain a changelog per rule set that references version numbers or git commits.
- Encourage authors to run an automated documentation generator that extracts rule summaries—this page can serve as the template.
