# `$sqlQuery`

Executes a parameterised SQL statement against a `SqlConnector` that has been registered with the enrichment connector registry.

## Syntax

```json
{
  "$sqlQuery": {
    "connection": "ordersDb",
    "query": "SELECT * FROM orders WHERE user_id = ?",
    "params": ["$user.id"],
    "expectSingle": true,
    "retry": {
      "name": "orders-db-retry",
      "maxAttempts": 3,
      "waitDurationMs": 25,
      "multiplier": 2.0
    },
    "circuitBreaker": {
      "name": "orders-db-breaker",
      "failureRateThreshold": 25,
      "minimumNumberOfCalls": 4,
      "waitDurationInOpenStateMs": 30000
    }
  }
}
```

## Options

| Field | Description | Default |
| --- | --- | --- |
| `connection` | Name of the `SqlConnector` registered in `ConnectorManager`. | **Required** |
| `query` | SQL string to execute. Use positional `?` parameters for binding. | **Required** |
| `params` | Array of expressions resolved before execution and bound in order. | `[]` |
| `expectSingle` | When `true`, returns only the first row (or `null` when no rows). Otherwise returns the entire result set as a list of maps. | `false` |
| `name` | Friendly name used to derive retry/breaker ids when none are supplied. | Derived from connection + query hash |
| `retry` | Resilience4j retry configuration (see [Resilience Patterns](../../shared/resilience.md)). | Disabled |
| `circuitBreaker` | Resilience4j circuit breaker configuration (see [Resilience Patterns](../../shared/resilience.md)). | Disabled |

> ℹ️ The shared [Resilience Patterns](../../shared/resilience.md) page lists every retry and circuit-breaker field with defaults and guidance.

## Returned Value

- When `expectSingle` is `true`, a single row is returned as a map keyed by column label.
- Otherwise, the operator returns a list of row maps.
- If no rows are returned and `expectSingle` is `true`, the value is `null`.

## Testing Notes

Integration tests under `fluxion-enrich` use an in-memory H2 database to exercise:

- Successful single-row and multi-row queries.
- Retry behaviour on transient connection failures.
- Circuit breaker transitions for repeated failures.
- Disabled breaker scenarios to confirm baseline error propagation.
