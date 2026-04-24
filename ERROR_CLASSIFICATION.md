# Error Classification - Orqestly

## Purpose

This document defines a stable taxonomy for classifying failures during orchestration, skill execution, integration sync, and deployment operations.

## Failure Classes

### 1. Transient Infrastructure Failure

- Redis unavailable
- Postgres unavailable
- network interruption
- container restart during run

Recovery: retry, restore service, rerun step or run when safe.

### 2. Transient Integration Failure

- Jira timeout
- temporary auth failure
- rate limit exceeded

Recovery: retry with backoff, revalidate credentials, sync again.

### 3. Product Defect

- actual application behavior differs from requirements
- stable replayable failure in UI or API validation

Recovery: publish defect with evidence and traceability links.

### 4. Environment Mismatch

- test data missing
- feature flag not enabled
- dependency URL incorrect

Recovery: mark blocked, fix environment, rerun.

### 5. Skill Execution Failure

- malformed skill input
- assertion engine error
- timeout in `http_request`

Recovery: correct input or skill config, rerun skill.

### 6. Governance Failure

- approval timeout
- rejected approval
- missing approver

Recovery: escalate or request changes according to gate policy.

## Classification Rules

- If the failure disappears on immediate retry, classify as transient.
- If the failure reproduces on a stable environment, classify as product defect or environment mismatch.
- If no approval was captured, classify as governance failure.
- If a skill returns invalid output, classify as skill execution failure.

## Required Metadata

- `failure_id`
- `run_id`
- `entity_id`
- `class`
- `severity`
- `detected_at`
- `detected_by`
- `evidence_refs`
- `recommended_action`

## Severity Guidance

- critical: blocks orchestration or release
- high: breaks core flow or governance
- medium: affects limited paths or causes rerun
- low: informational or non-blocking issue

## Related Documents

- [TECHNICAL_SPECIFICATION.md](TECHNICAL_SPECIFICATION.md)
- [SLO_AND_ALERTING.md](SLO_AND_ALERTING.md)
- [OPERATIONS_RUNBOOK.md](OPERATIONS_RUNBOOK.md)
