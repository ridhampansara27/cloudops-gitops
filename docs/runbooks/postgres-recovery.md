# PostgreSQL backup and restore runbook

## Backup layers

- **Logical:** OCI GitOps schedules a custom-format `pg_dump` at 02:15 UTC daily, uploaded under `logical/postgres/oci-dev/` to a private Object Storage bucket. Bucket lifecycle deletes objects after 30 days.
- **Volume:** Terraform assigns daily incremental backups retained seven days and weekly incremental backups retained 28 days to the PostgreSQL OCI Block Volume.
- **Encryption recovery:** Terraform manages a protected OCI Vault/key for Sealed Secrets controller recovery. Preserve the controller's key recovery procedure and restricted operator access separately.

## Verify routinely

1. Check the CronJob is unsuspended and inspect the latest Job completion and failure events.
2. Check that a new object exists in the expected bucket prefix. Compare timestamp and size with the Job record; keep the PAR URL and database credentials out of logs.
3. Periodically download an authorized backup to an isolated environment, run `pg_restore --list`, restore to an empty **disposable** PostgreSQL 17 database, and run application-level integrity checks.
4. Record the backup object identifier, restore date, PostgreSQL version, application/GitOps revision, outcome, and operator. Never publish a dump or secret in Git.

## Incident recovery sequence

1. Identify the incident time, latest clean recovery point, tenant/data impact, and compatibility with the intended application image.
2. Preserve evidence. Provision an isolated target and obtain a read-only copy of a verified logical backup or restore an OCI volume backup into an isolated volume.
3. Restore and validate **outside production**. Check Alembic revision, key tables, tenant boundaries, and application readiness.
4. Plan any production cutover, including downtime, credentials, customer impact, rollback point, and post-cutover validation, as a separately approved operation.

The successful commercial-launch drill establishes that restoration was demonstrated at that baseline; it does not guarantee every later backup is usable. No command in this document performs a live restore.
