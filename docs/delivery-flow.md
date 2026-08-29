# Tropical Management delivery flow

## Source-of-truth model

```text
Application repository              GitOps repository
----------------------              -----------------
Go / Next.js source                 Kustomize manifests
Tests                               Vault injection metadata
Dockerfiles                         image tags
Versioned SQL migrations            Argo CD hooks/application
Jenkinsfile                         deployment policy
```

Jenkins is CI plus image publisher/tag updater. Argo CD is CD/reconciliation.

## Normal release

```text
PR merged to app main
  -> Jenkins tests/vet/frontend/Sonar
  -> build app + db-migrator images
  -> push immutable tag to Harbor
  -> Jenkins updates GitOps image tags
  -> Argo CD sees OutOfSync
  -> automated full Sync
  -> PreSync db-migrator
       -> Vault pre-populates 5 DB DSN files
       -> migrate auth/audit/inventory/sales/chat
       -> fail closed on dirty/checksum/SQL/verification failure
  -> Job Complete 1/1
  -> Deployment rollout
  -> readiness admits healthy pods
```

The Phase B P0.2 rollout validated this path end-to-end without manual sync/restart.

## Why PreSync

Schema compatibility must be established before new application code is started. A migration failure therefore blocks the release rather than creating partially upgraded pods.

The migration Job is configured as a hook rather than a permanent workload. `BeforeHookCreation` allows a new sync to replace the previous completed Job, preserving the latest result for inspection between releases.

## Vault behavior

Migration annotations use Vault Agent injection with `agent-pre-populate-only: "true"`.

Required files become environment pointers for the migrator:

```text
AUTH_DB_DSN_FILE
AUDIT_DB_DSN_FILE
INVENTORY_DB_DSN_FILE
SALES_DB_DSN_FILE
CHAT_DB_DSN_FILE
```

Do not put DSN values in GitOps YAML.

## Release invariants

- normal release does not require manual Argo CD Sync;
- normal release does not require `kubectl rollout restart`;
- a failed migration means no application rollout;
- application runtime startup does not execute schema DDL;
- immutable image tags are used;
- desired state changes are traceable through Git history;
- rollback must consider database backward compatibility.

## Verification

```bash
kubectl -n test-app get jobs
kubectl -n test-app get pods
kubectl -n argocd get application tropical-management
```

Expected after a successful release:

```text
tropical-db-migrator  Complete  1/1
Argo CD               Synced / Healthy / Succeeded
application pods      Running, readiness healthy, stable restarts
```
