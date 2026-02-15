# Azure Event Hubs (deprecated)

Event Hubs connectors are no longer shipped with `fluxion-connect`. Use a custom
streaming source/sink (see [Custom Sources](custom-sources.md)) or contribute a
provider that implements `SourceConnectorProvider`/`SinkConnectorProvider` if you
need Event Hubs support in your deployment.

Key pointers for a custom implementation:
- Implement the streaming SPI and register via ServiceLoader.
- Expose `discoverStreams` if you want catalog metadata in UIs/CLIs.
- Reuse the shared resilience/registry wiring from `fluxion-connect` for
  connection management and retry/circuit breaker policies.
