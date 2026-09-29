# OCI deployment and rollback runbook

**Scope:** `cloudops-oci-dev` on OCI OKE. This is a procedure for an authorized operator, not an automatic deployment instruction.

## Before promotion

1. Record application SHA, GitOps base SHA, OCI namespace, active Argo CD revision, and most recent successful logical backup.
2. Check application CI, GHCR `linux/arm64` manifest, security scans, and Alembic migration compatibility.
3. Review the OCI values diff and `helm template` output. Verify no change to signup, secrets, canonical origin, backup suspension, or destructive resource pruning unless explicitly intended.
4. Merge the reviewed OCI GitOps PR. Diff the child Argo CD application against live resources and sync only after prerequisites are met.
5. Observe rollout, `/health/live`, `/health/ready`, sign-in, tenant-scoped API behavior, worker/Beat health, and backup scheduling. Record the deployed GitOps and image SHAs.

## Rollback decision

- For a compatible application-only regression, prepare a reviewed GitOps revert to the last known-good image tags. Inspect the Helm/Argo diff before controlled sync.
- If a database migration changed schema or data, first determine whether older code is compatible. A Git revert or old container tag does **not** reverse a migration.
- For data loss or corruption, stop and use the [isolated restore procedure](postgres-recovery.md) to assess a verified recovery point. Production data replacement needs a separate explicit decision and maintenance plan.
- Preserve incident evidence and the previous manifests; verify frontend, API, worker, and backup behavior after any recovery.

Do not delete the OCI namespace, PVC, backup bucket, Sealed Secrets key, or Terraform state during rollback.
