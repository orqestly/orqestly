# Milestone 3 Governance Test Pack - Orqestly

## Purpose

This document defines the governance-focused test cases for Milestone 3, validating approval behavior, timeout handling, escalation paths, and skill output integrity.

## Test Objectives

- Verify human-in-the-loop approval gates behave consistently.
- Verify timeout and fallback policies trigger correctly.
- Verify approval metadata is complete and auditable.
- Verify `http_request` skill outputs remain normalized and traceable.
- Verify governance failures are surfaced clearly in reports.

## Approval Test Cases

### Case 1: Pre-Execution Approval Granted

- Given a completed plan, when pre-execution approval is requested, the approver selects `approve`.
- Then the orchestration run transitions to execution without invalid state jumps.

### Case 2: Ambiguity Resolution Requires Changes

- Given a parser ambiguity flag, when the reviewer selects `request_changes`.
- Then the affected scope remains blocked until the revised input is accepted.

### Case 3: Defect Triage Rejected

- Given a defect bundle ready for publication, when the QA lead selects `reject`.
- Then publication is stopped and the rejection reason is stored.

### Case 4: Closure Deferred

- Given a completed run, when closure is deferred.
- Then the run remains in a review state and the next-review timestamp is recorded.

## Timeout And Escalation Test Cases

### Case 5: Approval Timeout

- Given a pending approval gate, when no decision arrives before timeout.
- Then the system moves to the configured fallback state and records escalation metadata.

### Case 6: Backup Approver Escalation

- Given a timed-out triage approval gate, when escalation is triggered.
- Then the backup approver is notified and the approval history remains intact.

### Case 7: Stale Decision Rejection

- Given an approval decision submitted after timeout.
- Then the decision is rejected or marked stale according to policy.

## Skill Output Validation Cases

### Case 8: HTTP Request Success

- Given a valid API request, when `http_request` executes successfully.
- Then the output includes status, payload, assertion results, duration, and evidence.

### Case 9: HTTP Request Assertion Failure

- Given an API response that does not match expected assertions.
- Then the skill marks the result as failed and preserves request/response evidence.

### Case 10: HTTP Request Timeout

- Given a slow API endpoint, when execution exceeds timeout.
- Then the skill returns a timeout failure with deterministic evidence.

## Governance Data Checks

- Every approval event must contain actor, timestamp, decision, and reason.
- Every timeout event must contain gate type, fallback action, and escalation target.
- Every skill execution must map to a run, step, and requirement reference.
- Every governance failure must appear in the final report.

## Pass Criteria

- All approval cases produce the expected status outcome.
- All timeout cases follow configured fallback behavior.
- All escalation cases preserve decision history.
- All skill validation cases produce normalized outputs.
- Reports contain traceable evidence for every failure.

## Related Documents

- [PHASE1_EXECUTION_PLAN.md](PHASE1_EXECUTION_PLAN.md)
- [APPROVAL_GATE_MATRIX.md](APPROVAL_GATE_MATRIX.md)
- [TECHNICAL_SPECIFICATION.md](TECHNICAL_SPECIFICATION.md)
- [MILESTONE2_REFERENCE_RUN.md](MILESTONE2_REFERENCE_RUN.md)
