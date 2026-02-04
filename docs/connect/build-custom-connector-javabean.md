# Build Custom Connector: JavaBean Action/Trigger

**When to use:** custom logic inside your service (SDKs, DB drivers, bespoke
code) without writing a new transport.

## Manifest
```json
"execution": { "type": "javaBean", "beanName": "myHandler" }
```

## SDK
```java
dispatcher.registerActionHandler("myHandler", (c, in) -> Map.of("ok", true));
dispatcher.executeAction(manifest, "myOp", ctx, Map.of());
```

## Trigger variant
Register the same bean as a trigger handler:
```java
dispatcher.registerTriggerHandler("myHandler", (c, cfg) -> Flux.just(Map.of("tick", 1)));
dispatcher.startTrigger(manifest, "myOp", ctx, Map.of());
```

## Full example (action)

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
      "execution": { "type": "javaBean", "beanName": "helloConnector" }
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
        return Map.of("greeting", "Hello " + name);
    }
}
```

Pipeline usage (inside a stage):
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

## Full example (action)

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
      "execution": { "type": "javaBean", "beanName": "helloConnector" }
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
        return Map.of("greeting", "Hello " + name);
    }
}
```

Pipeline usage (expression inside `$set`):
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
