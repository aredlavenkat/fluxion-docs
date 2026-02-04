# `$enrich`

Generic enrichment operator that invokes a manifest-backed connector operation
by reference. Use it when no specialized operator exists (e.g., for custom
connectors or non-HTTP/SQL actions).

**Usage scope:** place inside a stage (`$set`/`$addFields`) and assign the result
into your document.

## Syntax
```json
{
  "$enrich": {
    "connectionRef": "demo.http",
    "connectorId": "demo.http",
    "operationId": "ship",
    "input": { "orderId": "$order.id" }
  }
}
```

## Fields

| Field | Description | Required |
| --- | --- | --- |
| `connectionRef` | Name of the connection to use (from manifest/registry). | Yes |
| `connectorId` | Connector manifest id. Often same as `connectionRef`. | Yes |
| `operationId` | Operation within the connector manifest. | Yes |
| `input` | Map/expression passed as the operation input. | Optional |
| `retry` | Per-call retry config (if supported by the target connector). | Optional |
| `circuitBreaker` | Per-call circuit breaker config (if supported by the target connector). | Optional |

Notes:
- `$enrich` is transport-agnostic; it simply dispatches to the manifest/registry.
- Use `$httpCall`/`$sqlQuery` when available for tighter validation; fall back to `$enrich` for everything else.
- Resilience is respected if the underlying connector supports it (e.g., HTTP/SQL connectors allow connection-level or call-level `retry`/`circuitBreaker`).
- Triggers like polling/webhook/timer/JavaBean do not have built-in retry/CB knobs; add resilience in your handler code if needed.

## Example (HTTP action via manifest)

Manifest (simplified):
```json
{
  "id": "demo.http",
  "operations": { "ship": "ship" },
  "operationDefs": {
    "ship": {
      "kind": "action",
      "execution": { "type": "http", "method": "POST", "urlTemplate": "https://api.example.com/ship" }
    }
  }
}
```

Pipeline usage:
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

## Error handling
- Propagates connector errors (HTTP 4xx/5xx, SQL errors, etc.) unless the underlying connector/operator applies retry/CB.
- For resilience on HTTP/SQL, prefer `$httpCall`/`$sqlQuery` or configure retry/CB on the connection/operation itself.
