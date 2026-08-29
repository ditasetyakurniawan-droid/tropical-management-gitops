# Tropical Management operations runbook

## Quick health view

```bash
kubectl -n test-app get pods -o wide
kubectl -n test-app get jobs
kubectl -n argocd get application tropical-management
```

## Application rollout status

```bash
for d in \
  tropical-api-gateway tropical-audit tropical-auth tropical-chat \
  tropical-dashboard tropical-inventory tropical-sales tropical-web
do
  echo "===== $d ====="
  kubectl -n test-app rollout status deployment/$d --timeout=120s
done
```

## Migration Job

```bash
kubectl -n test-app get job tropical-db-migrator
kubectl -n test-app get pods -l app=tropical-db-migrator
```

Latest migrator log:

```bash
MIG_POD=$(kubectl -n test-app get pods \
  -l app=tropical-db-migrator \
  --sort-by=.metadata.creationTimestamp \
  -o jsonpath='{.items[-1:].metadata.name}')

kubectl -n test-app logs "$MIG_POD" -c tropical-db-migrator --timestamps
```

Success ends with:

```text
event=migration_run_completed targets=5
```

## If PreSync fails

Do not keep force-retrying and do not delete migration metadata first.

Check in this order:

1. migrator container log;
2. `vault-agent-init` log;
3. rendered target DB name without printing passwords;
4. DB reachability/privileges;
5. checksum/dirty-state message;
6. Argo CD operation message/events.

Example Vault init log:

```bash
kubectl -n test-app logs "$MIG_POD" -c vault-agent-init --timestamps
```

A checksum mismatch often means an applied migration file changed or two migration targets are accidentally using the same database. Fix ownership/configuration rather than weakening checksum enforcement.

## Verify target DB without printing credentials

For a running workload, parse only the database segment from the injected DSN file. Example:

```bash
kubectl -n test-app exec deploy/tropical-chat -c vault-agent -- sh -lc '
  dsn=$(cat /vault/secrets/chat-db)
  db=${dsn##*/}
  db=${db%%\?*}
  echo "CHAT_DB=$db"
'
```

Expected: `tropical_chat`.

## Health endpoints

For Go backends:

```text
/livez   must not depend on MySQL/downstream
/readyz  DB-backed services depend on MySQL
```

Readiness failures should remove the pod from traffic without creating liveness restart storms.

## Argo CD state

```bash
kubectl -n argocd get application tropical-management \
-o jsonpath='sync={.status.sync.status}{"\n"}health={.status.health.status}{"\n"}phase={.status.operationState.phase}{"\n"}revision={.status.operationState.syncResult.revision}{"\n"}message={.status.operationState.message}{"\n"}'
```

Normal steady state:

```text
sync=Synced
health=Healthy
phase=Succeeded
```

## Rollback rule

Prefer Git revert/image-tag rollback through GitOps. Do not make long-lived manual cluster mutations. Before code rollback, verify schema backward compatibility. Do not manually rewrite `schema_migrations` as a shortcut.

## Vault change rule

A Vault secret version change is not automatically reread by a running pod using pre-populated files. Recreated pods receive the latest secret. Planned secret rotation should therefore include a controlled workload rollout and verification.

## Evidence after a high-risk change

Capture:

- application commit and Jenkins build;
- Harbor tag/digest;
- GitOps revision;
- migration Job result/log;
- Argo CD result;
- post-rollout pod readiness/restarts;
- any DB metadata verification required by the change.
