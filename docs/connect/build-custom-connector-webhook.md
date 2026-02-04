# Build Custom Connector: Webhook Trigger

- **Manifest:** `execution.type=webhook`, `path`, `method`.
- **SDK:** `dispatcher.startTrigger(manifest, op, ctx, Map.of("port", 8080))` returns a Flux of payloads. The dispatcher hosts the HTTP listener.
