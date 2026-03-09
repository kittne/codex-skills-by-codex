---
name: opentelemetry-js
description: >
  Instrument and operate JavaScript/Node.js services with OpenTelemetry for traces, metrics,
  and logs. Use for SDK bootstrap, auto-instrumentation, resource attributes, propagation,
  exporter configuration, and production observability reliability.
---

# OpenTelemetry JS

## Workflow
1. Confirm observability goals and backend contract (OTLP collector/vendor).
2. Bootstrap NodeSDK before app code execution.
3. Define stable resource attributes and service identity.
4. Add auto/manual instrumentation where signal value is highest.
5. Configure exporters, batching, and shutdown handling.
6. Validate propagation across service boundaries.
7. Tune sampling and cardinality for production cost control.

## Preflight (Ask / Check First)
- Node.js and OpenTelemetry package versions.
- If using JS SDK 2.x, confirm runtime baseline (`^18.19.0 || >=20.6.0`) and TypeScript `>=5.0.4`.
- Collector endpoint/protocol and auth requirements.
- Required signals (traces/metrics/logs) and SLO use cases.
- Existing metric/trace naming conventions.
- Startup model (single process, workers, serverless).

## SDK Bootstrap and Resource Identity
- Initialize SDK before importing instrumented libraries.
- Set `service.name`, version, and environment attributes consistently.
- Merge detected resources with explicitly set service metadata.
- Keep one canonical service identity per deployable unit.
- Use deterministic shutdown hooks for flush behavior.

## Instrumentation Strategy
- Start with HTTP/framework/database instrumentations.
- Add manual spans only for domain-critical boundaries.
- Avoid redundant instrumentation that duplicates spans.
- Exclude noisy endpoints (health checks) from high-cardinality signals.
- Keep span names stable and semantic-convention aligned.
- Migrate deprecated `SEMATTRS_*`/`SEMRESATTRS_*` and `SemanticAttributes.*` forms to `ATTR_*` constants.

## Propagation and Context
- Use W3C trace context by default.
- Validate inbound/outbound propagation through gateways and queues.
- Guard against context loss in async boundaries.
- Keep baggage usage minimal and policy-driven.

## Export and Runtime Control
- Prefer OTLP to a collector for transport abstraction.
- Prefer `NodeSDK` lifecycle management over low-level tracer provider wiring in new code.
- In JS SDK 2.x+, keep exporter/propagator env-driven setup on `NodeSDK` (not low-level provider classes).
- Use batch processors for traces/logs in high-throughput services.
- Set metric export intervals deliberately.
- Bound queue sizes and retry behavior to protect memory.
- Fail gracefully when telemetry backend is unavailable.

### NodeSDK Pattern
```js
const { NodeSDK } = require('@opentelemetry/sdk-node')
const sdk = new NodeSDK({ serviceName: 'api-service' })
sdk.start()
process.on('SIGTERM', async () => { await sdk.shutdown(); process.exit(0) })
```

## Security and Data Hygiene
- Avoid exporting sensitive payload fields.
- Redact tokens, secrets, and personal data in span attributes.
- Keep telemetry transport encrypted.
- Restrict collector endpoints by network policy.

## Validation Commands
```bash
node -e "require('./telemetry'); console.log('otel bootstrap ok')"
curl -fsS http://localhost:4318/v1/traces || true
```

## Common Failure Modes
- SDK initialized after framework imports.
- Missing `service.name` causing unusable telemetry grouping.
- SDK 2.x migrations still using removed `BasicTracerProvider#addSpanProcessor` patterns.
- Context propagation broken across async or proxy boundaries.
- Cardinality explosion from dynamic attribute values.
- Application shutdown before telemetry flush completes.

## Definition of Done
- SDK bootstrap order is correct and tested.
- Resource identity is consistent across all signals.
- Propagation is verified in end-to-end flows.
- Exporters are stable under normal and degraded backend conditions.
- Sensitive data policies are enforced in telemetry.

## Rollout Checklist
- Telemetry startup/shutdown behavior is tested in staging.
- Signal volume and backend cost are baselined.
- Trace and metric naming aligns with team conventions.
- Alerting exists for exporter or collector failures.
- Sensitive attribute redaction is validated by tests.

## References
- `references/opentelemetry-js-2026-02-18.md`

## Reference Index
- `rg -n "NodeSDK|exporter|instrumentation" references/opentelemetry-js-2026-02-18.md`
- `rg -n "resource|service.name|attributes" references/opentelemetry-js-2026-02-18.md`
- `rg -n "propagation|context|W3C|B3" references/opentelemetry-js-2026-02-18.md`
- `rg -n "shutdown|batch|operations" references/opentelemetry-js-2026-02-18.md`
