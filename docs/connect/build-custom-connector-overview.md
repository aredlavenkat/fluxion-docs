# Build Custom Connectors

Use these pages when you need a recipe for a specific execution type. Each page
shows a manifest snippet and the matching SDK/SPI wiring. Built-in providers
today: Kafka source/sink and HTTP sink; add others via the SPI.

- [HTTP action](build-custom-connector-http.md)
- [JavaBean action/trigger](build-custom-connector-javabean.md)
- [Pipeline call action](build-custom-connector-pipeline-call.md)
- [Webhook trigger](build-custom-connector-webhook.md)
- [Polling trigger](build-custom-connector-polling.md)
- [Streaming trigger (source → sink)](build-custom-connector-streaming.md)
- [Timer trigger](build-custom-connector-timer.md)

If you are creating a new transport, start with the streaming page—it includes
the provider/runtime skeletons and ServiceLoader registration steps.
