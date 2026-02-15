# `$sqlQuery`

Executes a parameterised SQL statement through a registered `SqlConnector`. Ideal
for enriching documents with relational data during pipeline evaluation.

---

## 1. Prerequisites

| Requirement | Notes |
| --- | --- |
| Dependency | `ai.fluxion:fluxion-connect` plus JDBC driver (e.g., PostgreSQL, MySQL). |
| Connection registry | Register `SqlConnector` instances in `ConnectorManager`. |
| Resilience | Optional Resilience4j retry/circuit breaker configuration. |

---

## 2. Syntax

```json
{
  "$sqlQuery": {
    "connection": "ordersDb",
    "query": "SELECT * FROM orders WHERE user_id = ?",
    "params": ["$user.id"],
    "expectSingle": true,
    "retry": {
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

---

## 3. Options

| Field | Description | Default |
| --- | --- | --- |
| `connection` | Name of `SqlConnector` registered in the connector registry. | **Required** |
| `query` | SQL statement with positional `?` parameters. | **Required** |
| `params` | Array of expressions bound sequentially to `?` placeholders. | `[]` |
| `expectSingle` | Return first row or `null` when no rows. If `false`, returns a list of rows. | `false` |
| `retry` | Resilience4j retry configuration (see [Resilience Patterns](../../shared/resilience.md)); supports `retryOn` / `ignore` exception lists. | Disabled |
| `circuitBreaker` | Resilience4j circuit breaker configuration. | Disabled |

Resilience layers:
- **Connection-level**: define `retry`/`circuitBreaker` on the SQL connector; all `$sqlQuery` usages inherit them.
- **Call-level overrides**: set `retry` / `circuitBreaker` in the operator to override defaults for a single query.

---

## 4. Examples

### Fetch latest orders (multiple rows)

```json
{
  "$sqlQuery": {
    "connection": "ordersDb",
    "query": "SELECT id, total FROM orders WHERE customer_id = ? ORDER BY created DESC LIMIT 5",
    "params": ["$customerId"]
  }
}
```

### Fetch single profile record

```json
{
  "$sqlQuery": {
    "connection": "profileDb",
    "query": "SELECT email, country FROM profiles WHERE id = ?",
    "params": ["$user.id"],
    "expectSingle": true
  }
}
```

---

## 5. Returned value

- `expectSingle = true` → single row map or `null` if no rows.
- `expectSingle = false` → list of row maps. Column labels become map keys.
- Numeric/temporal types are mapped according to the JDBC driver.

---

## 6. Connectors

Register connectors through `ConnectorManager`:

```java
ConnectorManager.register("ordersDb", new SqlConnector(
        dataSource,
        SqlConnector.Settings.builder().maxConnections(10).build()
));
```

Supports connection pooling and connection-specific options.

---

## 7. Troubleshooting

| Symptom | Possible cause | Remedy |
| --- | --- | --- |
| `IllegalArgumentException: connection missing` | `connection` field omitted. | Provide connector name. |
| `SQLException` for syntax errors | Invalid SQL string. | Validate query manually or add integration tests. |
| `Empty result when expectSingle` | Query returned zero rows. | Accept `null` or change query to enforce existence. |
| `Duplicate key on sink` | Upsert logic in downstream sink misconfigured. | Check sink `mode`/`keyField`. |

---

## 8. Testing

- Run enrichment tests:
  ```bash
  mvn -pl fluxion-connect -am test -Dtest=*SqlQuery*
  ```
- Integration tests in the repo use H2 with prepared statements; replicate the
  pattern to test against your own schema.

---

## 9. References

| Path | Description |
| --- | --- |
| `fluxion-connect/src/main/java/.../SqlQueryOperator.java` | Operator implementation. |
| `fluxion-connect/src/test/java/.../SqlQueryOperatorTest.java` | Test coverage (H2 + Resilience scenarios). |
| [Resilience Patterns](../../shared/resilience.md) | Retry/circuit breaker configuration. |

Use `$sqlQuery` to pull relational data into pipelines without embedding JDBC
code directly in your services.
