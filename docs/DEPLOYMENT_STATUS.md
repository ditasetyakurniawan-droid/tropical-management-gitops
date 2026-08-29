# Tropical Management Deployment Status

> Canonical deployment and GitOps status for Tropical Steak House Internal OS.

Last updated: 2026-08-30

---

## Current Release

| Field | State |
|---|---|
| Release | `1.1.0-beta1` |
| Environment | `test-app` |
| GitOps revision | `930fde5de3c1863abcecfc1186dd538afc3e4cd3` |
| Workforce image | `harbor-dt.co.id/devops-apps/tropical-workforce:1.1.0-beta1-ed910abda3bd` |
| Argo CD sync | **Synced** |
| Argo CD health | **Healthy** |
| Last operation | **Succeeded** |
| Workforce deployment | **1/1 Ready** |
| Production | **Dormant** |
| Current stage | **Functional Validation / UAT** |

---

## Deployment Summary

Release 1.1 **Workforce & Shift Operations** is active in the internal
`test-app` environment.

Infrastructure deployment is complete.

Validated state:

```text
Application main    ed910abda3bd
Release             1.1.0-beta1
Workforce image     1.1.0-beta1-ed910abda3bd

Argo Sync           Synced
Argo Health         Healthy
Operation           Succeeded

Workforce Desired   1
Workforce Ready     1
Workforce Available 1

/livez              HTTP 200
/readyz             HTTP 200
```

This release is no longer in pre-activation or deployment smoke-test stage.

The current release gate is:

```text
FUNCTIONAL VALIDATION / UAT
```

---

## Delivery Pipeline

```text
Application main
       ↓
Jenkins
       ↓
Backend tests
       ↓
Frontend tests/build
       ↓
SonarQube
       ↓
Container build
       ↓
Harbor
       ↓
GitOps image update
       ↓
Argo CD
       ↓
PreSync database migration
       ↓
Kubernetes rollout
       ↓
Runtime health verification
```

---

## Workforce Runtime

### Deployment

```text
Deployment: tropical-workforce
Namespace:  test-app
Replicas:   1
Ready:      1
Available:  1
```

### Image

```text
harbor-dt.co.id/devops-apps/tropical-workforce:1.1.0-beta1-ed910abda3bd
```

### Service

```text
Service: tropical-workforce
Port:    8080
Type:    ClusterIP
```

---

## Database Migration

The centralized database migrator currently manages six targets:

```text
auth
audit
inventory
sales
chat
workforce
```

All six targets completed successfully during Release 1.1 activation.

Workforce schema:

```text
tropical_workforce
```

The database server runs outside the Kubernetes cluster.

No credentials or DSN values are stored in Git.

---

## Vault Integration

Workforce uses the existing Vault Agent Injector pattern.

Vault data key:

```text
WORKFORCE_DB_DSN
```

Runtime flow:

```text
Kubernetes ServiceAccount
        ↓
Vault Kubernetes Auth
        ↓
Vault Agent Injector
        ↓
/vault/secrets/workforce-db
        ↓
workforce-service
```

Human operator credentials are not used by application workloads.

---

## Activation Incident Record

Two configuration gaps were discovered during Release 1.1 activation.

### Invalid workforce DSN

The initial workforce DSN did not contain a valid MySQL database path.

Resolution:

```text
WORKFORCE_DB_DSN
→ valid tropical_workforce database path
```

### Missing database privilege

The existing runtime database account was authenticated successfully,
but did not initially have access to the new workforce schema.

Resolution:

```text
tropical_workforce.*
→ granted to the existing tropical database account
```

Existing schemas were not replaced or removed.

Affected existing applications remained unchanged:

- auth
- audit
- inventory
- sales
- chat

---

## Environment Safety

### test-app

```text
tropical-workforce replicas: 1
```

### Shared base

```text
tropical-workforce replicas: 0
```

### production

Production remains intentionally dormant.

Test environment activation must never implicitly activate production.

---

## Current Validation Stage

Infrastructure gates:

| Gate | Status |
|---|---|
| Image published | ✅ |
| Vault integration | ✅ |
| Database provisioned | ✅ |
| Database migration | ✅ |
| Argo sync | ✅ |
| Argo health | ✅ |
| Deployment rollout | ✅ |
| Liveness probe | ✅ |
| Readiness probe | ✅ |
| UI route available | ✅ |
| Functional UAT | ⏳ |

---

## Next Gate

Functional validation should cover:

- Owner workforce operations
- PIC workforce operations
- Karyawan self-service workflow
- shift creation
- checklist workflow
- clock-in / clock-out
- attendance history
- time-off request
- approval flow
- API authorization
- role isolation

After these scenarios pass, Release 1.1 can move from:

```text
FUNCTIONAL VALIDATION / UAT
```

to:

```text
RELEASE ACCEPTED
```

---

## Known Technical Debt

### Kustomize

The current configuration still uses deprecated syntax:

- `commonLabels`
- `patchesStrategicMerge`

These warnings are non-blocking for Release 1.1.

### Security backlog

The deferred security-hardening sprint still includes:

- supported Go toolchain upgrade
- `govulncheck`
- dependency scanning
- container image scanning
- NetworkPolicy
- ServiceAccount / RBAC review
- runtime vs migration DB privilege separation
- secret rotation runbook

These items must remain visible before production activation or external
exposure.

---

## Documentation Convention

Engineering documentation is **English-first**.

Use English for:

- architecture
- infrastructure
- APIs
- release states
- CI/CD
- security
- GitOps
- Kubernetes
- technical decisions

Bahasa Indonesia may still be used when it improves clarity for:

- restaurant operations
- product terminology
- internal user workflows
- contextual notes

Avoid translating standard technical terminology unnecessarily.
