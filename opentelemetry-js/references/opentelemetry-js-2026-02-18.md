# OpenTelemetry JS Reference (2026-02-18)

## Context7 Sources
- Library: `/open-telemetry/opentelemetry-js`
- NodeSDK README and related instrumentation examples.

## Practical Guidance
- Bootstrap `NodeSDK` before app framework imports.
- Configure trace/metric/log exporters (typically OTLP) through collector endpoints.
- Set service resource attributes (`service.name`, version, environment).
- Verify propagation (`traceparent`, baggage) across inbound/outbound calls.
- Use graceful shutdown (`sdk.shutdown()`) on SIGTERM/SIGINT.

## Recommended Controls
- Exclude health endpoints from noisy instrumentation.
- Bound batch/export queue settings.
- Redact sensitive attributes before export.

## Source URLs
- <https://github.com/open-telemetry/opentelemetry-js>
- <https://opentelemetry.io/docs/languages/js/>
