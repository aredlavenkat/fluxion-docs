# Connector Manifest & SDK

This page documents the connector manifest format and how the dispatcher/SDK
load it. For building custom providers/runtimes, see the “Build Custom Connector”
pages in the sidebar.

- Thin, reusable integrations for external systems (HTTP, DBs, Kafka, S3, etc.).
- Multi-tenant aware: never leak data/secrets across tenants.
- Declarative manifests (JSON/YAML) plus optional Java handlers.
- A fixed set of execution types—no connector-specific executors.

---

## Execution types (required)

Every operation declares an `execution.type` (do not invent new types):

- `http` — generic HTTP/REST calls
- `javaBean` — call a Spring bean implementing connector logic (SDKs, DB drivers, etc.)
- `pipelineCall` — invoke another SrotaX pipeline or ruleset
- `webhook` — inbound HTTP/webhook triggers
- `polling` — periodic polling of external systems
- `streaming` — streaming sources/sinks (Kafka, EventHub, Mongo, HTTP sink, custom)
- `timer` — cron triggers, delays, scheduled tasks

## LLM-friendly recipe
1. Pick `execution.type` (http | javaBean | pipelineCall | webhook | polling | streaming | timer).
2. Write a manifest with `id`, `version`, `operations`, `operationDefs[*].kind` (action/trigger), and `execution.type`.
3. Implement handlers (if using SDK):
   - Actions: `ConnectorActionHandler` or extend `AbstractActionHandler`.
   - Triggers: `ConnectorTriggerHandler` returning `Flux<O>`.
4. Enforce secrets/multi-tenancy via `ConnectorContext.resolveSecret`, keep handlers stateless.

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

**webhook (trigger)**
```json
{
  "operations": { "ingest": "ingest" },
  "operationDefs": {
    "ingest": {
      "operationId": "ingest",
      "kind": "trigger",
      "execution": {
        "type": "webhook",
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

**streaming (trigger)**
```json
{
  "operations": { "stream": "stream" },
  "operationDefs": {
    "stream": {
      "operationId": "stream",
      "kind": "trigger",
      "execution": {
        "type": "streaming",
        "stream": "orders",
        "source": { "type": "kafka", "bootstrapServers": "localhost:9092", "topic": "orders", "groupId": "g1" },
        "sink": { "type": "http", "endpoint": "https://my.service/ingest" }
      }
    }
  }
}
```

**streaming with pipeline process + pipeline sink (trigger)**
```json
{
  "operations": { "stream": "stream" },
  "operationDefs": {
    "stream": {
      "operationId": "stream",
      "kind": "trigger",
      "execution": {
        "type": "streaming",
        "stream": "orders",
        "process": { "type": "pipeline", "pipeline": "enrich-orders", "version": "v1" },
        "sink": {
          "type": "pipeline",
          "pipeline": "ship-orders",
          "next": { "type": "http", "endpoint": "https://my.service/notify" }
        }
      }
    }
  }
}
```
Notes:
- `process` runs before the sink; output feeds the sink chain/fan-out.
- `pipeline`/`pipelineCall` sinks forward their output into `next` sinks if present.

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

For custom providers/runtimes and per-type recipes, see:
- [Build custom connector: HTTP](build-custom-connector-http.md)
- [Build custom connector: JavaBean](build-custom-connector-javabean.md)
- [Build custom connector: Pipeline call](build-custom-connector-pipeline-call.md)
- [Build custom connector: Webhook](build-custom-connector-webhook.md)
- [Build custom connector: Polling](build-custom-connector-polling.md)
- [Build custom connector: Streaming](build-custom-connector-streaming.md)
- [Build custom connector: Timer](build-custom-connector-timer.md)

```java
@Configuration
public class HeartbeatTimerConfig {

    @Bean
    CommandLineRunner startTimer(PipelineExecutor pipelineExecutor) {
        return args -> {
            List<Stage> stages = DocumentParser.getStagesFromJsonArray(
                Files.readString(Path.of("pipelines/heartbeat.json"))
            );

            ScheduledExecutorService scheduler = Executors.newSingleThreadScheduledExecutor();
            Runnable runPipeline = () -> {
                List<Document> input = DocumentParser.getDocumentsFromJsonArray("[{}]"); // pipeline adds fields
                pipelineExecutor.run(input, stages, Map.of("tenantId", "acme")); // globals hold tenant
            };

            scheduler.scheduleAtFixedRate(runPipeline, 0, 5, TimeUnit.MINUTES);
        };
    }
}
```

Swap out the scheduler for your trigger runtime so that the timer manifest drives scheduling automatically and the emitted document (`firedAt`, `message`) becomes the first input into your pipeline.

## Pipeline invoker + resilience (starter defaults)
- The Spring Boot starter ships a default `PipelineCallInvoker` that POSTs to `/api/pipelines/{name}/{version}:run` on the pipeline-service.
- Configure target URL + timeout and bind retry/circuit-breaker via Resilience4j registries:
```yaml
fluxion:
  connect:
    pipeline:
      base-url: http://fluxion-pipeline-service:8085
      timeout-ms: 10000
