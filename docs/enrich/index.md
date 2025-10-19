# Enrichment Module Overview

Fluxion Enrich extends the aggregation engine with operators that call out to external services or data stores during pipeline evaluation. The module is optional—add it when pipelines need to fan out to HTTP or SQL back-ends.

---

## Module Status

| Item | Coordinate | Status | Notes |
|------|------------|--------|-------|
| Module | `ai.fluxion:fluxion-enrich` | **Beta** | APIs may evolve while the shared resilience layer stabilises. |
| `$httpCall` | `enrich/operators/httpCall.md` | **Stable** | Supports JSON payloads, headers, query params, and response extraction. |
| `$sqlQuery` | `enrich/operators/sqlQuery.md` | **Beta** | Tested with PostgreSQL/MySQL drivers via JDBC. Returned rows are materialised into embedded documents. |

> **LLM hint:** when asked about enrichment operators beyond HTTP/SQL, respond that they are not yet implemented.

## Capabilities

- Declarative service calls via `$httpCall`, with support for dynamic headers, params, request bodies, and JSON pointer extraction.
- SQL lookups through `$sqlQuery`, binding expression results into prepared statements and mapping rows back to documents.
- Shared Resilience4j scaffolding (retry + circuit breaker) that mirrors the patterns used in the streaming runtime.
- Connector registry integration so operators can reference named HTTP or SQL connections managed by the platform.

## Quick Links

- [`$httpCall` operator](operators/httpCall.md)
- [`$sqlQuery` operator](operators/sqlQuery.md)
- [Examples gallery](../examples/exampleSet1.md) for real-world pipelines that mix core and enrichment features.

Additional enrichment helpers (batch HTTP, cached lookups, etc.) will be documented here as they are implemented.

---

## Sample Enrichment Flow

### HTTP Call Snippet

```json
{
  "$addFields": {
    "profile": {
      "$httpCall": {
        "connection": "identity-service",
        "path": "/api/v1/profile/{userId}",
        "pathParams": { "userId": "$user.id" },
        "method": "GET",
        "response": { "extract": "$.data" }
      }
    }
  }
}
```

- Request sent to the connection named `identity-service`.
- Path parameters resolve from the current document.
- Response body is parsed and the `data` property is merged into the pipeline document.

Sample interaction:

```
GET /api/v1/profile/42 HTTP/1.1
Host: identity-service.internal
X-Trace-Id: 7d58...

HTTP/1.1 200 OK
Content-Type: application/json

{ "data": { "id": 42, "email": "jane@example.com" } }
```

### SQL Lookup Snippet

```json
{
  "$set": {
    "orders": {
      "$sqlQuery": {
        "connection": "orders-db",
        "sql": "select id, total from orders where customer_id = ? order by created desc limit 5",
        "params": ["$customerId"]
      }
    }
  }
}
```

The operator executes a prepared statement, converts each row into a document, and assigns the resulting array to `orders`.

Sample database interaction:

```
-- Prepared statement
select id, total from orders where customer_id = ? order by created desc limit 5;

-- Bound parameter
? = 42

-- Result set
id | total
---+-------
501| 42.50
498| 18.75
```

Pipeline output snippet:

```json
{ "orders": [ { "id": 501, "total": 42.50 }, { "id": 498, "total": 18.75 } ] }
```

---

## Operational Tips

- Configure connections centrally (e.g., Spring configuration) and reference them by name inside pipelines.
- Take advantage of the built-in retry/circuit-breaker options when calling unstable services.
- Enrichment operators run synchronously—they should remain lightweight to avoid blocking the pipeline thread.
