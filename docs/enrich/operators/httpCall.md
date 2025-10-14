# `$httpCall`

Performs an HTTP request during pipeline evaluation. Requests can target a literal URL or reuse a named `HttpConnector` for shared configuration.

## Syntax

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

## Options

| Field | Description | Default |
| --- | --- | --- |
| `connection` | Name of the registered `HttpConnector` providing base URL, headers, or auth. Optional when `url` is supplied. | `null` |
| `url` | Request URL template. Path parameters wrapped in `{}` are substituted with resolved `params`. Required if connection does not specify a URL. | Connector URL |
| `method` | HTTP method (GET, POST, PUT, PATCH, DELETE, etc.). | `GET` |
| `headers` | Map of header names to expressions. Resolved per request. | `{}` |
| `params` | Key/value expressions appended as query string parameters and used for `{param}` replacements. | `{}` |
| `body` | Expression resolved to build the request body. Objects are JSON encoded automatically. | `null` |
| `responsePath` | JSON Pointer applied to the decoded response body (e.g., `/data/items/0`). | Entire response |
| `connectTimeoutMs` | Connection timeout in milliseconds. | `5000` |
| `readTimeoutMs` | Read timeout in milliseconds. | `5000` |
| `retry` | Resilience4j retry configuration (see [Resilience Patterns](../../shared/resilience.md)). | Disabled |
| `circuitBreaker` | Resilience4j circuit breaker configuration (see [Resilience Patterns](../../shared/resilience.md)). | Disabled |

> ℹ️ Refer to the shared [Resilience Patterns](../../shared/resilience.md) page for the full list of retry and circuit-breaker fields with defaults.

## Response Handling

- Responses are parsed as JSON. Primitive or array responses are preserved.
- When `responsePath` is supplied, the pointer is resolved against the parsed payload.
- HTTP status codes ≥ 400 cause the operator to throw, participating in retry/breaker logic if enabled.

## Connectors

`HttpConnector` instances allow you to centralise base URLs, default headers, and authentication tokens:

```java
ConnectorManager.register("userService", new HttpConnector(
    "https://profiles/api/users",
    Map.of("Authorization", "Bearer ${TOKEN}")
));
```

Operators reference the connector by name, layering additional headers or overriding the URL as needed.

## Resilience

Supplying a unique `name` lets multiple pipelines share the same Resilience4j state. See [Resilience Patterns](../../shared/resilience.md) for configuration details and best practices.
