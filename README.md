# Tropical Management GitOps

Repository ini adalah **source of truth deployment Kubernetes** untuk Tropical Restaurant Management.

## Responsibility split

- `tropical-management-v1`: source code, tests, Dockerfiles, Jenkins pipeline.
- `tropical-management-gitops`: Kubernetes desired state, Kustomize overlays, Argo CD definitions.

Jenkins **tidak melakukan `kubectl apply` ke production**. Jenkins membangun dan menguji aplikasi, push immutable image ke Harbor, lalu mengubah image tag di repository ini. Argo CD mendeteksi commit tersebut dan melakukan reconciliation ke cluster.

## Repository layout

```text
apps/tropical-management/
  base/                 reusable non-secret resources
  overlays/dev/         development desired state
  overlays/prod/        production desired state
argocd/                 Argo CD Application definitions
docs/                   architecture and operating notes
```

## Security rules

Repository ini tidak boleh berisi password, Vault token, database DSN dengan credential, kubeconfig, private key, Harbor credential, atau Kubernetes Secret plaintext. Secret runtime akan berasal dari HashiCorp Vault.

> Workload Deployment sengaja belum diaktifkan pada bootstrap commit. Aplikasi saat ini masih membaca secret dari environment variables. Sebelum production deployment, backend akan ditambah dukungan `*_FILE` agar aman membaca file hasil Vault Agent injection tanpa shell wrapper.
