---
name: vault-best-practices
description: >
  Best practices for deploying, configuring, and operating HashiCorp Vault. Use when setting up or
  reviewing Vault environments (dev/prod), storage backends, TLS, unsealing, auth/policies, audit
  logging, secrets usage, HA, and operational hardening.
---

# Vault Best Practices

## Workflow
1. Classify the environment (dev/test/prod) and your threat model.
2. Choose deployment style (systemd, Docker, Kubernetes) and isolate Vault (single-tenancy when possible).
3. Harden the host/runtime and run Vault with least privilege (non-root, mlock, no swap, no core dumps).
4. Choose the storage backend and HA topology (prefer integrated storage/Raft for most new deployments).
5. Configure TLS end-to-end and restrict network access (ingress and egress).
6. Initialize and unseal safely (Shamir key shares or auto-unseal via KMS/HSM).
7. Enable audit logging early and ship logs to centralized, protected storage.
8. Configure auth methods and least-privilege policies (separate human vs machine access).
9. Enable only the required secrets engines and prefer dynamic secrets + short TTLs.
10. Validate operations: monitoring, backups/snapshots, restore drills, upgrades, and break-glass procedures.

## Quick Intake (Ask/Confirm)
- Vault version and edition (OSS/Enterprise) and where it runs (VM/bare metal, container, Kubernetes).
- Environment (dev/stage/prod) and availability requirements (RPO/RTO, maintenance windows).
- Storage backend choice (Raft/Consul/etc.) and desired HA posture (single node vs cluster).
- Unseal model: manual (Shamir) vs auto-unseal (KMS/HSM) and who holds recovery keys.
- Network model: public vs private, load balancer/proxy in front, allowed client IP ranges.
- TLS requirements: CA source, cert rotation process, and whether mTLS is required.
- Auth model: humans (OIDC/LDAP/GitHub) vs machines (AppRole/Kubernetes) and expected token TTLs.
- Audit/observability requirements: where audit logs go, retention, and who can access them.

## Core Guidance (Defaults)
- Do not run Vault as root; run as a dedicated, least-privilege service account.
- Do not expose Vault to the public internet without strong network controls and TLS.
- Do not use the root token day-to-day; use it only for bootstrap, then revoke it.
- Enable audit logging early; treat audit logs as sensitive and centralize them.
- Prefer auto-unseal (cloud KMS/HSM) for production; keep recovery keys for break-glass.

## Storage and HA
- Prefer integrated storage (Raft) unless you have a strong reason to use an external backend.
- Run enough nodes for quorum (3+ for production; 5 across multiple zones for higher resiliency).
- Protect the storage backend from tampering (Vault encrypts at rest, but deletion/corruption still hurts).
- Snapshot/backup regularly and test restores:
  - Raft: use `vault operator raft snapshot save ...` on a schedule.

## Auth, Policies, and Auditing
- Separate human and machine access:
  - Humans: OIDC/LDAP/GitHub/etc, MFA where possible, narrower admin policies (not root).
  - Machines: AppRole/Kubernetes auth, shortest practical TTLs, renewals handled by Vault Agent when possible.
- Use least-privilege policies; avoid broad wildcard permissions and avoid granting write access to `sys/*`.
- Prefer modular policies (small, reusable) and clear naming (verb + scope).
- Enable audit devices early and ship logs to centralized storage; restrict log access tightly.
- Use short TTLs for tokens and dynamic secrets; cap renewals with `max_ttl` where appropriate.

## Operations
- Monitoring minimums:
  - Seal status, leader changes, request latency/error rates, and auth failures/lockouts.
  - If HA: alert on cluster instability and raft peer health.
- Break-glass readiness:
  - Document unseal/recovery steps and practice them (tabletop + real restore drills).
  - Keep recovery keys separated; never store all key material together with Vault data.

## Hardening Checklist (Host/Runtime)
- Disable swap and core dumps; keep clocks synchronized (NTP).
- Use memory locking (`mlock`) when possible:
  - systemd: `LimitMEMLOCK=infinity` and `CAP_IPC_LOCK`.
  - Docker: add `IPC_LOCK` capability (or set `disable_mlock = true` only when forced).
- Restrict inbound traffic to Vault API (8200) to only trusted networks; restrict outbound to only required endpoints.
- Avoid sharing Vault hosts with unrelated workloads when feasible (reduce attack surface).

