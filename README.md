# Tropical Management GitOps

Repository ini adalah **source of truth deployment Kubernetes** untuk Tropical Restaurant Management.

## Responsibility split

- `tropical-management-v1`: source code, tests, Dockerfiles, Jenkins pipeline.
- `tropical-management-gitops`: Kubernetes desired state, Kustomize overlays, Argo CD definitions.

Jenkins membangun dan menguji aplikasi, push immutable image ke Harbor, lalu mengubah image tag di repository ini. Argo CD mendeteksi commit tersebut dan melakukan reconciliation ke cluster.

## Current Application Phase

Phase 3 sudah menambahkan:

- Premium tropical UI system
- RBAC admin/auditor/staff
- Internal audit workflow
- Inventory workflow
- General Live Chat realtime

Live Chat menggunakan service terpisah:

```
web -> api-gateway -> chat-service -> MySQL
```

Fitur:

- shared general room
- persistent message history
- Server Sent Events realtime delivery
- role badge
- user name accent color
- JWT based identity validation

Detail development tersedia di:

```
docs/phase3-live-chat-development-notes.md
```

## Repository layout

```text
apps/tropical-management/
  base/
  overlays/dev/
  overlays/prod/
argocd/
docs/
```

## Security rules

Repository ini tidak boleh berisi password, Vault token, database credential, kubeconfig, private key, Harbor credential, atau Kubernetes Secret plaintext.

Secret runtime berasal dari HashiCorp Vault.

## Next Deployment Phase

- Add chat-service Kubernetes workload
- Add Vault Agent injection
- Add Harbor image promotion flow
- Enable Argo CD automated sync after validation
- Add HPA/PDB/NetworkPolicy
