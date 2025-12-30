# Connector Manifest & SDK

This page documents the connector manifest format and the Java SDK used for custom
handlers. The goals:

- Thin, reusable integrations for external systems (HTTP, DBs, Kafka, S3, etc.).
- Multi-tenant aware: never leak data/secrets across tenants.
- Declarative manifests (JSON/YAML) plus optional Java handlers.
- A fixed set of execution types—no connector-specific executors.

---

## Execution types (required)

Every operation declares an `execution.type` (do not invent new types):

- `http` — generic HTTP/REST calls
- `javaBean` — call a Spring bean implementing connector logic (SDKs, DB drivers, etc.)
- `pipelineCall` — invoke another Fluxion pipeline or ruleset
- `httpServer` — inbound HTTP/webhook triggers
- `polling` — periodic polling of external systems
- `reactiveStream` — streaming sources (Kafka, Pub/Sub, CDC, etc.)
- `timer` — cron triggers, delays, scheduled tasks

## LLM-friendly recipe
1. Pick `execution.type` (http | javaBean | pipelineCall | httpServer | polling | reactiveStream | timer).
2. Write a manifest with `id`, `version`, `operations`, `operationDefs[*].kind` (action/trigger), and `execution.type`.
3. Implement handlers:
   - Actions: `ConnectorActionHandler` or extend `AbstractActionHandler`.
   - Triggers: `ConnectorTriggerHandler` returning `Flux<O>`.
   - Streaming: `SourceConnectorProvider` / `SinkConnectorProvider` + `discoverStreams` returning `ConnectorStreamDescriptor`.
4. Package:
   - Put manifest under `src/main/resources/manifests/`.
   - Add ServiceLoader files: `META-INF/services/ai.fluxion.core.engine.connectors.SourceConnectorProvider` / `...SinkConnectorProvider`.
   - Register Spring beans for `javaBean`/`httpServer`/`pipelineCall` executions.
5. Enforce secrets/multi-tenancy via `ConnectorContext.resolveSecret`, keep handlers stateless, use discovery endpoints for catalogs.

## Sample manifests by execution.type

**http (action)**
```json
{
  "operations": { "echo": "echo" },
  "operationDefs": {
    "echo": {
      "operationId": "echo",
      "kind": "action",
      "execution": {
        "type": "http",
        "method": "POST",
        "urlTemplate": "https://api.example.com/echo",
        "timeoutMs": 5000
      }
    }
  }
}
```

**javaBean (action)**
```json
{
  "operations": { "add": "add" },
  "operationDefs": {
    "add": {
      "operationId": "add",
      "kind": "action",
      "execution": {
        "type": "javaBean",
        "beanName": "addHandler"
      }
    }
  }
}
```

**pipelineCall (action)**
```json
{
  "operations": { "invoke": "invoke" },
  "operationDefs": {
    "invoke": {
      "operationId": "invoke",
      "kind": "action",
      "execution": {
        "type": "pipelineCall",
        "targetPipeline": "downstream.orders",
        "version": "1.0.0"
      }
    }
  }
}
```

**httpServer (trigger/webhook)**
```json
{
  "operations": { "ingest": "ingest" },
  "operationDefs": {
    "ingest": {
      "operationId": "ingest",
      "kind": "trigger",
      "execution": {
        "type": "httpServer",
        "path": "/webhook",
        "method": "POST"
      }
    }
  }
}
```

**polling (trigger)**
```json
{
  "operations": { "poll": "poll" },
  "operationDefs": {
    "poll": {
      "operationId": "poll",
      "kind": "trigger",
      "execution": {
        "type": "polling",
        "intervalMillis": 30000
      }
    }
  }
}
```

**reactiveStream (trigger)**
```json
{
  "operations": { "stream": "stream" },
  "operationDefs": {
    "stream": {
      "operationId": "stream",
      "kind": "trigger",
      "execution": {
        "type": "reactiveStream",
        "stream": "orders",
        "group": "connectors"
      }
    }
  }
}
```

**timer (trigger)**
```json
{
  "operations": { "tick": "tick" },
  "operationDefs": {
    "tick": {
      "operationId": "tick",
      "kind": "trigger",
      "execution": {
        "type": "timer",
        "cron": "PT1M"
      }
    }
  }
}
```

