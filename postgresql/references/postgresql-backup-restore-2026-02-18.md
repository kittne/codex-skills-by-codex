# PostgreSQL Backup/Restore Reference (2026-02-18)

## Context7 Sources
- Library: `/websites/postgresql_17`
- PostgreSQL docs for continuous archiving and PITR.

## Practical Guidance
- Use base backups plus WAL archiving for PITR workflows.
- Verify `archive_command` writes and archive continuity.
- Configure and test `restore_command` in isolated recovery runs.
- Treat logical dumps (`pg_dump`) as complementary, not PITR replacement.
- Rehearse timestamp-targeted recoveries and record measured RTO.

## Command Anchors
```sql
archive_command = 'test ! -f /mnt/server/archivedir/%f && cp %p /mnt/server/archivedir/%f'
restore_command = 'gunzip < /mnt/server/archivedir/%f.gz > %p'
```

## Source URLs
- <https://www.postgresql.org/docs/17/continuous-archiving>
- <https://www.postgresql.org/docs/17/backup>
