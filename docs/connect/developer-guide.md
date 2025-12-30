# Connector Developer Guide

How to build, package, and register connectors with Fluxion using manifests,
the Java SDK, and the streaming SPI.

## Quick checklist (copy/paste for LLMs)
- Pick an `execution.type` from: http | javaBean | pipelineCall | httpServer | polling | reactiveStream | timer.
- Write a manifest with `id`, `version`, `operations`, `operationDefs[*].kind` (action/trigger), and `execution.type`.
- If `javaBean`/`httpServer`/`pipelineCall`: implement a Spring bean `ConnectorActionHandler` or `ConnectorTriggerHandler`; use `ConnectorContext.resolveSecret`.
- If streaming (`reactiveStream`/`polling`): implement `SourceConnectorProvider`/`SinkConnectorProvider`, plus `discoverStreams` returning `ConnectorStreamDescriptor`.
- Package: place manifest under `src/main/resources/manifests/`, add `META-INF/services/...SourceConnectorProvider` / `...SinkConnectorProvider` for SPI; include your handler/provider classes.
- Keep pipelines connector-agnostic; use discovery endpoints to surface catalogs.
- Test with `ManifestConnectorDispatcher` (for actions/triggers) and Testcontainers for streaming.

## 1) Pick an execution type
- `http` — outbound HTTP/REST (declarative).
- `javaBean` — call a Spring bean (SDKs/DB drivers/custom logic).
- `pipelineCall` — invoke another Fluxion pipeline/ruleset.
- `httpServer` — inbound HTTP/webhook trigger.
-\u00a0`polling` — periodic trigger.
- `reactiveStream` — continuous stream (Kafka/EventHub/Mongo/etc).
- `timer` — cron/delay trigger.

## 2) Author the manifest
Define operations and execution in JSON/YAML:
```json
{
  "schemaVersion": "1.0.0",
  "id": "acme.crm",
  "version": "1.0.0",
  "configSchema": { "type": "object" },
  "operations": { "send": "send" },
  "operationDefs": {
    "send": {
      "operationId": "send",
      "kind": "action",
      "inputSchema": { "type": "object" },
      "outputSchema": { "type": "object" },
      "execution": { "type": "javaBean", "beanName": "crmSendHandler" }
    }
  }
}
```
Always include `execution.type`.

## 3) Implement handlers (SDK)
- **Actions**: implement `ConnectorActionHandler<I,O>` or extend
  `AbstractActionHandler<I,O>`.
- **Triggers/streams**: implement `ConnectorTriggerHandler<I,O>` returning
  `Flux<O>`.
- Use `ConnectorContext` for tenant/pipeline info, config, metrics, tracing, and
  `resolveSecret` for secret refs. Register beans with Spring
  (e.g., `@Component("crmSendHandler")`).

## 4) Streaming connectors (SPI)
- Implement `SourceConnectorProvider` / `SinkConnectorProvider` when adding new
  streaming sources/sinks.
- Provide `descriptor()`, `options()`, `discoverStreams(...)` returning
  `ConnectorStreamDescriptor` (name/namespace/schema/sync modes/cursor hints),
  and `create(...)` to build the runtime connector.
- Register providers via ServiceLoader:
  - `META-INF/services/ai.fluxion.core.engine.connectors.SourceConnectorProvider`
  - `META-INF/services/ai.fluxion.core.engine.connectors.SinkConnectorProvider`

## 5) Package & register
- Bundle manifests in your connector jar.
- Include handler beans for `javaBean`/`httpServer`/`pipelineCall` operations.
- Include ServiceLoader entries for SPI-based streaming connectors.
- Put the jar on the classpath; `ConnectorRegistry/Factory` loads manifests and
  providers, and dispatches by `execution.type`.

**Typical layout**
```
src/main/resources/manifests/my-connector.json   # manifest(s)
src/main/resources/META-INF/services/ai.fluxion.core.engine.connectors.SourceConnectorProvider
src/main/resources/META-INF/services/ai.fluxion.core.engine.connectors.SinkConnectorProvider
src/main/java/.../MyActionHandler.java           # for javaBean/httpServer/pipelineCall
src/main/java/.../MySourceProvider.java          # for reactiveStream/polling SPI
```
ServiceLoader file content example:
```
com.acme.connectors.MySourceProvider
```

## 6) Discovery & catalogs
- Surface stream catalogs via `discoverSourceStreams` /
  `destinationStreams` so UIs/CLIs can show schemas, sync modes, and cursor
  hints.
- Pipelines remain connector-agnostic; bind connectors/config at deploy time.

## 7) Testing
- Unit test handlers directly.
- Use the sample `ManifestConnectorDispatcher` (in `fluxion-connect` tests) to
  run manifest-backed operations locally.
- For streaming connectors, mirror the Kafka/EventHub/Mongo provider tests or
  run the e2e suite with Testcontainers.

## 8) Multi-tenancy & secrets
- Never hard-code secrets; resolve via `ConnectorContext.resolveSecret`.
- Keep handlers stateless; avoid sharing mutable state across tenants.
- Enforce auth for inbound `httpServer`/webhook triggers (signed tokens/headers)
  and apply per-tenant rate/backpressure limits.
