# Resilience Patterns

Fluxion reuses the same Resilience4j building blocks across enrichment operators and streaming connectors. This page centralises the retry and circuit-breaker options so you can reference one source of truth.

## Retry Options

| Field | Meaning | Default |
| --- | --- | --- |
| `name` | Identifier for the retry instance. Pipelines sharing the same name share state. | Derived from operator or connector name |
| `maxAttempts` | Total number of attempts (initial call + retries). | `1` |
| `waitDurationMs` | Fixed backoff between attempts in milliseconds. | `1000` |
| `multiplier` | When > 1.0, enables exponential backoff (`waitDurationMs` as the base). | `1.0` |
| `retryOn` | List of fully-qualified exception class names to include. | `[]` |
| `ignore` | Exceptions that should fail fast without retrying. | `[]` |

## Circuit Breaker Options

| Field | Meaning | Default |
| --- | --- | --- |
| `enabled` | Enables or disables the breaker. | `true` |
| `name` | Identifier for the breaker instance. Shared across pipelines. | Derived from operator or connector name |
| `failureRateThreshold` | Percentage of failures that trips the breaker to OPEN. | `50` |
| `slowCallRateThreshold` | Percentage of slow calls that counts towards failure. | `100` |
| `slowCallDurationThresholdMs` | Milliseconds defining a "slow" call. | `60000` |
| `waitDurationInOpenStateMs` | How long the breaker stays OPEN before half-open probes. | `10000` |
| `permittedCallsInHalfOpenState` | Trial calls allowed when HALF_OPEN. | `2` |
| `minimumNumberOfCalls` | Minimum sample size before metrics are considered. | `10` |
| `slidingWindowSize` | Size of the metrics window. | `100` |
| `slidingWindowType` | `0` for count-based, `1` for time-based. | `1` |
| `record` / `ignore` | Additional exception classes to record or ignore. | `[]` |

## Usage Guidelines

- Keep retry counts small for idempotent operations; for non-idempotent writes, prefer compensating logic.
- Pair circuit breakers with targeted alerts so you'll know when a downstream system is degrading.
- Use explicit `name` values when you want multiple pipelines to share the same resilience policy.
- Reset helpers (`SqlQueryOperator.resetResilience()`, `HttpCallOperator.resetResilience()`) are available in tests to ensure clean state between runs.

Refer to:

- [`$sqlQuery`](../enrich/operators/sqlQuery.md)
- [`$httpCall`](../enrich/operators/httpCall.md)

for operator-specific nuances.