resilience4j:
  retry:
    instances:
      pipeline-service:
        max-attempts: 3
        wait-duration: 500ms
  circuitbreaker:
    instances:
      pipeline-service:
        failure-rate-threshold: 50
        wait-duration-in-open-state: 5s
        permitted-number-of-calls-in-half-open-state: 3
        sliding-window-size: 10
```
- This invoker is used for `execution.type=pipelineCall`, streaming `process`, and pipeline sinks. Inputs are sent as `document`; outputs flow to the sink chain or back to the caller.

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

## How connectors are resolved at runtime

- **Manifest drives dispatch**: `connectionRef` + `connectorId` + `operationId` select the connector and operation. Enrichment operators (`$httpCall`, `$sqlQuery`, `$enrich`) pass these into the Connector Dispatcher; streaming connectors use `SourceConnectorProvider` / `SinkConnectorProvider`.
- **Scopes**: the connector registry is scoped by tenant/pipeline. Set the scope (`tenant::pipeline`) before registering connectors to avoid cross-tenant leakage.
- **ServiceLoader**: source/sink providers are discovered via `META-INF/services/ai.fluxion.core.engine.connectors.SourceConnectorProvider` (or sink). JavaBean executions are resolved from your DI container by `beanName`.
- **Auth/config**: `ConnectorContext` supplies tenant/pipeline IDs, config, secret resolution, logger, metrics, and tracing; never cache per-tenant secrets statically.

---

### Dispatcher vs. providers
- Built-in in the dispatcher: `webhook`, `polling`, `timer`, `javaBean`, `pipelineCall`, `http`. With a manifest that declares these types, the dispatcher spins up the webhook server, polling interval, timer ticks, or invokes the registered bean/pipeline/HTTP call. No provider SPI is needed—just register your action/trigger handlers (for javaBean/polling) and optional pipeline invoker/HTTP headers, etc.
- Invocation paths: use `ManifestConnectorDispatcher` directly, or invoke actions from pipelines via `$enrich` (or the specialized operators like `$httpCall`/`$sqlQuery` where available).
- Provider SPI required: `streaming` sources/sinks. You must have `SourceConnectorProvider` / `SinkConnectorProvider` implementations for the `source.type` / `sink.type` (Kafka/EventHub/Mongo/custom) discovered via ServiceLoader.
- Manifest loading: ServiceLoader only discovers providers. Manifests must be loaded separately—either via the Spring Boot starter classpath scan (e.g., `classpath*:manifests/*.json`) or manually with `ConnectorManifestLoader.load(...)` before using the dispatcher/`$enrich`.

For per-type recipes and custom connector creation, see the “Build Custom Connector”
pages in the sidebar (HTTP, JavaBean, pipeline call, webhook, polling, streaming,
timer).

## Execution-type examples at a glance

Use these patterns per `execution.type` when authoring manifests.

### http (action)
```json
{
  "execution": {
    "type": "http",
    "method": "POST",
    "urlTemplate": "https://api.example.com/echo",
    "headers": { "X-Tenant": "{{context.tenantId}}" },
    "timeoutMs": 5000
  }
}
```

### javaBean (action)
```json
{
  "execution": {
    "type": "javaBean",
    "beanName": "helloConnector"   // Spring/Micronaut bean implementing ConnectorActionHandler
  }
}
```

### pipelineCall (action)
```json
{
  "execution": {
    "type": "pipelineCall",
    "targetPipeline": "downstream.orders",
    "version": "1.0.0"
  }
}
```

### httpServer (trigger/webhook)
```json
{
  "execution": {
    "type": "httpServer",
    "path": "/webhook",
    "method": "POST"
  }
}
```

### polling (trigger)
```json
{
  "execution": {
    "type": "polling",
    "intervalMillis": 30000,
    "beanName": "pollerHandler"   // implements ConnectorTriggerHandler
  }
}
```

### reactiveStream (streaming)
```json
{
  "execution": {
    "type": "reactiveStream",
    "stream": "orders",
    "group": "connectors"
  }
}
// Provide SourceConnectorProvider via META-INF/services for stream "orders"
```

### timer (trigger)
```json
{
  "execution": {
    "type": "timer",
    "cron": "PT5M"
  }
}
```

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
