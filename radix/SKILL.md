---
name: radix
description: >
  Build, secure, and operate Radix-based applications and integrations. Use when designing Scrypto
  blueprints, transaction manifests, badge/AccessRule authorization, wallet-first UX with RDT/ROLA,
  subintents and pre-authorizations, protocol-version compatibility, node/gateway operations,
  performance tuning, governance, token economics, and compliance/privacy patterns.
---

# Radix

## Workflow
1. Confirm target network and protocol update compatibility before coding.
2. Model authorization and asset flow with Radix-native primitives first.
3. Design transaction manifests for wallet clarity and explicit guarantees.
4. Implement blueprint logic with least-privilege roles and ownership controls.
5. Validate performance and fee behavior with realistic transaction patterns.
6. Build a layered testing pipeline (unit, simulator integration, wallet flow checks).
7. Prepare operations runbooks for nodes, monitoring, updates, and recovery.
8. Ship with governance, economics, and compliance/privacy decisions documented.

## Preflight (Ask / Check First)
- Target environment: local simulator, Stokenet, or Mainnet.
- Required protocol features (AccountLocker, subintents, transaction V2).
- Scrypto/CLI and Rust toolchain versions pinned for target protocol update.
- Wallet UX requirements and transaction flow complexity.
- Security model for admin, recovery, and user permissions.
- Throughput/latency/fee budgets and abuse-risk assumptions.
- Need for dedicated node infrastructure vs third-party gateway dependence.
- Regulatory constraints (KYC/AML, privacy policy, audit requirements).

## Operating Principles
- Treat protocol versioning as a design constraint, not an afterthought.
- Use capability-based authorization (badges, proofs, AccessRules) consistently.
- Keep manifests human-readable and wallet-conforming when possible.
- Keep on-ledger data minimal and non-sensitive by default.
- Separate protocol-ops governance from application governance.
- Prefer incremental, measurable changes over large transactional rewrites.

## Version and Compatibility Strategy
- Maintain a compatibility matrix across protocol updates and toolchain versions.
- Pin Scrypto, CLI tools, and Rust compiler in reproducible build config.
- Gate feature usage by protocol support in code and CI checks.
- Validate network IDs and transaction headers before submission.
- Keep migration notes for update-specific behavior changes.

### Compatibility Checklist
- Protocol update requirement documented.
- Toolchain versions pinned and reproducible.
- CI validates target network/protocol assumptions.
- Feature flags mapped to protocol capabilities.
- Rollback behavior defined for incompatible releases.

## Core Primitives and Transaction Model
- Use manifest-first design because wallet and reviewers depend on manifest clarity.
- Treat transactions as atomic for state, but not free on failure.
- Use assertions early to prevent expensive late-stage failure.
- Keep SBOR payload size and complexity under explicit control.
- Model ledger indexing with state version/epoch awareness for integrations.

### Transaction Safety Rules
- Assert expected outcomes early.
- Withdraw and stage buckets intentionally.
- Yield/deposit in clear order for multi-actor flows.
- Bound cost-unit and tip assumptions.
- Avoid giant all-in-one manifests when segmentation is safer.

## Authorization and Smart Contract Security
- Map each privileged method to explicit badge/proof requirements.
- Keep least-privilege role graphs and ownership paths documented.
- Treat proofs as evidence, not consumable value.
- Use buckets/escrow when “spending” voting or entitlement power.
- Choose `OwnerRole::Fixed` vs `OwnerRole::Updatable` deliberately.
- Secure owner badges in accounts/access controllers that can actually produce proofs.

### Security Validation Loop
1. Define role-to-method authorization map.
2. Write adversarial tests for proof misuse and replay-like patterns.
3. Validate manifests and semantics with toolkit/static checks.
4. Run integration tests for auth zone, deposits, and role transitions.
5. Perform external review for high-value economic logic.

## Wallet UX, RDT, Subintents, and AccountLocker
- Use Radix dApp Toolkit for wallet connection and transaction requests.
- Prefer conforming manifest patterns for high-confidence wallet previews.
- Publish metadata and verified links for trust attribution.
- Use subintents/pre-authorizations for delegated fees and multi-party flows.
- Follow subintent constraints (yield flow, fee lock placement, explicit assertions).
- Use AccountLocker for deferred deposits/claims instead of ad hoc deposit workarounds.

### UX Quality Checklist
- Manifest intent is understandable in wallet preview.
- Metadata keys and dApp definition links are complete.
- Fee payer and user guarantees are explicit.
- Multi-party transaction responsibilities are documented.
- Recovery/fallback path exists for rejected or expired intents.

