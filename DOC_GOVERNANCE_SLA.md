# Documentation Governance SLA - Orqestly

## Purpose

This document defines ownership, review cadence, and update expectations for the planning and roadmap documentation set.

## Governance Principles

- Every critical document has a named owner role.
- Documents must stay synchronized with implementation decisions.
- Changes to contracts, deployment, or operational behavior must update linked docs.

## Review Cadence

- Weekly review during active Phase 1 implementation.
- Monthly review once Phase 1 stabilizes.
- Immediate update when a milestone gate changes scope or behavior.

## Document Categories And Owners

### Strategy And Requirements

- Owner: Product Owner
- Review focus: scope, outcomes, and roadmap alignment

### Architecture And Technical Contracts

- Owner: Orchestrator Owner
- Review focus: lifecycle rules, system boundaries, and implementation contracts

### Governance And Quality

- Owner: QA Lead
- Review focus: evidence, approvals, error classification, and release readiness

### Integrations And Deployment

- Owner: Integration Owner
- Review focus: Jira sync, deployment topology, and infra compatibility

### Skills And Runtime Behavior

- Owner: Skill Owner
- Review focus: skill contracts, evidence format, and execution consistency

## Update Triggers

- New milestone added or removed.
- New failure mode discovered in reference run.
- Deployment topology changes.
- Stack freeze changes.
- Metrics or SLOs are revised.

## Staleness Rules

- If a linked artifact changes, dependent docs must be reviewed before the next release gate.
- If a doc is not reviewed for a full quarter, it is considered stale and must be revalidated.

## Sign-Off Requirements

- Major planning changes require owner review and sign-off.
- Release-related changes require product, orchestration, and QA confirmation.

## Related Documents

- [PHASE1_ROLLOUT_CHECKLIST.md](PHASE1_ROLLOUT_CHECKLIST.md)
- [SIGNOFF_TEMPLATE.md](SIGNOFF_TEMPLATE.md)
- [TECH_STACK_FREEZE.md](TECH_STACK_FREEZE.md)
