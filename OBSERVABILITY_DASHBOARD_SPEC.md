# Observability Dashboard Specification - Orqestly

## Purpose

This document defines the dashboard panels, alerts, and operational views needed to monitor the Phase 1 Orqestly deployment.

## Dashboard Objectives

- make service health visible at a glance
- expose orchestration bottlenecks early
- track approval backlog and sync failures
- show skill execution reliability
- support release-readiness checks

## Core Dashboards

### 1. Service Health Dashboard

Panels:

- API health status
- worker health status
- Redis connectivity
- Postgres connectivity
- edge proxy routing status

### 2. Orchestration Flow Dashboard

Panels:

- active runs
- runs by status
- step transition failures
- approval backlog age
- blocked run count

### 3. Integration Dashboard

Panels:

- Jira sync success rate
- Jira sync failure count
- latest sync error reasons
- unresolved sync conflicts

### 4. Skill Execution Dashboard

Panels:

- `http_request` success rate
- `http_request` timeout count
- skill execution duration
- skill failure classification

### 5. Release Readiness Dashboard

Panels:

- reference run success
- evidence bundle completeness
- approval latency
- open critical alerts

## Alert Definitions

### Critical Alerts

- API unavailable twice in succession
- worker unavailable for 10 minutes
- approval SLA breached by 2x
- Jira sync failure rate above threshold
- reference run fails

### Warning Alerts

- queue depth trending upward
- approval backlog growing
- repeated skill failures on same run type

## Data Sources

- orchestration run records
- approval events
- status transition logs
- Jira sync logs
- skill execution outputs
- health check events

## Refresh Cadence

- live panels: near real-time
- summary panels: every 5 to 15 minutes
- release readiness: at each milestone gate

## Ownership

- Orchestrator owner: service and orchestration panels
- Integration owner: sync panels and alerts
- QA lead: skill and evidence panels
- Product owner: release readiness view

## Related Documents

- [SLO_AND_ALERTING.md](SLO_AND_ALERTING.md)
- [OPERATIONS_RUNBOOK.md](OPERATIONS_RUNBOOK.md)
- [METRICS_COLLECTION_SPEC.md](METRICS_COLLECTION_SPEC.md)
