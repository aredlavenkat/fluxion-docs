# Build Custom Connector: HTTP Action

- **Manifest:** `execution.type=http` with `method`, `urlTemplate`, headers, retry/CB.
- **SDK:** `dispatcher.executeAction(manifest, op, ctx, body)`.
- **When to customize:** auth headers, retry/CB config, templating.
