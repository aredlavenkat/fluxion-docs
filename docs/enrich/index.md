# Enrichment Operators Overview

The enrichment operators ship with `fluxion-connect` and let pipelines call
external services or data stores during execution. Use them when a pipeline
needs to fetch HTTP resources or run SQL queries inline.

---

## 1. Prerequisites

| Requirement | Notes |
| --- | --- |
| SrotaX modules | `fluxion-core`, `fluxion-connect` (contains enrichment + connectors). |
| HTTP/SQL back-ends | Services or databases reachable from your pipelines. |
| Configuration | Named connections (e.g., Spring beans) for HTTP and SQL targets. |
| Resilience layer | Optional Resilience4j dependencies for retries/circuit breakers (recommended). |

---

## 2. Module status

| Item | Doc | Status | Notes |
| --- | --- | --- | --- |
| Module | `ai.fluxion:fluxion-connect` (enrichment packaged here) | **Beta** | APIs may evolve alongside the shared resilience layer. |
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
| `fluxion-connect/src/main/java/.../$httpCall` | Implementation of the HTTP operator. |
| `fluxion-connect/src/main/java/.../$sqlQuery` | JDBC-backed SQL operator. |
| `https://docs.srotax.com/enrich/operators/httpCall/` | Detailed HTTP options with examples. |
| `https://docs.srotax.com/enrich/operators/sqlQuery/` | Detailed SQL options with examples. |
| `https://docs.srotax.com/examples/` | Pipelines mixing core and enrichment features. |

Run enrichment tests with:

```bash
mvn -pl fluxion-connect -am test
```

This validates HTTP/SQL operators and shared resilience components.

---

## 8. Building a custom enrichment operator

Enrichment operators are expression operators registered via `OperatorContributor`. They return a value; embed them inside a stage such as `$addFields`/`$set` to write the result into your document.

Example: `FooLookup` operator that calls an external service through the shared enrichment transport:

```java
package ai.fluxion.enrich.evaluator.operators.foo;

import ai.fluxion.core.engine.ExecutionContext;
import ai.fluxion.core.engine.enrichment.EnrichmentTransport;
import ai.fluxion.core.expression.Context;
import ai.fluxion.core.expression.Operator;
import ai.fluxion.core.expression.OperatorContributor;

import java.util.Map;
import java.util.ServiceLoader;

public final class FooLookupOperator implements OperatorContributor, Operator {
    private final EnrichmentTransport transport = ServiceLoader.load(EnrichmentTransport.class)
            .findFirst()
            .orElseThrow(() -> new IllegalStateException("No EnrichmentTransport found (add fluxion-connect)"));

    @Override
    public Map<String, Operator> contributors() {
        return Map.of("$fooLookup", this);
    }

    @Override
    public Object apply(Context ctx, Object rawConfig) {
        FooConfig cfg = FooConfig.from(rawConfig);
        var req = EnrichmentTransport.HttpRequest.builder()
                .connectionRef(cfg.connection())
                .method("GET")
                .url(cfg.url(ctx))
                .headers(cfg.headers(ctx))
                .build();
        var resp = transport.executeHttp(req, ExecutionContext.current());
        return cfg.extract(resp.body()); // select a sub-field or return the whole payload
    }
}
```

Register it via ServiceLoader:

```
# src/main/resources/META-INF/services/ai.fluxion.core.expression.OperatorContributor
ai.fluxion.enrich.evaluator.operators.foo.FooLookupOperator
```

Use it in a pipeline:

```json
{
  "$addFields": {
    "foo": {
      "$fooLookup": {
        "connection": "fooService",
        "url": "https://api.example.com/foo/{id}",
        "params": { "id": "$entity.id" },
        "responsePath": "/body/data"
      }
    }
  }
}
```
