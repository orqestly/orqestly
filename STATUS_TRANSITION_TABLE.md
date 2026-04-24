# Status Transition Table - Orqestly

## Purpose

This document defines the allowed lifecycle status transitions for orchestration runs, scenarios, and execution steps.

## Statuses

- `pending`
- `in_progress`
- `blocked`
- `needs_review`
- `failed`
- `done`

## Allowed Transitions

| From | To | Trigger | Notes |
|---|---|---|---|
| `pending` | `in_progress` | Work starts | Normal start path |
| `pending` | `blocked` | Dependency missing | Requires unblock reason |
| `in_progress` | `needs_review` | Human decision needed | Opens approval gate |
| `in_progress` | `failed` | Hard failure | Include failure class |
| `in_progress` | `done` | Success criteria met | Evidence must exist |
| `blocked` | `pending` | Blocker removed | Returns to queue |
| `blocked` | `in_progress` | Immediate restart | Use when owner resumes directly |
| `needs_review` | `in_progress` | Changes requested | Continue after updates |
| `needs_review` | `failed` | Review rejects outcome | Capture decision reason |
| `needs_review` | `done` | Review approves outcome | Capture approver metadata |
| `failed` | `in_progress` | Rerun approved | Requires rerun reason |

## Forbidden Transitions

- Any direct transition not listed in Allowed Transitions.
- Any transition that skips required approval when status is `needs_review`.
- Any closure action without evidence and traceability metadata.

## Required Transition Metadata

- `entity_id`
- `entity_type` (`orchestration`, `scenario`, `step`)
- `from_status`
- `to_status`
- `changed_by`
- `changed_at`
- `reason`
- `run_id`
- `traceability_refs`

## Validation Rules

- Transitions must be validated before persistence.
- Invalid transitions must be rejected and logged.
- Transition history must be immutable and auditable.

## Related Documents

- [TECHNICAL_SPECIFICATION.md](TECHNICAL_SPECIFICATION.md)
- [SYSTEM_REQUIREMENTS.md](SYSTEM_REQUIREMENTS.md)
- [PHASE1_EXECUTION_PLAN.md](PHASE1_EXECUTION_PLAN.md)
