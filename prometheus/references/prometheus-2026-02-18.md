# Prometheus Reference (2026-02-18)

## Context7 Sources
- Library: `/prometheus/docs`
- Prometheus docs on instrumentation, rules, and alerting.

## Practical Guidance
- Keep metric names stable and unit-aware.
- Enforce low-cardinality label policy.
- Use recording rules for expensive queries and normalized aggregates.
- Add `for` windows and runbook annotations to alert rules.
- Validate config and rules with `promtool` before deployment.

## Source URLs
- <https://prometheus.io/docs/>
- <https://github.com/prometheus/docs>
