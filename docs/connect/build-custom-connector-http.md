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
- Keep secrets out of manifests; resolve via `ConnectorContext::secretResolver`.
- Resilience layers:
  - **Connection-level** (HTTP connection config/manifest) via `retry` / `circuitBreaker` defaults; all `$httpCall` uses inherit them.
  - **Call-level overrides** in the `$httpCall` expression (`retry`, `circuitBreaker`) for per-call tuning.
  - `retry` supports `retryOn` / `ignore` lists (exception class names) to control which errors trigger retries.

## Full example (action + pipeline usage)

Manifest (`src/main/resources/manifests/http-action.json`):
```json
{
  "schemaVersion": "1.0.0",
  "id": "demo.http",
  "version": "1.0.0",
  "operations": { "ship": { "$ref": "#/operationDefs/ship" } },
  "operationDefs": {
    "ship": {
      "operationId": "ship",
      "kind": "action",
      "inputSchema": { "type": "object", "properties": { "orderId": { "type": "string" } } },
      "outputSchema": { "type": "object" },
      "execution": {
        "type": "http",
        "method": "POST",
        "urlTemplate": "https://api.example.com/ship",
        "timeoutMs": 8000,
        "headers": { "Authorization": "Bearer {{SECRET_TOKEN}}" },
        "retry": {
          "maxAttempts": 3,
          "waitDurationMs": 100,
          "retryOn": ["java.io.IOException"],
          "ignore": ["java.net.UnknownHostException"]
        }
      }
    }
  }
}
```

Dispatcher call:
```java
ConnectorManifest manifest = new ConnectorManifestLoader()
    .load(Path.of("src/main/resources/manifests/http-action.json"));
Object resp = dispatcher.executeAction(manifest, "ship", ctx, Map.of("orderId", "A-123"));
```

Pipeline usage (expression inside a stage):
```json
{
  "$set": {
    "shippingResponse": {
      "$enrich": {
        "connectionRef": "demo.http",
        "connectorId": "demo.http",
        "operationId": "ship",
        "input": { "orderId": "$order.id" }
      }
    }
  }
}
```
