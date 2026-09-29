# GitOps architecture and PostgreSQL protection

## OCI application tree

```mermaid
flowchart TB
    MAIN["GitOps main"] --> ROOT["cloudops-oci-root · auto reconcile Application CRs"]
    ROOT --> APP["cloudops-oci-dev · manual workload sync"]
    ROOT --> SEALED["Sealed Secrets · manual controller sync"]
    APP --> HELM["Shared Helm chart + OCI values"]
    HELM --> FRONT["Frontend · NGINX · Cloudflare Tunnel"]
    HELM --> API["FastAPI · Celery · PostgreSQL · Redis"]
    classDef source fill:#dbeafe,stroke:#2563eb,color:#172554;
    classDef gate fill:#fef3c7,stroke:#d97706,color:#78350f;
    classDef package fill:#ede9fe,stroke:#7c3aed,color:#2e1065;
    classDef runtime fill:#dcfce7,stroke:#16a34a,color:#14532d;
    class MAIN,ROOT source;
    class APP,SEALED gate;
    class HELM package;
    class FRONT,API runtime;
```

The OCI root uses only `argocd/oci-applications`. The older `argocd/root-app.yaml` selects `argocd/applications` and must not be confused with the OCI root.

## PostgreSQL backup and isolated restore

```mermaid
flowchart LR
    PG[("PostgreSQL PVC · OCI Block Volume")] -->|"02:15 UTC daily"| DUMP["CronJob · pg_dump -Fc"]
    DUMP --> FILE["Temporary dump"]
    FILE --> UP["Uploader · scoped PAR Secret"]
    UP --> OBJ[("Private OCI Object Storage · 30-day lifecycle")]
    PG -->|"OCI policy"| VOL[("Daily and weekly volume backups")]
    OBJ -->|"operator selects verified artifact"| LAB["Isolated restore target"]
    LAB --> CHECK["Integrity and application checks"]
    classDef data fill:#dcfce7,stroke:#16a34a,color:#14532d;
    classDef process fill:#ede9fe,stroke:#7c3aed,color:#2e1065;
    classDef safe fill:#dbeafe,stroke:#2563eb,color:#172554;
    class PG,OBJ,VOL data;
    class DUMP,FILE,UP process;
    class LAB,CHECK safe;
```

The dump container gets the database password but not the OCI upload credential; the uploader gets the scoped upload credential but not the database password. They share only the temporary dump. CronJob concurrency is `Forbid`, and OCI values set `enabled: true` and `suspend: false`. The one-shot validation job remains disabled by default. Terraform independently configures volume backups, bucket retention, and recovery-key vault resources.

The logical dump is not itself proof of recoverability. A restore drill was completed for the certified launch; future operators should record the artifact, isolated target, integrity result, and date without publishing secrets. See [recovery runbook](runbooks/postgres-recovery.md).
