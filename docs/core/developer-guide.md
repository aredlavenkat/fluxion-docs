# SrotaX Core Developer Guide

This guide explains how to understand, build, test, and extend the SrotaX Core Engine itself. Use
`fluxion-core-engine` when you want to add new stages, expression operators, streaming behaviors, or
core utilities and ship them independently of the surrounding connectors/enrichers.

## Repository focus

- **Aggregation** – pipelines are defined with MongoDB-style stages (`$match`, `$setWindowFields`,
  `$group`, `$subPipeline`, …) that are executed via `PipelineExecutor`, `StageRegistry`, and
  `SubPipelineStageHandler`.
- **Expressions** – the `ExpressionEvaluator` resolves operands, registers operators through
  `OperatorRegistry`, and allows contributors to register custom expression logic.
- **Streaming runtime** – `StreamingPipelineExecutor` plus `StreamingErrorPolicy` drive continuous
  execution, observability, metrics, and connector workflows (e.g., `HttpStreamingConnector`).
- **Extensions & SPIs** – stage/operator contributors are discovered via Java `ServiceLoader`
  implementations (`StageHandlerContributor`, `OperatorContributor`, etc.).

## Building & testing

```bash
mvn -pl . test
```

The command compiles the core module, runs the entire unit/integration suite, and covers both batch
and streaming paths (including the `$subPipeline` skip/bubble behavior described in the unit tests).
Run the suite after any API, operator, or stage change to keep regressions out of CI.

## Documentation ecosystem

- **Core overview**: [http://docs.srotax.com/core/](http://docs.srotax.com/core/)
- **This developer guide**: [http://docs.srotax.com/core/developer-guide/](http://docs.srotax.com/core/developer-guide/)
- **Stage reference**: [http://docs.srotax.com/stages/](http://docs.srotax.com/stages/)
- **Operator reference**: [http://docs.srotax.com/operators/](http://docs.srotax.com/operators/)
- **Streaming reference**: [http://docs.srotax.com/streaming/](http://docs.srotax.com/streaming/)

The same content is published at `http://docs.srotax.com/` for easy sharing.


Update the MkDocs content whenever you add new stages/operators or change their behavior. Contributors
should run `pip install mkdocs mkdocs-material` inside `fluxion-docs` and execute `mkdocs serve` to
preview their documentation changes locally.

## Contribution checklist

1. Keep public packages under `ai.fluxion.core` backward compatible when possible. Use new
   `StageHandlerContributor`/`OperatorContributor` implementations for hot-pluggable functionality.
2. Add or update unit/streaming tests in `fluxion-core/src/test/java/...` to cover new behaviors.
3. Pair code changes with documentation updates inside `fluxion-docs` (stage/operator pages, this
   guide, or the high-level overview) so the new behavior is discoverable by future contributors.
4. Run `mvn -pl . test` before pushing and mention the failing/perfect pipeline JSON + logs if you
   need help reproducing an issue.

## Helpful references

- Maven module: `/fluxion-core-engine/pom.xml`
- Source entry points: `PipelineExecutor`, `SubPipelineStageHandler`, `StreamingPipelineExecutor`
- Registration SPI: `StageRegistry`, `OperatorRegistry`, `StageHandlerContributor`, `OperatorContributor`
- Streaming operator sources: `fluxion-core-engine/src/main/java/ai/fluxion/core/engine/streaming`
- Observability metrics: `StageMetrics`, `MetricsReporter`, `StageMetricsOtelBridge`

Keep an eye on this guide and the docs repo when you touch any of the above files so future
contributors understand how SrotaX Core works end-to-end.
