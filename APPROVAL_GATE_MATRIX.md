# Approval Gate Matrix - Orqestly

## Purpose

This document defines human-in-the-loop approval gates, decision actions, timeout behavior, and fallback outcomes.

## Approval Actions

- `approve`
- `reject`
- `request_changes`
- `defer`

## Gate Definitions

| Gate | Trigger | Required Approver | Allowed Actions | Timeout Behavior | Fallback |
|---|---|---|---|---|---|
| Pre-execution | Plan generated and ready to run | Product owner or QA lead | Approve, Reject, Request Changes, Defer | Move to `needs_review` alert state after timeout | Block run until decision |
| Ambiguity resolution | Parser marks ambiguous requirement | Product owner or domain reviewer | Approve, Reject, Request Changes, Defer | Escalate to owner group | Keep affected scope `blocked` |
| Defect triage | Defect bundle ready for publication | QA lead | Approve, Reject, Request Changes, Defer | Escalate to backup approver | Keep defect in `needs_review` |
| Closure | Run marked ready to close | Product owner or QA lead | Approve, Reject, Request Changes, Defer | Escalate and hold closure | Keep run `needs_review` |

## Required Approval Metadata

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

## Operating Rules

- All approval events must be logged with actor and timestamp.
- Rejected approvals must include reason and required follow-up.
- Deferred decisions must include next-review timestamp.
- Timeout escalation must be deterministic and auditable.

## Related Documents

- [TECHNICAL_SPECIFICATION.md](TECHNICAL_SPECIFICATION.md)
- [SYSTEM_REQUIREMENTS.md](SYSTEM_REQUIREMENTS.md)
- [PHASE1_EXECUTION_PLAN.md](PHASE1_EXECUTION_PLAN.md)
