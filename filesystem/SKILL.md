---
name: filesystem
description: >
  Design and operate filesystem snapshot backup and recovery workflows for production systems.
  Use for snapshot policy design, incremental replication, retention governance, integrity checks,
  restore drills, and disaster recovery runbooks across tools like Restic, OpenZFS, and Btrfs.
---

# Filesystem Backups

## Workflow
1. Confirm RPO/RTO targets and data classification.
2. Choose snapshot and backup primitives by platform.
3. Define snapshot cadence and retention policy.
4. Configure replication or offsite backup transport.
5. Implement integrity verification and scrub/check schedules.
6. Run restore drills and measure recovery timing.
7. Operationalize monitoring, alerting, and runbooks.

## Preflight (Ask / Check First)
- Filesystem and storage platform (ZFS, Btrfs, other).
- Backup destination type (local, object storage, remote host).
- Encryption and key management requirements.
- Recovery objectives by dataset criticality.
- Current restore-test frequency and outcomes.

## Strategy Selection
- Use filesystem snapshots for fast point-in-time rollback.
- Use offsite replicated backups for disaster scenarios.
- Separate hot operational snapshots from long-term archives.
- Keep retention policy explicit and automated.
- Align policy with legal/compliance retention constraints.

## Snapshot and Replication Patterns
- Create immutable/read-only snapshots for incremental replication baselines.
- Use incremental send/receive where supported to reduce transfer cost.
- Protect critical snapshots with holds/pins when available.
- Keep dataset/subvolume layout designed for restore granularity.

### Example Commands
```bash
# OpenZFS
zfs snapshot pool/data@daily-2026-02-18
zfs send -i pool/data@daily-2026-02-17 pool/data@daily-2026-02-18 | ssh backup "zfs receive backuppool/data"

# Btrfs
btrfs subvolume snapshot -r /data /snapshots/data-2026-02-18
```

## Retention and Repository Hygiene
- Apply time-based retention tiers (daily/weekly/monthly/yearly).
- Dry-run retention changes before destructive prune/forget operations.
- Run repository integrity checks on a schedule.
- Track storage growth and prune impact.

## Restore Drills and Verification
- Test file-level, directory-level, and full-volume restores.
- Measure real recovery time against RTO targets.
- Validate application consistency after restore.
- Document deterministic restore steps and fallback paths.
- Rehearse key-loss and credential-loss contingencies.

## Operations and Monitoring
- Alert on snapshot/backup job failures and lag.
- Monitor replication delay and backlog growth.
- Track scrub/check results and corruption signals.
- Keep backup metadata and logs centralized.
- Review backup policy quarterly with stakeholders.

## Security Controls
- Encrypt backup data at rest and in transit.
- Scope backup credentials to minimum required permissions.
- Rotate keys and credentials on defined cadence.
- Enforce access controls and audit trails for restore operations.

## Validation Commands
```bash
restic check --read-data
zpool scrub tank
btrfs scrub status /
```

## Common Failure Modes
- Assuming backups are valid without restore drills.
- Retention policy misconfiguration deleting needed recovery points.
- Mutable snapshots breaking incremental replication expectations.
- No offsite copy for regional failure scenarios.
- Backup credentials stored in plaintext configs.

## Definition of Done
- Backup architecture meets documented RPO/RTO targets.
- Snapshot, replication, and retention are automated and observable.
- Integrity checks and restore drills run on schedule.
- Security controls protect data and credentials.
- Recovery runbooks are tested and current.

## References
- `references/filesystem-2026-02-18.md`

## Reference Index
- `rg -n "snapshot|send|receive|subvolume" references/filesystem-2026-02-18.md`
- `rg -n "retention|forget|prune|check" references/filesystem-2026-02-18.md`
- `rg -n "scrub|integrity|restore" references/filesystem-2026-02-18.md`
- `rg -n "security|encryption|credentials" references/filesystem-2026-02-18.md`
