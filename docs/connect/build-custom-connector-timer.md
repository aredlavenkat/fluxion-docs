# Build Custom Connector: Timer Trigger

**When to use:** cron/interval ticks that emit simple events into the pipeline.

## Manifest
```json
"execution": { "type": "timer", "cron": "PT30S" }
```

## SDK
```java
dispatcher.startTrigger(manifest, "tick", ctx, Map.of());
```
