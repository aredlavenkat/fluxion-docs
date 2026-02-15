# MongoDB (deprecated)

MongoDB streaming connectors are not currently bundled with `fluxion-connect`.
If you need change-stream ingestion or MongoDB sinks, implement a custom
connector using the streaming SPI (see [Custom Sources](custom-sources.md)) and
register it via ServiceLoader.

Tips:
- Build on `AbstractAsyncStreamingSource` for change-stream polling and queueing.
- Surface `discoverStreams` catalogs if you want UI/CLI discovery.
- Use the shared connection/resilience registries from `fluxion-connect` when
  wiring authentication or retry/breaker policies.
