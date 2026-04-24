# Evidence Retention Policy - Orqestly

## Purpose

This document defines how Orqestly captures, stores, retains, and purges evidence artifacts generated during orchestration runs.

## Evidence Categories

- UI screenshots
- API request and response payloads
- structured logs
- approval events
- orchestration state snapshots
- sync audit trails

## Retention Classes

### Run-Scope Evidence

- Stored for the duration of the active run plus a short diagnostic window.
- Used for immediate debugging and release validation.

### Cross-Run Evidence

- Stored only when needed for pattern reuse, repeat defect analysis, or audit purposes.
- Access is limited to authorized roles.

### Defect Evidence

- Stored with the defect record and linked to traceability references.
- Kept until the associated defect is closed and the retention window expires.

## Retention Defaults

- Run-scope evidence: 30 days
- Cross-run decision history: 90 days
- Defect evidence: 180 days
- Released audit bundle: 1 year

These defaults may be overridden by compliance or customer policy.

## Storage Rules

- Evidence must be stored in a structured, versioned format.
- Sensitive fields must be masked before persistence when possible.
- Evidence bundles must include metadata for run ID, entity ID, and capture timestamp.

## Purge Rules

- Purge expired evidence automatically on a scheduled job.
- Preserve legal or compliance holds until released.
- Log every purge action with timestamp and actor/service identity.

## Access Control

- Product owner: read summary evidence
- QA lead: read full evidence bundles
- Orchestrator owner: read operational evidence and logs
- Integration owner: read sync-related evidence
- Skill owner: read skill-specific evidence

## Redaction Rules

- Mask secrets, tokens, and credentials.
- Redact PII when not required for defect reproduction.
- Prefer hash or reference token over raw secret values.

## Audit Requirements

- Every evidence write, read, export, and purge must be logged.
- Audit logs must preserve who accessed evidence and why.

## Related Documents

- [TECHNICAL_SPECIFICATION.md](TECHNICAL_SPECIFICATION.md)
- [OPERATIONS_RUNBOOK.md](OPERATIONS_RUNBOOK.md)
- [ERROR_CLASSIFICATION.md](ERROR_CLASSIFICATION.md)
