# Tropical Management Phase 3 Development Notes

## Current Architecture

The application is split into:

- Application repository: `tropical-management-v1`
- GitOps repository: `tropical-management-gitops`

Delivery flow:

Developer -> Git commit -> Jenkins -> Docker image -> Harbor -> GitOps image tag update -> Argo CD reconciliation -> Kubernetes.

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

## Local Development

Start services:

```bash
docker compose up -d
```

Validate:

```bash
docker compose ps
```

Open:

```
http://localhost:3000/chat
```

## Future Production Checklist

- Add Vault Agent secret file injection
- Add Kubernetes Deployment for chat-service
- Add Service and Ingress routing
- Enable Argo CD automated synchronization after validation
- Add HPA, PDB, and NetworkPolicy

## Development Rule

Every new service must include:

- Dockerfile
- health endpoint
- environment documentation
- Kubernetes manifest
- GitOps overlay update
- README update
