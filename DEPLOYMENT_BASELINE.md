# Deployment Baseline - Orqestly

## Purpose

This document defines the Phase 1 deployment assumptions for container build, runtime orchestration, and service connectivity on the target VPS.

## Deployment Model

- Every Orqestly component is built as a Docker image by GitHub CI/CD.
- Deployment runtime uses `docker-compose.yaml` for service wiring and startup.
- Existing VPS infra services are reused instead of duplicated.

## Existing VPS Infrastructure

- `infra-redis` (Redis)
- `infra-postgres` (PostgreSQL)
- `edge-nginx` (main reverse proxy)

These containers are treated as external platform dependencies.

## Service Topology

- `orqestly-api`: public/internal API service for orchestration control.
- `orqestly-worker`: background worker for lifecycle execution and skills.

Both services should use:

- Redis at `infra-redis:6379`
- PostgreSQL at `infra-postgres:5432`

## Networking Rules

- Orqestly services must join a shared Docker network that can reach infra containers.
- Orqestly API must join the edge proxy network so `edge-nginx` can route traffic.
- Internal worker services do not require direct public exposure.

## Reverse Proxy Routing

- `edge-nginx` is the single external entry point.
- Route incoming traffic to `orqestly-api` container on its internal service port.
- Health endpoints must be exposed for proxy and deployment checks.

## Configuration Baseline

Minimum required environment variables:

- `DATABASE_URL`
- `REDIS_URL`
- `ORQESTLY_ENV`
- `LOG_LEVEL`
- `JIRA_BASE_URL`
- `JIRA_PROJECT_KEY`

Secrets should be injected through environment management, not committed to repository.

## CI/CD Baseline

- Build and tag one image per component.
- Push images to registry (for example GHCR).
- Deploy by updating image tags consumed by `docker-compose.yaml`.
- Run post-deploy health checks and rollback on failed readiness checks.

## Operational Checks

- API container can connect to `infra-postgres`.
- API and worker can connect to `infra-redis`.
- `edge-nginx` route returns healthy API response.
- Worker can process at least one reference orchestration task.

## Related Documents

- [TECH_STACK_FREEZE.md](TECH_STACK_FREEZE.md)
- [PHASE1_EXECUTION_PLAN.md](PHASE1_EXECUTION_PLAN.md)
- [PHASE1_ROLLOUT_CHECKLIST.md](PHASE1_ROLLOUT_CHECKLIST.md)
- [MILESTONE2_REFERENCE_RUN.md](MILESTONE2_REFERENCE_RUN.md)
