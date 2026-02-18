---
name: prometheus
description: >
  Design and operate Prometheus monitoring with sustainable metric models, alert quality,
  and production-safe TSDB operations. Use for metric naming and labels, recording/alert rules,
  scrape architecture, cardinality control, retention tuning, and incident diagnostics.
---

# Prometheus

## Workflow
1. Confirm SLOs and critical service signals.
2. Define metric model and label policy before instrumentation.
3. Configure scrape jobs and service discovery with security constraints.
4. Implement recording rules for expensive or standardized queries.
5. Build alert rules with clear severity and runbook links.
6. Tune retention and storage based on ingestion profile.
7. Validate alert and query behavior under failure scenarios.

## Preflight (Ask / Check First)
- Prometheus version and deployment model.
- Ingestion volume and retention targets.
- Alertmanager routing and escalation policy.
- Known cardinality hotspots.
- Multi-tenant or federation requirements.

## Metric and Label Design
- Use stable, descriptive metric names with consistent units.
- Prefer low-cardinality labels and bounded value sets.
- Avoid dynamic identifiers (request IDs, user IDs) as labels.
- Keep most metrics unlabeled or minimally labeled.
- Prefer counters/histograms with clear semantics.

## Scrape and Rule Architecture
- Keep scrape intervals aligned with SLO sensitivity.
- Separate high-churn targets from steady infrastructure jobs.
- Use recording rules for heavy dashboards and common alert predicates.
- Group related rules and set rule evaluation intervals intentionally.
- Keep rule files versioned and reviewed like code.

### Rule Group Pattern
```yaml
groups:
  - name: service-latency
    interval: 30s
    rules:
      - record: service:http_request_rate5m
        expr: sum by (service) (rate(http_requests_total[5m]))
```

## Alert Quality and Reliability
- Alert on symptoms that map to user impact.
- Add `for` windows to reduce flapping.
- Include clear labels/annotations and runbook pointers.
- Keep warning vs critical thresholds consistent.
- Test alert queries against realistic failure data.

## TSDB and Operations
- Set retention by business need and storage budget.
- Monitor scrape failures, target churn, and WAL pressure.
- Plan compaction/storage overhead with growth forecasts.
- Keep backup/restore strategy for rules and critical state.
- Validate upgrade path and config compatibility.

## Security and Governance
- Restrict scrape endpoints and admin APIs.
- Protect service discovery credentials.
- Isolate multi-tenant data where required.
- Keep audit trail for rule/config changes.

## Validation Commands
```bash
promtool check config prometheus.yml
promtool check rules rules/*.yml
curl -fsS http://localhost:9090/-/ready
```

## Common Failure Modes
- High-cardinality labels exhausting memory.
- Alert rules without `for` causing flapping storms.
- Expensive ad hoc queries running as dashboard defaults.
- Stale scrape targets from discovery misconfiguration.
- Retention settings disconnected from disk capacity.

## Definition of Done
- Metric model is documented and cardinality-safe.
- Rule and alert sets pass `promtool` checks.
- Alerting is actionable and noise-controlled.
- Retention and storage policies are capacity-validated.
- Security controls and ownership are defined.

## Operational Checklist
- Cardinality budget review is part of instrumentation PRs.
- Alert coverage maps to top service-level risks.
- Rule files have ownership and on-call runbooks linked.
- Retention settings are reviewed against monthly growth.
- Scrape failures and stale targets are continuously monitored.

## References
- `references/prometheus-2026-02-18.md`

## Reference Index
- `rg -n "metric|label|cardinality" references/prometheus-2026-02-18.md`
- `rg -n "rule group|recording|alert" references/prometheus-2026-02-18.md`
- `rg -n "retention|TSDB|operations" references/prometheus-2026-02-18.md`
- `rg -n "promtool|validation" references/prometheus-2026-02-18.md`
