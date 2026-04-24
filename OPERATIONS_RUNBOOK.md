# Operations Runbook - Orqestly

## Purpose

This runbook describes how to operate, troubleshoot, and recover the Phase 1 Orqestly deployment running in Docker Compose against the existing VPS infrastructure.

## Operating Model

- Orqestly services are deployed as Docker images and wired through `docker-compose.yaml`.
- `edge-nginx` is the public entry point.
- `infra-postgres` and `infra-redis` are shared platform dependencies.

## Service Checks

### API Service

- Verify container health endpoint returns success.
- Verify API can connect to `infra-postgres`.
- Verify API can connect to `infra-redis`.
- Verify `edge-nginx` routes traffic to API service.

### Worker Service

- Verify worker container is running and attached to the compose network.
- Verify worker can consume tasks from Redis.
- Verify worker can persist orchestration updates to PostgreSQL.

## Common Failures And Recovery

### 1. API Container Fails Health Check

Possible causes:

- missing environment variable
- database unavailable
- image mismatch

Recovery steps:

1. inspect container logs
2. validate `DATABASE_URL` and `REDIS_URL`
3. confirm infra containers are healthy
4. redeploy latest known-good image if needed

### 2. Worker Stuck Or Not Processing Tasks

Possible causes:

- Redis connectivity issue
- task queue backlog
- invalid skill payload

Recovery steps:

1. check worker logs and Redis connectivity
2. verify queued task payloads
3. retry stuck jobs after fixing the root cause
4. record incident in the run log

### 3. Approval Gate Stuck

Possible causes:

- approver unavailable
- timeout not configured
- notification failure

Recovery steps:

1. confirm gate type and approver assignment
2. apply escalation policy from `APPROVAL_GATE_MATRIX.md`
3. record decision or fallback action
4. resume orchestration only after the gate is resolved

### 4. Jira Sync Fails

Possible causes:

- invalid Jira credentials
- field mapping mismatch
- network failure

Recovery steps:

1. verify Jira credentials and project key
2. compare payload against `JIRA_FIELD_MAPPING.md`
3. retry sync after correcting mapping or credentials
4. preserve sync failure details in evidence

## Incident Response Flow

1. Identify affected service.
2. Determine whether the failure is infrastructure, application, or integration related.
3. Apply the recovery steps for the failure class.
4. Document the incident, root cause, and resolution.
5. Update related runbooks or checklists if the issue repeats.

## Restart Order

1. Confirm infra containers are healthy.
2. Restart worker service.
3. Restart API service.
4. Validate edge proxy routing.
5. Run a reference health check.

## Evidence To Capture

- container logs
- health check outputs
- queue state snapshots
- approval event records
- sync failure payloads

## Related Documents

- [DEPLOYMENT_BASELINE.md](DEPLOYMENT_BASELINE.md)
- [PHASE1_ROLLOUT_CHECKLIST.md](PHASE1_ROLLOUT_CHECKLIST.md)
- [MILESTONE2_REFERENCE_RUN.md](MILESTONE2_REFERENCE_RUN.md)
- [RUNBOOK_CHECKLISTS.md](RUNBOOK_CHECKLISTS.md)
