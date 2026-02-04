# Fluxion Connect Spring Boot Starter

Auto-configures Fluxion Connect in Spring Boot apps: registers streaming
providers, loads manifests, and exposes connector discovery without manual
bootstrap code.

## Modules to include
- Core SPI: `ai.fluxion:fluxion-connect`
- Transports (pick what you need):
  - `ai.fluxion:fluxion-connect-kafka`
  - `ai.fluxion:fluxion-connect-eventhub`
  - `ai.fluxion:fluxion-connect-mongo`
  - (HTTP sink lives in `fluxion-connect`)
- Starter: `ai.fluxion:fluxion-connect-spring-boot-starter`

Example:
```xml
<dependency>
  <groupId>ai.fluxion</groupId>
  <artifactId>fluxion-connect-spring-boot-starter</artifactId>
  <version>${fluxion.version}</version>
</dependency>
<dependency>
  <groupId>ai.fluxion</groupId>
  <artifactId>fluxion-connect-kafka</artifactId>
  <version>${fluxion.version}</version>
</dependency>
```

## What the starter does
- Enables ServiceLoader discovery of `SourceConnectorProvider` /
  `SinkConnectorProvider` from any included transport module.
- Loads connector manifests on the classpath (default location:
  `classpath*:manifests/*.json`) into the manifest catalog.
- Registers connector providers with the registry for streaming execution and
  discovery endpoints.

## Configuration
Minimal config (defaults used when omitted):
```yaml
fluxion:
  connect:
    manifests:
      locations:
        - classpath*:manifests/*.json
```

## Usage
- **Manifest-driven**: place manifests under the configured locations; the
  starter loads them and providers are auto-registered.
- **SDK/SPI**: include the transport modules; providers are auto-registered via
  ServiceLoader, no manual registry wiring needed.

## Notes
- To add a new transport, publish its module with ServiceLoader files; adding the
  dependency is enough for auto-registration.
- If you need custom locations or to disable loading, override the properties in
  your `application.yml`.
