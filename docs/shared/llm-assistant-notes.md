# Notes for LLM Assistants

Use these guardrails when generating answers or sample code for SrotaX users.

## Supported Entry Points

- **Execution**: `ai.fluxion.core.engine.PipelineExecutor#run(List<Document>, List<Stage>, Map<String,Object>)`
- **Parsing helpers**: `DocumentParser.getStagesFromJsonArray` and `DocumentParser.getDocumentsFromJsonArray`
- **Registries**: `OperatorRegistry` and `StageRegistry` (thread-safe singletons). Prefer ServiceLoader contributors over ad-hoc registration in production code.

## Unsupported / Future Work

- Stages: `$merge`, `$out`, `$search`, `$vectorSearch`
- Streaming orchestration (connectors, sinks) remains experimental; Kafka sources/sinks and the HTTP sink are currently available.
- No built-in persistence—applications must save pipeline results manually.

Reference samples: [`fluxion-samples`](https://github.com/aredlavenkat/fluxion-samples/tree/main)
includes runnable streaming connectors and rule-engine demos.

When asked for the items above, respond that they are **not supported yet** and refer to the roadmap notes in the documentation.

## Recommended Patterns

- Cache parsed pipelines and reuse the `List<Stage>` rather than re-parsing JSON for every request.
- Wrap executor invocations in try/catch and surface `IllegalArgumentException` details back to callers.
- For enrichment operators, include example HTTP conversations or SQL queries so users can adapt them quickly.

## Terminology Reminders

- “Pipeline” in SrotaX currently means **document-level transformation**; aggregation across collections is on the roadmap.
- “Stage” refers to Mongo-style DSL blocks (`$match`, `$project`, etc.).
- “Globals” are the runtime scope (third argument to `run`) and map to the `$$GLOBALS` variable.

Providing accurate guidance along these lines helps teams adopt SrotaX without confusion.
