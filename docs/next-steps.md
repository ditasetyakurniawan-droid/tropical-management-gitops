# Bootstrap next steps

1. Make local Docker Compose build green.
2. Add `*_FILE` configuration support to Go services for Vault Agent injected files.
3. Add one Deployment/Service per microservice and web frontend.
4. Add Harbor image names and immutable tags through Kustomize.
5. Add readiness/liveness probes and resource requests/limits.
6. Add Vault Agent annotations and dedicated Kubernetes auth role.
7. Add PodDisruptionBudget, HPA and NetworkPolicy.
8. Add ingress and TLS only after the internal service path is healthy.
9. Extend Jenkins to update the GitOps image tag after Harbor push.
10. Enable Argo CD automated sync first for dev, then promote to prod.
