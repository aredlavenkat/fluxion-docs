# Build Custom Connector: JavaBean Action/Trigger

- **Manifest:** `execution.type=javaBean`, `beanName`.
- **SDK:**
```java
dispatcher.registerActionHandler("myHandler", (c, in) -> Map.of("ok", true));
dispatcher.executeAction(manifest, "myOp", ctx, Map.of());
```
- **Trigger:** same beanName with `startTrigger(...)`.
