# Build Custom Connector: Polling Trigger

**When to use:** periodic pulls from systems that don’t push events.

## Manifest
```json
"execution": { "type": "polling", "intervalMillis": 30000, "handlerBean": "poller" }
```

## SDK
```java
dispatcher.registerTriggerHandler("poller",
    (c, cfg) -> Flux.interval(Duration.ofSeconds(30)).map(i -> Map.of("tick", i)));
dispatcher.startTrigger(manifest, "poll", ctx, Map.of());
```

If `handlerBean` is omitted, the dispatcher emits ticks by interval.***
