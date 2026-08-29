# Next deployment and platform milestones

## Completed baseline

- application workloads + services
- Vault Agent runtime injection
- Harbor immutable images
- Jenkins GitOps image updates
- Argo CD automated sync
- `/livez` `/readyz` production-like probes
- graceful shutdown contract
- standalone versioned DB migrator
- Argo CD PreSync migration hook
- fail-closed checksum/dirty protection
- dedicated `tropical_chat` DB ownership
- E2E automated migration-before-rollout validation

## P0.2.1 Local-development parity

Owned primarily by the application repo, but platform docs should track it:

- fresh Compose bootstrap through standalone migrator;
- local chat DSN -> `tropical_chat`;
- no reintroduction of runtime startup DDL.

## P0.3 Resources and availability

Add measured requests/limits first, then availability policy:

- CPU/memory requests/limits per workload;
- replica decisions based on state/scaling characteristics;
- rolling update settings;
- PDB for eligible multi-replica workloads;
- topology spread/anti-affinity where useful;
- HPA only after resource requests and metrics are reliable.

Keep `tropical-chat` single replica until shared pub/sub exists.

## P0.4 Security hardening

- split DB runtime identities from migration identity;
- least-privilege grants;
- securityContext/non-root/read-only filesystem where compatible;
- NetworkPolicy ingress/egress;
- service-account/RBAC review;
- registry/dependency image scanning policy;
- secret rotation procedure.

## P1 Observability

ELK infrastructure exists but application integration is not yet considered complete.

Target:

- centralized structured logs;
- request IDs preserved end-to-end;
- Kubernetes/container metadata enrichment;
- Prometheus/Grafana service + DB metrics;
- alerts for error rate, readiness, restart storms, migration failure, DB pressure;
- OpenTelemetry traces where useful;
- dashboard/runbook links.

## P1 Data and edge resilience

- define backup/RPO/RTO and run restore drills;
- decide whether MySQL single-host SPOF is acceptable for the homelab target;
- test DB outage/disk pressure;
- ingress/TLS/certificate lifecycle;
- external exposure/firewall policy.

## Product sequencing rule

Prioritize reliability/security work that protects existing product workflows before adding infrastructure-heavy features. New product work should include acceptance criteria, service/data owner, migration impact, observability impact, rollback strategy, and operational cost.
