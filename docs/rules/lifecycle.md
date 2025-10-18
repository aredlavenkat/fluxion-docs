# Lifecycle & Governance

The rule engine is often used in regulated domains where auditability and change control matter. This guide outlines how to manage rule versions from draft to production.

## Artefact lifecycle

1. **Draft** – rules are authored in feature branches or sandbox environments. Validation runs locally.
2. **Review** – lints must pass; peer review focuses on correctness, salience, and metadata quality.
3. **Approval** – sign-off from domain owners. Record the approval in rule metadata (`"approvedBy"`, `"ticket"`).
4. **Deployment** – rule sets are packaged with application releases or delivered to a rule registry.
5. **Monitoring** – track match rates, shared attributes, and action outcomes. Compare against baselines.
6. **Retirement** – mark rules as deprecated, remove actions/hooks, and clean up metadata once retired.

## Versioning strategy

- Use semantic versions on rule sets (`1.2.0`). Bump:
  - MAJOR when salience ordering or semantics change dramatically.
  - MINOR when adding/removing rules or metadata.
  - PATCH for action/hook tweaks that keep behaviour equivalent.
- Store previous versions for auditing; `RuleSet` instances are immutable so you can snapshot them easily.

## Audit trails

- Persist rule JSON and metadata for each deployment (e.g. in object storage or git).
- Log rule evaluations with rule ID, salience, match outcome, and shared attributes.
- Use `RuleDebugStageTrace` in incident investigations; archive traces only when necessary because they may contain PII.

## Change management metadata

Embed governance fields inside `metadata`:

```json
"metadata": {
  "owner": "payments-risk",
  "approvedBy": "alice@example.com",
  "ticket": "RISK-2042",
  "runbookUrl": "https://wiki/rules/high-value"
}
```

Make the owning service surface this metadata in dashboards or change logs.

## Deployment patterns

- **Config service** – store rule sets in a central service; services fetch and cache them. Provide a diff endpoint for tooling.
- **Embedded jar** – bundle rules with application code. Rebuild and redeploy to roll out changes; keep release notes comprehensive.
- **Hybrid** – embed a baseline ruleset and allow overrides from a registry at runtime. Ensure validation runs on both sources.

## Rollback plan

- Keep the previous `RuleSet` in memory. If monitoring flags regressions, swap back instantly.
- Version metadata should include `supersedes` or `previousVersion` so tooling can automate rollback selection.

## Multitenancy

If you host rules for multiple teams or customers:

- Namescope rule IDs (`tenant.ruleId`) and salience ranges per tenant.
- Consider separate `RuleEngine` instances or partitions to keep shared attributes isolated.
- Enforce quotas (number of rules, evaluation latency) per tenant at the orchestration layer.

Governance ensures the rule engine remains predictable, audited, and compliant with organisational policies.
