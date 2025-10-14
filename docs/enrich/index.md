# Enrichment Module Overview

Fluxion Enrich extends the aggregation engine with operators that call out to external services or data stores during pipeline evaluation.

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
