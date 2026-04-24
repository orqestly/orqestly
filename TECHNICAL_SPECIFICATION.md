# Technical Specification - Orqestly

## Purpose

This document bridges product requirements and architecture by defining concrete lifecycle contracts, approval controls, status/state transitions, memory behavior, integration mappings, and skill interfaces.

## Lifecycle Contract

Orqestly runs as a cycle, not a one-time command:

1. Intake
2. Parse and Normalize
3. Plan
4. Execute
5. Review and Approval
6. Decision (Rerun or Close)
7. Reporting

Each stage must define entry conditions, exit conditions, and valid next transitions.

## Status and Transition Model

### Core Statuses

- `pending`
- `in_progress`
- `blocked`
- `needs_review`
- `failed`
- `done`

### Transition Rules

- `pending` -> `in_progress` or `blocked`
- `in_progress` -> `needs_review`, `failed`, or `done`
- `needs_review` -> `in_progress`, `failed`, or `done`
- `blocked` -> `pending` or `in_progress`
- `failed` -> `in_progress` (rerun) or terminal close
- `done` -> terminal close

Invalid transitions must be rejected and logged.

### Required State Fields

- `entity_id`
- `entity_type` (orchestration, scenario, step)
- `status`
- `previous_status`
- `changed_by`
- `changed_at`
- `reason`
- `run_id`

## Human-in-the-Loop Approval Contract

### Approval Gates

- Pre-execution approval
- Ambiguity resolution approval
- Defect triage approval
- Closure approval

### Approval Actions

- `approve`
- `reject`
- `request_changes`
- `defer`

### Required Approval Fields

- `approval_id`
- `gate_type`
- `subject_id`
- `requested_by`
- `approver`
- `decision`
- `decision_reason`
- `requested_at`
- `resolved_at`
- `timeout_at`

Timeout behavior must be configurable per gate.

## Memory Model

### Run Memory

Stores context for the active run:

- normalized requirements
- plan snapshots
- step outcomes
- pending decisions

### Cross-Run Memory

Stores reusable operational history:

- previous approvals and rationales
- known flaky patterns
- defect-classification corrections
- validated scenario templates

### Retention Boundaries

- Memory retention must be configurable.
- Sensitive evidence must follow security policy boundaries.
- Purge and archival operations must be auditable.

## Integration Contract (Jira-Style)

### Minimum Mapping

- internal defect ID -> external issue key
- defect severity -> issue priority
- defect title -> issue summary
- defect details/evidence -> issue description and attachments
- internal status -> external workflow state

### Sync Behavior

- Create issue on approved defect publication.
- Update issue when orchestration status changes.
- Reconcile and log sync conflicts.
- Preserve a full sync audit trail.

## Skill Framework Contract

### Skill Interface

Each skill must provide:

- `skill_name`
- `version`
- `input_schema`
- `output_schema`
- `evidence_schema`
- `error_schema`

### Runtime Behavior

- Orchestrator selects a skill per planned step.
- Skill output must be normalized before reporting.
- Every skill call must emit evidence and execution metadata.

## Baseline Skill: http_request

### Purpose

Validate API behavior through deterministic HTTP request execution.

### Input Contract

- `method`
- `url`
- `headers`
- `query`
- `body`
- `expected_status`
- `expected_assertions`
- `timeout_ms`

### Output Contract

- `actual_status`
- `response_headers`
- `response_body`
- `assertion_results`
- `duration_ms`
- `outcome_status`

### Evidence Contract

- serialized request and response
- assertion evaluation details
- timestamps and execution identifier

### Failure Modes

- network failure
- timeout
- assertion mismatch
- malformed response
- authentication/authorization failure

## Traceability Requirements

- Every plan step must map to one or more requirements.
- Every skill execution must map to a plan step.
- Every defect must map to source requirements and evidence artifacts.
- Every external issue must map back to the originating defect and run.

## Acceptance Criteria

- Lifecycle stages and transitions are enforceable and auditable.
- Human approvals are captured with full decision metadata.
- Status model answers done/next/pending/blocked questions reliably.
- Memory behavior supports reruns without leaking sensitive data.
- Jira-style integration succeeds with deterministic mapping.
- `http_request` skill produces consistent, triage-ready outputs.

## Related Documents

- [SYSTEM_REQUIREMENTS.md](SYSTEM_REQUIREMENTS.md)
- [ARCHITECTURE.md](ARCHITECTURE.md)
- [SUCCESS_METRICS.md](SUCCESS_METRICS.md)
- [SECURITY.md](SECURITY.md)
