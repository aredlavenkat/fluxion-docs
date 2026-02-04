# Build Custom Connector: HTTP Action

**When to use:** outbound HTTP calls with optional retry/circuit-breaker and
templating.

## Manifest
```json
{
  "operations": { "call": "call" },
  "operationDefs": {
    "call": {
      "kind": "action",
      "execution": {
        "type": "http",
        "method": "POST",
        "urlTemplate": "https://api.example.com/orders",
        "timeoutMs": 8000
      }
    }
  }
}
```

## SDK
```java
Object out = dispatcher.executeAction(manifest, "call", ctx, Map.of("orderId", "A-1"));
```

## Notes
- Headers can carry retry/CB configs (or use manifest fields).
- Keep secrets out of manifests; resolve via `ConnectorContext::secretResolver`.***
