# Metrics Collection Specification - Orqestly

## Purpose

This document defines how Orqestly collects, calculates, and reviews success metrics for Phase 1.

## Metric Families

- requirement traceability
- execution reliability
- defect report quality
- human-in-the-loop governance
- lifecycle status integrity
- integration effectiveness
- memory and reuse quality
- skill runtime quality

## Collection Sources

- orchestration run records
- approval event history
- status transition history
- defect and Jira sync logs
- skill execution outputs
- evidence bundles

## Collection Frequency

- run-level metrics: collected per orchestration run
- daily operational metrics: summarized once per day
- release metrics: reviewed at each milestone gate
- trend metrics: reviewed weekly

## Metric Definitions

### Traceability Coverage

- Ratio of validation steps linked to requirements.

### Approval Latency

- Time from approval request to decision.

### Status Validity

- Ratio of valid status transitions to total transitions.

### Jira Sync Success

- Ratio of successful issue sync operations to total sync attempts.

### Evidence Completeness

- Ratio of runs with complete required evidence to total runs.

### Skill Output Success

- Ratio of successful skill executions to total skill executions.

## Aggregation Rules

- Metrics must be computed from immutable source records.
- Manual edits to metric history are not allowed.
- Any metric correction must preserve original raw data and a correction note.

## Reporting Rules

- Report metrics in a milestone review summary.
- Flag metrics that miss target thresholds.
- Include trends, not only point-in-time values.

## Ownership

- Orchestrator owner: runtime metrics
- QA lead: evidence quality and defect metrics
- Integration owner: sync and external system metrics
- Product owner: milestone-level reporting

## Related Documents

- [SUCCESS_METRICS.md](SUCCESS_METRICS.md)
- [SLO_AND_ALERTING.md](SLO_AND_ALERTING.md)
- [PHASE1_EXECUTION_PLAN.md](PHASE1_EXECUTION_PLAN.md)
