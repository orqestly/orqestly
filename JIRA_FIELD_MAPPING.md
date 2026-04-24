# Jira Field Mapping - Orqestly

## Purpose

This document defines deterministic mapping between Orqestly defect/status data and Jira issue fields.

## Defect to Issue Mapping

| Orqestly Field | Jira Field | Mapping Rule |
|---|---|---|
| `defect_id` | `Issue Key` (linked reference) | Stored after issue creation |
| `defect_title` | `Summary` | Direct copy |
| `defect_description` | `Description` | Direct copy with traceability links |
| `severity` | `Priority` | Map by severity table |
| `requirement_refs` | `Description` or custom traceability field | Include all requirement IDs |
| `evidence_links` | `Attachments` and description links | Upload artifacts and reference URLs |
| `owner` | `Assignee` | Default by ownership rule |
| `status` | `Status` | Map via lifecycle status mapping |

## Severity Mapping

| Orqestly Severity | Jira Priority |
|---|---|
| `critical` | Highest |
| `high` | High |
| `medium` | Medium |
| `low` | Low |

## Status Mapping

| Orqestly Status | Jira Status (example workflow) |
|---|---|
| `pending` | To Do |
| `in_progress` | In Progress |
| `blocked` | Blocked |
| `needs_review` | In Review |
| `failed` | In Progress (with failure label) |
| `done` | Done |

## Sync Rules

- Create Jira issue after defect triage approval.
- Update Jira status on every valid lifecycle status change.
- Reject sync when required fields are missing.
- Log all sync attempts, payloads, responses, and reconciliation actions.

## Conflict Handling

- If external status conflicts with internal status, open reconciliation task.
- Preserve Orqestly state as source of truth until reconciliation is approved.
- Record final decision and update both systems.

## Related Documents

- [TECHNICAL_SPECIFICATION.md](TECHNICAL_SPECIFICATION.md)
- [SYSTEM_REQUIREMENTS.md](SYSTEM_REQUIREMENTS.md)
- [PHASE1_EXECUTION_PLAN.md](PHASE1_EXECUTION_PLAN.md)
