# Capacity Plan - Phase 1

## Purpose

This document defines the initial capacity assumptions for Orqestly Phase 1 so the platform can run predictably on the target VPS infrastructure.

## Capacity Assumptions

- Single VPS deployment environment.
- Shared infra services: `infra-postgres` and `infra-redis`.
- One public Orqestly API service behind `edge-nginx`.
- One worker service for background orchestration and skill execution.

## Initial Load Profile

- Low-concurrency internal usage.
- Small number of parallel orchestration runs.
- Form-based UI validation as the primary workload.
- Baseline API validation through `http_request` skill.

## Resource Targets

### API Service

- Memory: start with 512 MB to 1 GB container budget.
- CPU: baseline 0.5 to 1 vCPU equivalent allocation.

### Worker Service

- Memory: start with 1 GB container budget.
- CPU: baseline 1 vCPU equivalent allocation.

### Database And Cache

- PostgreSQL and Redis are managed by shared infra containers.
- Capacity assumptions should remain aligned with their existing VPS limits.

## Queue And Concurrency Assumptions

- Start with one worker process and conservative task concurrency.
- Increase concurrency only after reference runs and governance tests are stable.
- Keep approval backlog and queue depth within SLO thresholds.

## Scale Triggers

- Queue depth grows beyond warning thresholds.
- Approval backlog becomes persistent.
- Reference runs begin to fail due to resource pressure.
- Jira sync or API skill execution slows due to saturation.

## Expansion Policy

- Add worker replicas before expanding feature scope.
- Tune concurrency only after observing real run data.
- Revisit capacity after Milestone 3 and after first operational release.

## Related Documents

- [DEPLOYMENT_BASELINE.md](DEPLOYMENT_BASELINE.md)
- [SLO_AND_ALERTING.md](SLO_AND_ALERTING.md)
- [PHASE1_EXECUTION_PLAN.md](PHASE1_EXECUTION_PLAN.md)
