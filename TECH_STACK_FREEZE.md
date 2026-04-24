# Tech Stack Freeze - Orqestly

## Purpose

This document freezes the recommended implementation stack for Phase 1 so engineering execution can move without repeated tooling churn.

## Freeze Scope

- Runtime and language choices
- Core orchestration and API framework
- UI automation and API validation tooling
- Data and persistence layer
- Queueing and async execution
- Reporting and observability
- CI/CD and deployment baseline

## Frozen Stack (Phase 1)

### Runtime and Language

- Python 3.12
- Type hints required for core modules

### Service and API Layer

- FastAPI for orchestration service APIs
- Pydantic v2 for schemas and validation
- Uvicorn as ASGI runtime

### Orchestration and Worker Execution

- Celery for async task orchestration
- Redis for broker and short-lived orchestration state

### UI and API Validation Skills

- Playwright (Python) for UI automation
- HTTPX for `http_request` skill execution

### Persistence

- PostgreSQL for durable metadata, run history, approvals, and mappings
- SQLAlchemy 2.x and Alembic for ORM and migrations

### Reporting and Evidence

- JSON artifact bundles for reports and evidence
- Optional CSV defect export for external triage workflows

### Observability

- Structured JSON logging
- OpenTelemetry instrumentation
- Prometheus-compatible metrics exposure

### Integrations

- Jira Cloud REST API integration for issue sync

### CI/CD

- GitHub Actions for lint, test, package, and deployment pipelines

### Containerization and Runtime Topology

- Every component is built into a Docker image through GitHub CI/CD.
- Runtime composition is managed through `docker-compose.yaml`.
- Existing VPS infrastructure containers (`infra-redis`, `infra-postgres`) are reused.
- `edge-nginx` is the primary reverse proxy and routes traffic to Orqestly API services.

## Versioning Policy

- Pin major versions for all runtime-critical dependencies.
- Lock dependency graph with a reproducible lock file.
- Introduce upgrades only through scheduled stack review windows.

## Dependency Manifest Source Of Truth

- Python dependency source of truth: [pyproject.toml](pyproject.toml)
- Runtime services install from `project.dependencies`.
- Engineering tooling installs from `project.optional-dependencies.dev`.
- Lock file generation should be derived from this manifest only.

## Decision Rationale

- FastAPI and Pydantic align with schema-first contracts in lifecycle/status/approval models.
- Celery + Redis provides practical, proven async orchestration for Phase 1 complexity.
- PostgreSQL supports auditability, transition history, and integration mappings reliably.
- Playwright and HTTPX cover both UI and API skill baselines with clear Python integration.

## Freeze Boundaries

- No framework replacement during Phase 1 except for critical security or stability reasons.
- New tools must integrate with frozen contracts before adoption.
- Any exception requires product owner and orchestrator owner approval.

## Review Window

- First stack review after Milestone 3 completion.
- Emergency review allowed only for production-blocking issues.

## Related Documents

- [SYSTEM_REQUIREMENTS.md](SYSTEM_REQUIREMENTS.md)
- [TECHNICAL_SPECIFICATION.md](TECHNICAL_SPECIFICATION.md)
- [PHASE1_EXECUTION_PLAN.md](PHASE1_EXECUTION_PLAN.md)
- [PHASE1_ROLLOUT_CHECKLIST.md](PHASE1_ROLLOUT_CHECKLIST.md)
- [DEPLOYMENT_BASELINE.md](DEPLOYMENT_BASELINE.md)
- [docker-compose.yaml](docker-compose.yaml)
