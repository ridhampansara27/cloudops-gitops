# Security policy

Report vulnerabilities privately through GitHub private vulnerability reporting when available, or contact the maintainer privately. Never put token values, plaintext PAR URLs, Kubernetes Secret contents, or customer data into issues or PRs.

OCI application Secrets are external or stored as strictly scoped Sealed Secrets ciphertext. Review scope, namespace/name binding, image provenance, migration behavior, Argo CD pruning, and backup effects for each production PR. The OCI child application requires controlled synchronization; the root automatically reconciles only Application resources. The historical AWS application tree is not selected by the OCI root.
