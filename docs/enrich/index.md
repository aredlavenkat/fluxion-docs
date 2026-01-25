# Enrichment Module Overview

SrotaX Enrich adds operators that call external services or data stores during
pipeline execution. Use it when a pipeline needs to fetch HTTP resources or run
SQL queries inline.

---

## 1. Prerequisites

| Requirement | Notes |
| --- | --- |
| SrotaX modules | `fluxion-core`, `fluxion-enrich`; optionally `fluxion-connect` for connector integration. |
| HTTP/SQL back-ends | Services or databases reachable from your pipelines. |
| Configuration | Named connections (e.g., Spring beans) for HTTP and SQL targets. |
| Resilience layer | Optional Resilience4j dependencies for retries/circuit breakers (recommended). |

---

## 2. Module status

| Item | Doc | Status | Notes |
| --- | --- | --- | --- |
| Module | – | **Beta** | APIs may evolve alongside the shared resilience layer. |
| `$httpCall` | [operators/httpCall.md](operators/httpCall.md) | **Stable** | JSON payloads, headers, query params, response extraction. |
| `$sqlQuery` | [operators/sqlQuery.md](operators/sqlQuery.md) | **Beta** | Prepared statements via JDBC (PostgreSQL/MySQL verified). |

> If asked about additional enrichment operators, state that only HTTP/SQL are
> available today and reference the SPI documentation for future extensions.

---

## 3. Capabilities

- `$httpCall` – Declarative HTTP requests with dynamic path/headers/body and
  JSON pointer extraction for responses.
- `$sqlQuery` – Parameterised SQL lookups mapped back into embedded documents.
- Shared Resilience4j scaffolding (retry + circuit breaker) mirroring the
  streaming runtime.
- Connection registry integration so pipelines can reference centrally managed
  connection definitions.

---

## 4. Usage patterns

### HTTP call

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

- `connection` refers to a named HTTP client configured in your application.
- Path parameters resolve from the current document.
- `response.extract` uses JSON Pointer to select part of the payload.

### SQL lookup

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

- Prepared statement executed against the named JDBC connection.
- Parameters can be literal values or expressions evaluated per document.
- Result set is converted into an array of documents.

---

## 5. Configuration table

| Field | Operator | Description |
| --- | --- | --- |
| `connection` | HTTP/SQL | Logical connection name resolved by your app. |
| `method` | HTTP | HTTP verb (`GET`, `POST`, …). |
| `path`, `pathParams` | HTTP | URL templates and parameter bindings. |
| `headers`, `query`, `body` | HTTP | Optional request metadata and payload. |
| `response.extract` | HTTP | JSON pointer to select part of the response. |
| `sql` | SQL | Prepared statement text. |
| `params` | SQL | Array of bound parameters (expressions supported). |
| `rowMapper` (future) | SQL | Hook for custom row mapping. |

Configure connections via your DI framework (Spring beans, Micronaut singletons,
manual registries) and reference by `connection` name inside pipeline definitions.

---

## 6. Operational guidance

- Keep enrichment calls lightweight to avoid blocking pipeline threads.
- Reuse resilience policies (retry, circuit breaker) to insulate downstream
  services. `$httpCall` and `$sqlQuery` accept resilience configuration blocks.
- Cache responses if the same upstream data is fetched frequently.
- Monitor outbound call latency and error rates alongside pipeline metrics.

---

## 7. References

| Path | Description |
| --- | --- |
| `fluxion-enrich/src/main/java/.../$httpCall` | Implementation of the HTTP operator. |
| `fluxion-enrich/src/main/java/.../$sqlQuery` | JDBC-backed SQL operator. |
| `https://docs.srotax.com/enrich/operators/httpCall/` | Detailed HTTP options with examples. |
| `https://docs.srotax.com/enrich/operators/sqlQuery/` | Detailed SQL options with examples. |
| `https://docs.srotax.com/examples/` | Pipelines mixing core and enrichment features. |

Run enrichment tests with:

```bash
mvn -pl fluxion-enrich -am test
```

This validates HTTP/SQL operators and shared resilience components.
