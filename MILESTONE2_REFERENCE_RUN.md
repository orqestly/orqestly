# Milestone 2 Reference Run - Orqestly

## Purpose

This document defines a single canonical end-to-end run used to validate Milestone 2 readiness.

## Reference Scenario

Validate a simple form-based onboarding flow with one API verification step:

- UI flow: open form, enter required fields, submit, confirm success state.
- API check: validate one post-submit endpoint with `http_request` skill.

## Inputs

- Requirements artifact (Markdown or JSON).
- Optional UI/UX spec excerpt for form constraints.
- API endpoint contract for post-submit verification.

## Expected Lifecycle Path

1. `pending` (run created)
2. `in_progress` (planning and execution started)
3. `needs_review` (human approval gate for triage)
4. `done` (approved and closed)

If runtime errors occur, path may include `failed` and rerun transition back to `in_progress`.

## Expected Approval Events

- Pre-execution approval: decision recorded before execution starts.
- Defect triage approval: decision recorded before publication.
- Closure approval: decision recorded before final close.

All approval events must include actor, timestamp, decision, and reason.

## Expected Jira Sync Events

- Issue created for approved defects.
- Issue status updated when orchestration status changes.
- Sync log entry persisted for every create/update action.

## Required Evidence

- UI execution evidence (step logs and screenshots).
- API skill evidence (request/response payload and assertions).
- Status transition history for orchestration and step entities.
- Approval event history for all triggered gates.
- Defect report linked to requirements and evidence.

## Pass Criteria

- End-to-end run completes with valid lifecycle transitions only.
- Required approvals are captured and auditable.
- Evidence bundle is complete and triage-ready.
- At least one API verification executes through `http_request` skill.
- Defect publication and Jira sync behavior are deterministic.

## Fail Criteria

- Any invalid status transition occurs.
- Any mandatory approval gate is bypassed.
- Any required evidence artifact is missing.
- Jira sync is missing required mapping fields.

## Execution Checklist

- Confirm input artifacts are valid.
- Execute orchestration run for the reference scenario.
- Verify transitions against [STATUS_TRANSITION_TABLE.md](STATUS_TRANSITION_TABLE.md).
- Verify approvals against [APPROVAL_GATE_MATRIX.md](APPROVAL_GATE_MATRIX.md).
- Verify Jira mappings against [JIRA_FIELD_MAPPING.md](JIRA_FIELD_MAPPING.md).
- Confirm pass/fail against criteria in this document.

## Related Documents

- [PHASE1_EXECUTION_PLAN.md](PHASE1_EXECUTION_PLAN.md)
- [TECHNICAL_SPECIFICATION.md](TECHNICAL_SPECIFICATION.md)
- [STATUS_TRANSITION_TABLE.md](STATUS_TRANSITION_TABLE.md)
- [APPROVAL_GATE_MATRIX.md](APPROVAL_GATE_MATRIX.md)
- [JIRA_FIELD_MAPPING.md](JIRA_FIELD_MAPPING.md)
