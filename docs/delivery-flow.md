# Jenkins + Harbor + Argo CD Delivery Flow

```text
GitHub application repo
        |
        v
      Jenkins
  test / vet / build / scan
        |
        v
      Harbor
 immutable image:<git-sha>
        |
        v
Jenkins updates image tag here
        |
        v
   GitOps commit / PR
        |
        v
      Argo CD
        |
        v
    Kubernetes
```

## Rules

1. Image tags must be immutable Git SHAs; avoid `latest` in production.
2. Jenkins may publish images and propose/update desired state, but Argo CD owns cluster reconciliation.
3. Production secret values never enter Git.
4. Promotion to production is a Git change from a tested image SHA, giving an auditable rollback point.
5. Argo CD automated sync remains off until the bootstrap stack has passed local and cluster integration tests.
