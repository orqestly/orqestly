# SLO and Alerting Baseline - Orqestly

## Purpose

This document defines the minimum service-level objectives and alerting thresholds for Phase 1 operations.

## SLO Targets

### API Availability

- Target: 99.5% monthly availability for the API service.
- Measurement: successful health check response over time.

### Worker Availability

- Target: 99.5% monthly availability for the worker service.
- Measurement: worker process health and queue consumption capability.

### Approval Latency

- Target: 90% of approvals resolved within 4 business hours.
- Measurement: approval request to decision timestamp.

### Jira Sync Success

- Target: 95% of sync attempts succeed without manual intervention.
- Measurement: successful issue create/update events divided by total sync attempts.

### Reference Run Success

- Target: 100% success for the canonical reference run after Milestone 2.
- Measurement: run outcome against [MILESTONE2_REFERENCE_RUN.md](MILESTONE2_REFERENCE_RUN.md).

## Alert Thresholds

### Critical

- API health check fails for 2 consecutive probes.
- Worker is unable to process tasks for 10 minutes.
- Approval gate exceeds SLA by more than 2x.
- Jira sync failure rate exceeds 10% in a rolling 30-minute window.

### Warning

- Queue depth is growing for 15 minutes.
- Approval backlog is aging beyond expected window.
- Reference run produces flaky or retry-heavy outcomes.

## Alert Routing

- Operational alerts go to Orchestrator Owner and QA Lead.
- Integration alerts go to Integration Owner.
- Skill failures go to Skill Owner.
- Release-blocking issues go to Product Owner.

## Dashboard Signals

- API health
- worker health
- Redis connectivity
- Postgres connectivity
- approval backlog age
- queue depth
- Jira sync success rate
- status transition failures
- failed skill executions

## Escalation Policy

1. Triage within the owning role.
2. Escalate to secondary owner if unresolved.
3. Escalate to Product Owner if the issue blocks release or governance.

## Related Documents

- [DEPLOYMENT_BASELINE.md](DEPLOYMENT_BASELINE.md)
- [PHASE1_ROLLOUT_CHECKLIST.md](PHASE1_ROLLOUT_CHECKLIST.md)
- [OPERATIONS_RUNBOOK.md](OPERATIONS_RUNBOOK.md)
- [OBSERVABILITY_DASHBOARD_SPEC.md](OBSERVABILITY_DASHBOARD_SPEC.md)
