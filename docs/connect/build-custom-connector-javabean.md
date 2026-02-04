# Build Custom Connector: JavaBean Action/Trigger

**When to use:** custom logic inside your service (SDKs, DB drivers, bespoke
code) without writing a new transport.

## Manifest
```json
"execution": { "type": "javaBean", "beanName": "myHandler" }
```

## SDK
```java
dispatcher.registerActionHandler("myHandler", (c, in) -> Map.of("ok", true));
dispatcher.executeAction(manifest, "myOp", ctx, Map.of());
```

## Trigger variant
Register the same bean as a trigger handler:
```java
dispatcher.registerTriggerHandler("myHandler", (c, cfg) -> Flux.just(Map.of("tick", 1)));
dispatcher.startTrigger(manifest, "myOp", ctx, Map.of());
```***
