# Build Custom Connector: Polling Trigger

**When to use:** periodic pulls from systems that don’t push events.

## Manifest
```json
"execution": { "type": "polling", "intervalMillis": 30000, "handlerBean": "poller" }
```

## SDK
```java
dispatcher.registerTriggerHandler("poller",
    (c, cfg) -> Flux.interval(Duration.ofSeconds(30)).map(i -> Map.of("tick", i)));
dispatcher.startTrigger(manifest, "poll", ctx, Map.of());
```

If `handlerBean` is omitted, the dispatcher emits ticks by interval.

## Full example (trigger)

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
    public OrdersPoller(OrdersClient client) { this.client = client; }

    @Override
    public Flux<Map<String, Object>> trigger(ConnectorContext ctx, Map<String, Object> input) {
        return Flux.defer(() -> {
            String since = ctx.getState("lastSeenId", "0");
            List<Order> newOrders = client.fetchSince(since);
            String maxId = newOrders.stream().map(Order::id).max(String::compareTo).orElse(since);
            ctx.setState("lastSeenId", maxId);
            return Flux.fromIterable(newOrders)
                       .map(order -> Map.of(
                               "orderId", order.id(),
                               "tenantId", ctx.getTenantId(),
                               "total", order.total()));
        });
    }
}
```
