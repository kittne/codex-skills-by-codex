# Filesystem Backup Reference (2026-02-18)

## Context7 Sources
- `/restic/restic`
- `/websites/openzfs_github_io_openzfs-docs`
- `/websites/btrfs_readthedocs_io_en`

## Practical Guidance
- Use snapshots for fast local rollback, plus offsite replicated backups for DR.
- Keep immutable/read-only snapshot baselines for incremental send/receive workflows.
- Apply explicit retention using restic `forget` policies and safe dry-run checks.
- Run integrity checks regularly (`restic check --read-data`, pool/subvolume scrubs).
- Rehearse restores and record measured RTO.

## Command Anchors
```bash
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --prune
restic check --read-data
zfs send -R pool/fs@snap | ssh backup "zfs receive backup/fs"
btrfs subvolume snapshot -r /data /snapshots/data-2026-02-18
```

## Source URLs
- <https://github.com/restic/restic>
- <https://openzfs.github.io/openzfs-docs/>
- <https://btrfs.readthedocs.io/en/latest/>