## Performance and Scalability
- Treat execution/finalization/storage costs as first-class design metrics.
- Keep transaction/state updates small and predictable.
- Partition state to reduce contention and future-proof for sharding.
- Avoid hot singletons for high-throughput features.
- Use priority/tip strategy intentionally under contention.

## Tooling, Testing, and CI/CD
- Standardize team toolchain: `scrypto`, `resim`, `rtmc`, `rtmd`, `scrypto-bindgen`.
- Use Radix Engine Toolkit for static validation and manifest/tx processing.
- Run a three-layer test strategy:
  - unit tests (`scrypto-test`)
  - integration simulator tests (manifest/auth flows)
  - wallet-preview conformance checks
- Treat protocol updates as CI gates, not manual checks.
- Enforce lint rules for metadata and compatibility notes.

### Typical Validation Commands
```bash
scrypto test
resim reset
resim run ./manifests/happy_path.rtm
```

```bash
rtmc --manifest ./manifests/swap.rtm
rtmd --compiled ./manifests/swap.rtmc
```

## Governance and Upgradeability
- Separate network/protocol operations from app-level governance powers.
- Keep governance authority badge-based and auditable.
- Use multi-factor access controller patterns for high-impact admin actions.
- If using updatable ownership, document change process and delay controls.
- Prefer versioned deployments with explicit migration/route cutover plans.
- Lock mutable royalty/metadata/admin settings once finalized where possible.

## Economics, Fees, and Incentives
- Model user cost using Radix fee semantics, including committed failures.
- Keep royalty policy predictable; avoid surprise mutability.
- Consider fee delegation only with anti-abuse controls.

## Compliance and Privacy
- Treat on-ledger data as public and permanent.
- Keep PII off-ledger; store only references/proofs where required.
- Use Persona + ROLA patterns for off-ledger authentication.

## Deployment and Operations
- For critical workloads, run dedicated node infrastructure with hot-swappable backup.
- Automate node deployment and updates with reproducible configuration.
- Protect keystores, API endpoints, and metrics interfaces with strict access controls.
- Monitor health, sync, and performance with Prometheus/Grafana.
- Rehearse upgrade readiness and disaster recovery procedures.

### Operations Checklist
- Node and backup node health verified.
- Keystore backup and restore drill completed.
- Monitoring dashboards and alerts active.
- Update readiness process documented.
- Incident runbook includes failover and rollback steps.

## Ecosystem and Documentation
- Publish clear component/resource metadata and ownership attribution.
- Version documentation with protocol compatibility tables.
- Share reusable blueprints/integration examples where feasible.
- Keep changelogs and migration notes tied to releases.
- Align public docs with wallet behavior and manifest support.

## Common Failure Modes
- Using address-based auth habits instead of capability proofs.
- Treating proof amount as consumable authority.
- Shipping non-conforming manifests that degrade wallet safety UX.
- Assuming sharded parallelism guarantees not yet available on target network.
- Unpinned toolchains causing protocol compatibility breaks.
- Storing sensitive data directly on-ledger.
- Running production integrations without node redundancy or monitoring.

## Definition of Done
- Compatibility matrix and toolchain pins are current.
- Authorization model and ownership controls are explicit and tested.
- Wallet UX flows are conforming and user-safe.
- Performance/fee behavior is measured under representative load.
- CI validates manifests, semantics, and protocol assumptions.
- Operations runbooks and monitoring are production-ready.

## References
- `references/radix.md`

## Reference Index
- `rg -n "Scope, assumptions, and versioning|protocol updates|Cuttlefish|Bottlenose" references/radix.md`
- `rg -n "Transaction model|manifest|atomicity|Committed Failure|SBOR" references/radix.md`
- `rg -n "Authorization model|badges|AccessRules|AuthZone|proof" references/radix.md`
- `rg -n "dApp Toolkit|Gateway API|ROLA|subintents|pre-authorizations" references/radix.md`
- `rg -n "Security|code hardening|Access Controller|OwnerRole" references/radix.md`
- `rg -n "Performance|scalability|costing|tip|state partition" references/radix.md`
- `rg -n "testing|CI/CD|resim|RET|rtmc|rtmd" references/radix.md`
- `rg -n "Governance|upgradeability|backward compatibility|LTS" references/radix.md`
## Quick Questions (When Stuck)
- Is this problem protocol compatibility, auth modeling, or manifest design?
- Can this permission be narrowed with a badge/proof role split?
- Should this flow be subintent-based instead of one giant transaction?
- What evidence shows this design remains safe under adversarial behavior?
