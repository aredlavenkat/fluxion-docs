# Build Custom Connector: Timer Trigger

- **Manifest:** `execution.type=timer`, `cron` (ISO-8601 duration or cron string).
- **SDK:** `dispatcher.startTrigger(manifest, op, ctx, Map.of())` emits ticks.
