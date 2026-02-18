---
name: grafana
description: >
  Design and operate Grafana for production observability with actionable dashboards,
  reliable alerting, and strong governance. Use for panel/query design, alert routing,
  provisioning as code, permissions/RBAC, and operational troubleshooting.
---

# Grafana

## Workflow
1. Confirm user personas and decision paths for dashboards.
2. Build dashboard hierarchy by domain, ownership, and criticality.
3. Design panels around questions, not raw metric dumping.
4. Configure alert rules and notification policies with runbook linkage.
5. Provision dashboards/datasources/alerts as code.
6. Enforce RBAC and folder permissions.
7. Validate usability during incident drills.

## Preflight (Ask / Check First)
- Grafana version and deployment topology.
- Datasources and query cost constraints.
- Team/folder ownership model.
- Alerting backend and escalation process.
- Current dashboard sprawl and stale panel count.

## Dashboard Design Principles
- One dashboard per operational question set.
- Keep top row focused on service health indicators.
- Use consistent units, legends, and time ranges.
- Add drill-down links for deep diagnostics.
- Avoid panel duplication without clear ownership.

## Alerting Strategy
- Define severity tiers and contact targets clearly.
- Keep alert expressions aligned with SLO semantics.
- Include concise annotations with next steps.
- Route alerts by team ownership and environment.
- Test mute windows and maintenance policies.

## Provisioning and Change Control
- Manage dashboards and alerts as versioned files.
- Review dashboard JSON changes in PRs.
- Use reload/provisioning APIs for controlled rollouts.
- Keep migration notes for schema/plugin upgrades.
- Track dashboard lifecycle (active, deprecated, archived).

## Access Control and Governance
- Use folder-level permissions as default boundary.
- Apply least-privilege roles for editors/admins.
- Audit high-impact permission changes.
- Separate shared global dashboards from team-owned spaces.

### Permission API Checks
```bash
curl -H "Authorization: Bearer $TOKEN" "$GRAFANA/api/dashboards/uid/$UID/permissions"
```

## Operations and Reliability
- Monitor datasource health and query latency.
- Detect slow dashboards and expensive queries.
- Keep plugin inventory minimal and reviewed.
- Backup provisioning and dashboard state for recovery.
- Rehearse Grafana upgrade and rollback flow.

## Validation Commands
```bash
curl -fsS "$GRAFANA/api/health"
curl -fsS -H "Authorization: Bearer $TOKEN" "$GRAFANA/api/search"
```

## Common Failure Modes
- Dashboards optimized for creators, not responders.
- Alert rules lacking ownership/runbooks.
- Permission drift exposing sensitive dashboards.
- Provisioning changes applied manually and lost later.
- Datasource query cost spikes from unbounded panels.

## Definition of Done
- Dashboard set is role-focused and actionable.
- Alerting is owned, routed, and tested.
- Provisioning and RBAC policies are codified.
- Operational health and query performance are monitored.
- Upgrade and recovery procedures are validated.

## Operational Checklist
- Dashboard owners are documented for each folder.
- Stale dashboards are reviewed and retired on cadence.
- Alert annotations include runbook and escalation target.
- Provisioning reload workflow is scripted and tested.
- Permission audits are performed after org/team changes.
- Datasource credentials are rotated and validated.
- Critical dashboards are exercised during incident drills.

## References
- `references/grafana-2026-02-18.md`

## Reference Index
- `rg -n "dashboard|panel|design" references/grafana-2026-02-18.md`
- `rg -n "alert|notification|routing" references/grafana-2026-02-18.md`
- `rg -n "provisioning|reload|as code" references/grafana-2026-02-18.md`
- `rg -n "permissions|RBAC|governance" references/grafana-2026-02-18.md`
