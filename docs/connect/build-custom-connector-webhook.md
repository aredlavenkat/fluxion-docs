# Build Custom Connector: Webhook Trigger

**When to use:** inbound HTTP/webhook events that should emit payloads into the
pipeline.

## Manifest
```json
"execution": { "type": "webhook", "path": "/webhook", "method": "POST" }
```

## SDK
```java
Flux<Map<String,Object>> flux =
    dispatcher.startTrigger(manifest, "ingest", ctx, Map.of("port", 8080));
```

The dispatcher hosts the HTTP listener; payloads are emitted as a Flux.***