## TLS and Network (Minimums)
- Use TLS 1.2+ (prefer TLS 1.3); never run plaintext in shared environments.
- Ensure end-to-end TLS if using a load balancer (TLS passthrough or re-encrypt to Vault).
- Avoid `tls_skip_verify` outside of isolated local dev; use a trusted CA chain instead.
- Consider HSTS for the UI and mTLS for additional client gating when appropriate.

## Initialization and Unsealing (Safety)
- If using Shamir:
  - Use multiple key shares with a quorum threshold (e.g., 5 shares, 3 required).
  - Distribute shares to separate trusted custodians; consider PGP-encrypted shares.
- Root token handling:
  - Use only for bootstrap (enable auth methods, create first admin policy/roles), then revoke.
  - If needed later, generate a new root token via `vault operator generate-root` (requires quorum).
- Prefer auto-unseal for production:
  - Configure a `seal` stanza (cloud KMS/HSM) and store recovery keys for break-glass.

## Validation Commands (Common)
```bash
# Status / health
vault status
vault operator raft list-peers

# Audit
vault audit list

# Auth / policies
vault auth list
vault policy list

# Storage snapshots (Raft)
vault operator raft snapshot save vault.snap
```

## Common Failure Modes (Call Out)
- "Dev mode" or TLS disabled accidentally used outside local dev.
- Root token kept around and used for automation.
- Unseal/recovery keys stored together or stored on Vault hosts.
- Policies that allow broad `sys/*` access or wildcard secret reads.
- Audit logging enabled late or logs stored insecurely.
- Backups/snapshots exist but restores were never tested.

## References
- Use `references/vault-best-practices.md` for the full guide, snippets, and deeper checklists.
- Find topics quickly:
  - `rg -n \"Environment and Dependency Preparation|Systemd Service Example\" references/vault-best-practices.md`
  - `rg -n \"Vault Initialization and Unsealing|vault operator init|generate-root\" references/vault-best-practices.md`
  - `rg -n \"Integrated Storage \\(Raft\\)|Consul|Storage Backend Considerations\" references/vault-best-practices.md`
  - `rg -n \"TLS Setup and Recommended Configuration|tls_min_version|HSTS|Mutual TLS\" references/vault-best-practices.md`
  - `rg -n \"Audit Logging Setup|vault audit enable|Centralize\" references/vault-best-practices.md`

## Extended Guidance
Use this when planning upgrades, DR, or policy migrations.

## Upgrade Checklist
- Read release notes for breaking changes and deprecated flags.
- Snapshot/backup before upgrade and validate restore.
- Upgrade non-leader nodes first where possible.

## Policy Validation Checklist
- Run a least-privilege review for every policy update.
- Keep policies in version control and require peer review.
- Use `vault policy read` to verify the active policy content.

## DR and Recovery
- Test restore drills quarterly (or per compliance requirements).
- Keep recovery keys in separate locations and custody.

## Reference Index (Expanded)
- `rg -n "Recovery keys|generate-root" references/vault-best-practices.md`
- `rg -n "Audit logging|centralize" references/vault-best-practices.md`

## Operational Reminders
- Review audit log retention policies regularly.
- Validate TLS certificate rotation procedures.

## Quick Questions (When Stuck)
- What is the minimal change that solves the issue?
- What is the rollback plan?
- What is the highest-risk assumption?
- What is the simplest validation step?
- What is the known-good baseline?
- What evidence would change the decision?
- What is the user-visible impact?
- What is the operational impact?
- What is the most likely failure mode?
- What is the fastest safe experiment?

## Reference Index (Extra)
- `rg -n "Checklist|checklist" references/vault-best-practices.md`
- `rg -n "Example|examples" references/vault-best-practices.md`
- `rg -n "Workflow|process" references/vault-best-practices.md`
- `rg -n "Pitfall|anti-pattern" references/vault-best-practices.md`
- `rg -n "Testing|validation" references/vault-best-practices.md`
- `rg -n "Security|risk" references/vault-best-practices.md`
- `rg -n "Configuration|config" references/vault-best-practices.md`
- `rg -n "Deployment|operations" references/vault-best-practices.md`
- `rg -n "Troubleshoot|debug" references/vault-best-practices.md`
- `rg -n "Performance|latency" references/vault-best-practices.md`
- `rg -n "Reliability|availability" references/vault-best-practices.md`
- `rg -n "Monitoring|metrics" references/vault-best-practices.md`
- `rg -n "Error|failure" references/vault-best-practices.md`
- `rg -n "Decision|tradeoff" references/vault-best-practices.md`
- `rg -n "Migration|upgrade" references/vault-best-practices.md`
