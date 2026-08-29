# Tropical Management Phase 3 Development Notes

## Current Architecture

The application is split into:

- Application repository: `tropical-management-v1`
- GitOps repository: `tropical-management-gitops`

Current delivery flow:

```text
Developer -> Git commit/merge -> Jenkins -> Sonar/tests -> Harbor -> GitOps image tag update -> Argo CD -> PreSync DB migration -> Kubernetes rollout
```

Application runtime pods do not own schema migration during startup. Database schema changes are executed by the standalone `tropical-db-migrator` before rollout.

## Current Feature Status

Implemented:

- RBAC admin/auditor/staff
- Internal audit workflow
- Inventory management
- Premium tropical UI system
- General Live Chat

## Live Chat Architecture

`chat-service` is an independent Go microservice.

Responsibilities:

- Store messages in MySQL
- Validate authenticated user identity
- Broadcast messages using Server Sent Events
- Maintain one global room for all authenticated users

Identity source:

API Gateway verifies JWT and forwards:

- user id
- display name
- role

The chat service never trusts browser submitted identity data.

Current database ownership:

```text
chat-service -> tropical_chat
```

The previous shared-schema arrangement with `tropical_auth` has been retired. Migration metadata is maintained independently in `tropical_chat.schema_migrations`.

## Local Development

Existing local environments with already initialized database volumes may still start with:

```bash
docker compose up -d
```

However, after P0.2 the runtime services no longer execute schema DDL during startup. A **fresh** Docker Compose environment must not rely on service startup migrations.

P0.2.1 remains open to make local bootstrap match Kubernetes:

```text
MySQL ready
  -> standalone db-migrator
  -> schema ready
  -> application services
```

Until that parity work is complete, do not treat `docker compose up -d` against an empty database volume as the production-equivalent bootstrap path.

## Deployment Status and Remaining Work

Implemented:

- Kubernetes Deployment/Service for `chat-service`
- Vault Agent secret-file injection
- Harbor immutable image delivery
- Jenkins GitOps image-tag update
- Argo CD automated synchronization
- `/livez` and `/readyz` probes
- graceful shutdown
- versioned PreSync database migration
- fail-closed migration behavior

Remaining platform hardening:

- measured CPU/memory requests and limits
- replica/rolling-update strategy
- PodDisruptionBudget for eligible workloads
- HPA after resource requests and metrics are reliable
- NetworkPolicy and RBAC review
- ingress/TLS lifecycle hardening
- shared pub/sub such as Redis or NATS before horizontally scaling `chat-service`

## Development Rule

Every new service must include:

- Dockerfile
- `/livez` and `/readyz` health contract
- environment and secret-file documentation
- Kubernetes manifest
- GitOps overlay update
- migration impact assessment
- observability and rollback notes
- README/documentation update
