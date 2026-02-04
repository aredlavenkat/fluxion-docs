# Build Custom Connector: Pipeline Call Action

**When to use:** invoke another pipeline/version from within an action.

## Manifest
```json
"execution": { "type": "pipelineCall", "targetPipeline": "orders", "version": "v1" }
```

## SDK
Register an invoker (or a default one):
```java
dispatcher.registerPipelineInvoker("orders", (id, ver, c, input) -> callOrders(id, ver, input));
dispatcher.executeAction(manifest, "someOp", ctx, Map.of("orderId", "A-1"));
```***
