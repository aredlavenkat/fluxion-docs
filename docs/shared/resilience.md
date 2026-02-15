# Resilience Patterns (Retry & Circuit Breaker)

SrotaX leverages Resilience4j across enrichment operators (`$httpCall`,
`$sqlQuery`) and streaming connectors. Enrichment now ships inside
`fluxion-connect` alongside the HTTP/Kafka connectors, so retry/breaker settings
are handled in one place. This guide consolidates the available options, usage
tips, and testing helpers.

---

## 1. Retry options

| Field | Meaning | Default |
| --- | --- | --- |
| `name` | Identifier for the retry instance. Pipelines sharing the same name also share retry state. | Derived from operator/connector name |
| `maxAttempts` | Total attempts (initial call + retries). | `1` |
| `waitDurationMs` | Fixed backoff between attempts (milliseconds). | `1000` |
| `multiplier` | > 1.0 enables exponential backoff (`waitDurationMs` as base). | `1.0` |
| `retryOn` | List of exception class names to include. | `[]` |
| `ignore` | Exceptions to fail fast without retrying. | `[]` |

### Example

```json
{
  "retry": {
    "name": "orders-db-retry",
    "maxAttempts": 3,
    "waitDurationMs": 50,
    "multiplier": 2.0,
    "retryOn": ["java.sql.SQLTransientException"],
    "ignore": ["java.sql.SQLException"]
  }
}
```

---

## 2. Circuit breaker options

| Field | Meaning | Default |
| --- | --- | --- |
| `enabled` | Enables/disables the breaker. | `true` |
| `name` | Identifier for the breaker instance. Shared names share state. | Derived from operator/connector name |
| `failureRateThreshold` | Percentage of failures that opens the breaker. | `50` |
| `slowCallRateThreshold` | Percentage of slow calls counted as failures. | `100` |
| `slowCallDurationThresholdMs` | Milliseconds defining a slow call. | `60000` |
| `waitDurationInOpenStateMs` | Time to stay OPEN before half-open probes. | `10000` |
| `permittedCallsInHalfOpenState` | Trial calls while HALF_OPEN. | `2` |
| `minimumNumberOfCalls` | Minimum sample size before metrics apply. | `10` |
| `slidingWindowSize` | Size of metrics window. | `100` |
| `slidingWindowType` | `0` for count-based, `1` for time-based. | `1` |
| `record` / `ignore` | Additional exception class names to record/ignore. | `[]` |

### Example

```json
{
  "circuitBreaker": {
    "name": "user-service-breaker",
    "failureRateThreshold": 30,
    "slowCallRateThreshold": 50,
    "slowCallDurationThresholdMs": 2000,
    "waitDurationInOpenStateMs": 30000,
    "minimumNumberOfCalls": 5,
    "permittedCallsInHalfOpenState": 3,
    "slidingWindowSize": 20,
    "slidingWindowType": 0
  }
}
```

---

## 3. Usage guidelines

- Keep retries low for idempotent operations; avoid retrying non-idempotent
  writes without compensating actions.
- Use unique `name` values per downstream system to isolate metrics; reuse names
  only when pipelines intentionally share state.
- Combine retry + circuit breaker for noisy/slow dependencies (retry handles
  transient spikes, breaker protects against sustained failure).
- Surface breaker events to your monitoring/alerting stack (Resilience4j emits
  metrics you can wire into Micrometer/Prometheus).

---

## 4. Testing utilities

- `$httpCall`: `HttpCallOperator.resetResilience()` resets retry/breaker state in
  tests.
- `$sqlQuery`: `SqlQueryOperator.resetResilience()` does the same for SQL.
- Streaming connectors expose similar reset hooks if needed (e.g., Kafka).

Run connector/enrichment tests to validate configuration:

```bash
mvn -pl fluxion-connect -am test
```

---

## 5. References

| Doc | Notes |
| --- | --- |
| [`$httpCall`](../enrich/operators/httpCall.md) | Shows resilience usage in HTTP calls. |
| [`$sqlQuery`](../enrich/operators/sqlQuery.md) | SQL enrichment with retry/breaker. |
| Resilience4j | [Official docs](https://resilience4j.readme.io/docs). |

Apply these settings consistently across operators and connectors to tune
resilience without scattering configuration details.