---

## Manifest shape (JSON/YAML)

```json
{
  "schemaVersion": "1.0.0",
  "id": "slack.webapi",
  "version": "1.0.0",
  "displayName": "Slack Web API",
  "category": "messaging",
  "auth": {
    "type": "bearerToken",
    "configSchema": { "type": "object" }
  },
  "configSchema": { "type": "object" },
  "operations": {
    "postMessage": { "$ref": "#/operationDefs/postMessage" }
  },
  "operationDefs": {
    "postMessage": {
      "operationId": "postMessage",
      "kind": "action",
      "inputSchema": { "type": "object" },
      "outputSchema": { "type": "object" },
      "execution": {
        "type": "http",
        "method": "POST",
        "urlTemplate": "{{config.baseUrl}}/chat.postMessage",
        "authStyle": "bearerTokenFromConfig.botTokenRef",
        "timeoutMs": 10000
      }
    }
  }
}
```

Fields:

- `id`, `version`, `displayName`, `category`
- `auth`: `type`, optional `configSchema`
- `configSchema`: connector-level JSON Schema
- `operations`: map of operation name → ref into `operationDefs`
- `operationDefs[*]`: `operationId`, `kind` (`action` or `trigger`), `inputSchema`,
  `outputSchema`, `execution` (must include `execution.type`)

---

## Java Connector SDK

Use these interfaces when custom logic is needed (typically for `javaBean`, `polling`,
`reactiveStream`):

- `ConnectorActionHandler<I, O>` — one-shot actions (send email/HTTP/db write).
- `ConnectorTriggerHandler<I, O>` — triggers/streams; returns `Flux<O>`.
- `ConnectorContext` — tenant/pipeline metadata, config, secret resolver, logger,
  metrics, tracing; multi-tenant safe.
- `AbstractActionHandler<I, O>` — convenience wrapper to record latency and log failures.

Example action handler:

```java
@Component("smtpEmailConnector")
public class SmtpEmailConnector extends AbstractActionHandler<SendEmailInput, SendEmailOutput> {
    private final JavaMailSender mailSender;
    public SmtpEmailConnector(JavaMailSender mailSender) { this.mailSender = mailSender; }

    @Override
    protected SendEmailOutput doExecute(ConnectorContext ctx, SendEmailInput input) throws Exception {
        SmtpConfig cfg = ctx.getConnectorConfig(SmtpConfig.class);
        ctx.getLogger().info("Sending email for tenant {} to {}", ctx.getTenantId(), input.to);
        // ... send ...
        return new SendEmailOutput("sent");
    }
}
```

---

## Secrets and multi-tenancy

- Store only secret references (e.g., `secret://vault/t-123/slack/botToken`) in config.
- Mark secret fields in schemas with `"x-secret": true`.
- Resolve secrets only via `ConnectorContext.resolveSecret(secretRef)`.
- Never log or emit secret values.

---

## Orchestration boundary

- Pipeline orchestration lives in the engine (`ConnectorEngine`, `PipelineEngine`).
- Handlers should perform external I/O only; do not embed orchestration logic.
- Triggers/streams are started by the trigger runtime; actions are invoked by pipeline
  connector nodes.

---

## Catalogs (streams/sinks)

- `ConnectorStreamDescriptor` exposes stream metadata (name, namespace, schema,
  sync modes, cursor/PK hints) for `reactiveStream`/`polling` connectors and sinks.
- Use discovery endpoints (`/api/connectors/discovery/sources|sinks` in pipeline-service)
  to fetch catalogs; pipeline specs remain connector-agnostic.

---

## Checklist for new connectors

- Pick an `execution.type` from the fixed list.
- Provide a manifest with `id/version/displayName/category`, `auth`, `configSchema`,
  `operations` + `operationDefs` (with `execution.type`).
- For custom logic, implement `ConnectorActionHandler`/`ConnectorTriggerHandler` or extend
  `AbstractActionHandler`.
- Respect multi-tenancy and secret handling via `ConnectorContext`.
- (Streaming) Expose `ConnectorStreamDescriptor` if applicable for catalogs.
