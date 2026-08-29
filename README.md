# Tropical Management GitOps

This repository is the Kubernetes deployment source of truth for Tropical Management. Application code belongs in `tropical-management-v1`; deployment state and release orchestration belong here.

## Current delivery architecture

```text
Application GitHub main
        |
        v
      Jenkins
        |
        +--> tests / Sonar
        +--> Harbor immutable images
        `--> GitOps image tag update
                    |
                    v
                  Argo CD
                    |
                    +--> PreSync tropical-db-migrator
                    |        `--> Vault -> MySQL
                    |
                    `--> Kubernetes Deployments
```

Normal application releases are automatic. Jenkins changes desired image tags; Argo CD reconciles them. A database migration failure stops the sync before new application pods roll out.

## Repository responsibilities

- Kubernetes Deployments and Services
- Kustomize base/overlays
- Vault Agent injection configuration
- Argo CD Application definitions
- migration PreSync hook
- environment-specific desired image tags
- platform deployment runbooks

This repository must not contain database passwords, Vault tokens, kubeconfig, private keys, Harbor credentials, or plaintext application Secrets.

## Runtime environment

```text
Kubernetes control plane  192.168.100.51-.53
Kubernetes workers        192.168.100.54-.56
Jenkins                   192.168.100.57
Harbor                    192.168.100.58
SonarQube + ELK           192.168.100.59
MySQL DB host             192.168.100.70
Application namespace     test-app
Registry                  harbor-dt.co.id/devops-apps
```

The DB host currently runs MySQL 8.0 in Docker plus mysqld-exporter and phpMyAdmin. This is production-like homelab infrastructure, not a guarantee of database HA.

## Current application workloads

- `tropical-api-gateway`
- `tropical-audit`
- `tropical-auth`
- `tropical-chat`
- `tropical-dashboard`
- `tropical-inventory`
- `tropical-sales`
- `tropical-web`
- `tropical-db-migrator` as an Argo CD PreSync Job

## Health/lifecycle policy

Go backend Deployments use:

```text
startupProbe    /livez
readinessProbe  /readyz
livenessProbe   /livez
terminationGracePeriodSeconds: 30
```

DB-backed readiness depends on MySQL. Liveness does not.

## Database migration policy

`tropical-db-migrator` runs as `argocd.argoproj.io/hook: PreSync` with `restartPolicy: Never`. Vault Agent uses pre-populate-only mode so secret files exist before the Job starts and the Job can terminate normally.

Migration target ownership:

```text
auth       tropical_auth
audit      tropical_audit
inventory  tropical_inventory
sales      tropical_sales
chat       tropical_chat
```

See [`docs/delivery-flow.md`](docs/delivery-flow.md) and [`docs/operations-runbook.md`](docs/operations-runbook.md).

## Current status

- Vault runtime injection: **implemented**.
- Harbor immutable image delivery: **implemented**.
- Argo CD automated application sync: **implemented**.
- production-like health probes/graceful shutdown: **implemented**.
- PreSync versioned DB migration: **implemented and E2E validated**.
- resources/PDB/NetworkPolicy/least-privilege hardening: **next**.
- ELK application integration/central observability: **not yet complete**.

See [`docs/next-steps.md`](docs/next-steps.md).
