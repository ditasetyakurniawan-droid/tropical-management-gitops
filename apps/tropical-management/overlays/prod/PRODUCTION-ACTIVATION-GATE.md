# Production Overlay Activation Gate

The `prod` overlay is intentionally dormant. Do not point an Argo CD Application at it until all items below are complete.

- [ ] Production Vault role and policy exist.
- [ ] Production Vault secret path exists and contains every required application secret.
- [ ] Every inherited plaintext development credential has been removed or overridden by a production Vault `*_FILE` patch.
- [ ] `APP_ENV=production` is set on sensitive application workloads.
- [ ] Production resource requests/limits are sized from `test-app` measurements.
- [ ] Namespace LimitRange and ResourceQuota are sized from real cluster capacity.
- [ ] NetworkPolicy is rendered and connectivity-tested.
- [ ] Backup and restore procedure has been tested.
- [ ] Kustomize render contains no development DSN or bootstrap password.
- [ ] `kubectl diff` / Argo CD diff has been reviewed before first sync.

Do not copy `test-app` Vault paths into this overlay. Production must use its own Vault role/path and an explicit promotion review.
