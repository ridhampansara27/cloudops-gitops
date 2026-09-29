# Contributing

Branch from current `main` and keep each environment promotion focused. Never replace an OCI image tag with `latest` or merge an unreviewed change to backup, secrets, Argo CD root selection, or destructive resources.

Run Helm lint/render for the default chart and affected overlays. CI lints and renders every overlay and checks generated YAML. In the PR, state the image SHA, affected environment, diff summary, migration compatibility, backup status, rollback approach, and whether a controlled OCI Argo CD sync is needed.

Development promotion PRs are separate from OCI production promotion. Do not close or change open deployment PRs as part of documentation cleanup.
