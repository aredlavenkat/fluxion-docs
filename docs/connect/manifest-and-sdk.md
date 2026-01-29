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
- `pipelineCall` — invoke another SrotaX pipeline or ruleset
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

### End-to-end: timer-triggered pipeline (5-minute heartbeat)

1) Manifest (`src/main/resources/manifests/ops.timer.json`)

```json
{
  "schemaVersion": "1.0.0",
  "id": "ops.timer",
  "version": "1.0.0",
  "operations": { "tick": { "$ref": "#/operationDefs/tick" } },
  "operationDefs": {
    "tick": {
      "operationId": "tick",
      "kind": "trigger",
      "execution": { "type": "timer", "cron": "PT5M" }
    }
  }
}
```

2) Pipeline definition (`pipelines/heartbeat.json`)

```json
[
  {
    "$set": {
      "firedAt": "$$NOW",
      "message": "heartbeat"
    }
  }
]
```

3) Wire-up (Spring Boot sketch)

Load the manifest and pipeline, then schedule the pipeline every five minutes. In production you typically let the trigger runtime read the manifest and manage scheduling; the snippet below shows the wiring inline so you can see the pieces.

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

## Example: custom action connector (JavaBean) + manifest

Manifest (`src/main/resources/manifests/hello.json`):

```json
{
  "schemaVersion": "1.0.0",
  "id": "demo.hello",
  "version": "1.0.0",
  "operations": { "sayHello": { "$ref": "#/operationDefs/sayHello" } },
  "operationDefs": {
    "sayHello": {
      "operationId": "sayHello",
      "kind": "action",
      "inputSchema": { "type": "object", "properties": { "name": { "type": "string" } } },
      "outputSchema": { "type": "object" },
      "execution": {
        "type": "javaBean",
        "beanName": "helloConnector"
      }
    }
  }
}
```

Handler (`HelloConnector.java`):

```java
@Component("helloConnector")
public class HelloConnector extends AbstractActionHandler<Map<String,Object>, Map<String,Object>> {
    @Override
    protected Map<String,Object> doExecute(ConnectorContext ctx, Map<String,Object> input) {
        String name = String.valueOf(input.getOrDefault("name", "world"));
        ctx.getLogger().info("Hello {} from tenant {}", name, ctx.getTenantId());
        return Map.of("greeting", "Hello " + name);
    }
}
```

Use from a pipeline (expression-style inside `$set`):

```json
{
  "$set": {
    "greeting": {
      "$enrich": {
        "connectionRef": "demo.hello",
        "connectorId": "demo.hello",
        "operationId": "sayHello",
        "input": { "name": "$user.name" }
      }
    }
  }
}
```

This reuses the shared connector dispatcher: the enrichment operator invokes `demo.hello`/`sayHello` via the JavaBean execution you registered.

---

## Example: polling trigger (Java SDK)

Manifest (`src/main/resources/manifests/orders.poller.json`):

```json
{
  "schemaVersion": "1.0.0",
  "id": "demo.orders.poller",
  "version": "1.0.0",
  "operations": { "pollNew": { "$ref": "#/operationDefs/pollNew" } },
  "operationDefs": {
    "pollNew": {
      "operationId": "pollNew",
      "kind": "trigger",
      "inputSchema": { "type": "object" },
      "outputSchema": { "type": "object" },
      "execution": {
        "type": "polling",
        "intervalMillis": 60000,
        "beanName": "ordersPoller"
      }
    }
  }
}
```

Handler (`OrdersPoller.java`):

```java
@Component("ordersPoller")
public class OrdersPoller implements ConnectorTriggerHandler<Map<String, Object>, Map<String, Object>> {
    private final OrdersClient client;

    public OrdersPoller(OrdersClient client) {
        this.client = client;
    }

    @Override
    public Flux<Map<String, Object>> trigger(ConnectorContext ctx, Map<String, Object> input) {
        return Flux.defer(() -> {
            String since = ctx.getState("lastSeenId", "0");
            List<Order> newOrders = client.fetchSince(since);
            String maxId = newOrders.stream().map(Order::id).max(String::compareTo).orElse(since);
            ctx.setState("lastSeenId", maxId); // checkpoint
            return Flux.fromIterable(newOrders)
                       .map(order -> Map.of(
                               "orderId", order.id(),
                               "tenantId", ctx.getTenantId(),
                               "total", order.total()));
        });
    }
}
```

Use this as a trigger when deploying a pipeline: each emitted map becomes the first document into the pipeline every minute.

---

## Example: reactiveStream source provider

Manifest (`src/main/resources/manifests/orders.stream.json`):

```json
{
  "schemaVersion": "1.0.0",
  "id": "demo.orders.stream",
  "version": "1.0.0",
  "operations": { "orders": { "$ref": "#/operationDefs/orders" } },
  "operationDefs": {
    "orders": {
      "operationId": "orders",
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

Source provider (`OrdersStreamProvider.java`):

```java
public final class OrdersStreamProvider implements SourceConnectorProvider {
    @Override
    public ConnectorStreamDescriptor descriptor() {
        return ConnectorStreamDescriptor.builder()
                .stream("orders")
                .namespace("demo")
                .schema(Map.of("type", "object"))
                .build();
    }

    @Override
    public Publisher<ConnectorStreamDescriptor> discoverStreams(ConnectorContext ctx) {
        return Flux.just(descriptor());
    }

    @Override
    public Publisher<Document> create(SourceConnectorProvider.Options options, ConnectorContext ctx) {
        ReactiveOrdersClient client = options.getConfig(ReactiveOrdersClient.class);
        return client.subscribe("orders-topic")
                     .map(payload -> Document.from(Map.of(
                             "orderId", payload.id(),
                             "total", payload.total(),
                             "tenantId", ctx.getTenantId())));
    }
}
```

ServiceLoader registration (`src/main/resources/META-INF/services/ai.fluxion.core.engine.connectors.SourceConnectorProvider`):

```
com.acme.connectors.OrdersStreamProvider
```

Deploy the manifest and jar; bind the `reactiveStream` operation to a streaming pipeline so events from the client feed the first stage continuously.

---

## Execution-type examples at a glance

Use these patterns per `execution.type` when authoring manifests. Pair them with the SDK interfaces shown above and register providers via ServiceLoader when applicable.

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
