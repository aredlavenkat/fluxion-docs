# `$httpCall`

Performs an HTTP request during pipeline evaluation. Supports dynamic URL/parameter
substitution, custom headers, payloads, and optional resilience settings (retry,
circuit breaker).

**Usage scope:** `$httpCall` is an expression operator. Use it inside a stage such as `$addFields`/`$set` and assign the result into your document:

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

---

## 1. Prerequisites

| Requirement | Notes |
| --- | --- |
| Dependency | `ai.fluxion:fluxion-connect` (contains enrichment operators and HTTP sink). |
| Connection registry | Optional `HttpConnector`/connection registry entry for shared base URLs/auth. |
| Resilience | Resilience4j (optional) for retry/circuit breaker configuration. |

---

## 2. Syntax

```json
{
  "$httpCall": {
    "method": "POST",
    "connection": "userService",
    "url": "https://profiles/api/users/{id}",
    "params": { "id": "$user.id" },
    "headers": {
      "X-Trace-Id": "$request.traceId"
    },
    "body": {
      "email": "$user.email"
    },
    "responsePath": "/data",
    "retry": { "maxAttempts": 3, "waitDurationMs": 50 },
    "circuitBreaker": { "name": "user-service-breaker", "failureRateThreshold": 30 }
  }
}
```

---

## 3. Options

| Field | Description | Default |
| --- | --- | --- |
| `connection` | Name of registered `HttpConnector`. Required (provides base URL/auth/resilience defaults). | **required** |
| `url` | Request URL template. `{}` placeholders replaced by `params`. | Connector URL |
| `method` | HTTP verb (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`, etc.). | `GET` |
| `headers` | Map of header expressions (evaluated per request). | `{}` |
| `params` | Map of query/path parameters. | `{}` |
| `body` | Expression evaluated for the request body (JSON encoded if object). | `null` |
| `responsePath` | JSON Pointer applied to the parsed response body. | Entire payload |
| `connectTimeoutMs` | Connection timeout in milliseconds. | `5000` |
| `readTimeoutMs` | Read timeout in milliseconds. | `5000` |
| `retry` | Resilience4j retry config (see [Resilience Patterns](../../shared/resilience.md)); supports `retryOn` / `ignore` exception lists. | Disabled |
| `circuitBreaker` | Resilience4j circuit breaker config. | Disabled |

Resilience layers:
- **Connection-level**: define `retry`/`circuitBreaker` on the HTTP connector; all `$httpCall` usages inherit them.
- **Call-level overrides**: set `retry` / `circuitBreaker` in the operator to override defaults for a single call.

---

## 4. Examples

### Simple GET with query params

```json
{
  "$httpCall": {
    "method": "GET",
    "url": "https://api.weather/v1/city",
    "params": { "zip": "$address.zip" }
  }
}
```

### POST with connector + JSON body

```json
{
  "$httpCall": {
    "connection": "userService",
    "method": "POST",
    "url": "/users/{id}",
    "params": { "id": "$user.id" },
    "headers": { "X-Request-Id": "$request.traceId" },
    "body": {
      "email": "$user.email",
      "status": "$user.status"
    },
    "responsePath": "/data"
  }
}
```

---

## 5. Response handling

- Responses are parsed as JSON. Arrays/primitives are preserved.
- If `responsePath` is provided, the JSON Pointer is resolved to select part of
  the payload.
- HTTP status codes ≥ 400 throw an exception; retry/breaker logic handles the
  exception if configured.

---

## 6. Connectors

Register connectors once and reference them by name:

```java
ConnectorManager.register("userService", new HttpConnector(
    "https://profiles/api/users",
    Map.of("Authorization", "Bearer ${TOKEN}")
));
```

Pipelines can override headers or provide additional path/query parameters.

---

## 7. Troubleshooting

| Symptom | Possible cause | Remedy |
| --- | --- | --- |
| `IllegalArgumentException: url missing` | Neither `url` nor connector base URL provided. | Supply `url` or configure the connector with a base URL. |
| Timeout exceptions | Slow downstream service. | Increase `connectTimeoutMs`/`readTimeoutMs` or configure retries. |
| `JsonProcessingException` | Response body not JSON. | Wrap operator in `$function` to handle plain text/binary payloads. |
| Circuit breaker always open | Shared breaker between unrelated endpoints. | Use unique breaker names per service. |

---

## 8. Testing

- Run enrichment tests:
  ```bash
  mvn -pl fluxion-connect -am test -Dtest=*HttpCall*
  ```
- Use mock web servers (e.g., OkHttp MockWebServer) to simulate behaviour during CI.

---

## 9. References

| Path | Description |
| --- | --- |
| `fluxion-connect/src/main/java/.../HttpCallOperator.java` | Operator implementation. |
| `fluxion-connect/src/test/java/.../HttpCallOperatorTest.java` | Unit/integration tests. |
| [Resilience Patterns](../../shared/resilience.md) | Retry/circuit breaker configuration. |

Use `$httpCall` for declarative service calls in both rule and streaming pipelines.
