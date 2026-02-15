# Build Custom Connector: HTTP Action

**When to use:** outbound HTTP calls with optional retry/circuit-breaker and
templating. The HTTP action uses the same registry/resilience wiring as the
HTTP sink and `$httpCall` enrichment operator.

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
- Prefer `connectionRef` + registry-backed connections for base URLs, headers,
  auth, and resilience defaults; override per-operation if needed.
- Keep secrets out of manifests; resolve via `ConnectorContext::resolveSecret`
  or external connection loaders.
- Resilience layers:
  - **Connection-level**: set `retry` / `circuitBreaker` on the connection
    definition; all `$httpCall` or connector operations inherit them.
  - **Operation-level overrides**: set `retry` / `circuitBreaker` inside the
    manifest for this operation only.
  - `retry` supports `retryOn` / `ignore` lists (exception class names) to
    control which errors trigger retries.

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
      "$httpCall": {
        "connection": "demo.http",
        "path": "/ship",
        "method": "POST",
        "body": { "orderId": "$order.id" }
      }
    }
  }
}
```
