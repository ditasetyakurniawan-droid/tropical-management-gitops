# Release 1.1 Workforce GitOps

Release 1.1 adds Kubernetes delivery support for the Tropical Steak House
workforce service.

## Included

- tropical-workforce Deployment
- tropical-workforce Service
- WORKFORCE_SERVICE_URL gateway routing
- Vault-injected WORKFORCE_DB_DSN
- workforce DB migrator wiring
- test-app runtime resource/security configuration
- Kustomize image management

## Activation gate

The workforce Deployment remains at `replicas: 0` until the application
pipeline has published the tropical-workforce image and the workforce
database migration is ready.

## Vault prerequisite

The existing test application Vault secret must contain the key:

`WORKFORCE_DB_DSN`

The DSN value itself must never be committed to Git.

## Production

Production remains dormant and workforce is not activated as part of
Release 1.1.

<!-- release-1.1-deployment-outcome -->

## Deployment Outcome

Release 1.1 is active in the internal `test-app` environment.

Current state:

- Application image: `1.1.0-beta1-ed910abda3bd`
- Argo CD: **Synced**
- Application health: **Healthy**
- Database migration: **Succeeded**
- Workforce deployment: **1/1**
- Production: **Dormant**
- Release stage: **Functional Validation / UAT**

Infrastructure delivery and smoke validation are complete.

See [`DEPLOYMENT_STATUS.md`](DEPLOYMENT_STATUS.md) for the canonical
deployment record.
