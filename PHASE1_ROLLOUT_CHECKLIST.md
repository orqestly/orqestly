# Phase 1 Rollout Checklist - Orqestly

## Purpose

This checklist provides the release path from Milestone 1 through Milestone 3 with clear verification ownership and sign-off requirements.

## Rollout Sequence

1. Milestone 1 contract lock
2. Milestone 2 reference run validation
3. Milestone 3 governance and skill validation
4. Phase 1 readiness review and release sign-off

## Milestone 1 Checklist (Contracts Locked)

- [ ] Review status transitions in [STATUS_TRANSITION_TABLE.md](STATUS_TRANSITION_TABLE.md)
- [ ] Review approval gates in [APPROVAL_GATE_MATRIX.md](APPROVAL_GATE_MATRIX.md)
- [ ] Review Jira mapping rules in [JIRA_FIELD_MAPPING.md](JIRA_FIELD_MAPPING.md)
- [ ] Confirm contract consistency with [TECHNICAL_SPECIFICATION.md](TECHNICAL_SPECIFICATION.md)
- [ ] Product owner sign-off recorded
- [ ] Orchestrator owner sign-off recorded

## Milestone 2 Checklist (Core Orchestration Running)

- [ ] Execute reference scenario in [MILESTONE2_REFERENCE_RUN.md](MILESTONE2_REFERENCE_RUN.md)
- [ ] Validate status transitions against allowed paths
- [ ] Validate required approval events are captured
- [ ] Validate required evidence bundle completeness
- [ ] Validate Jira sync events for created/updated issues
- [ ] QA lead sign-off recorded

## Milestone 3 Checklist (Governance And Skill Completion)

- [ ] Execute governance tests in [MILESTONE3_GOVERNANCE_TEST_PACK.md](MILESTONE3_GOVERNANCE_TEST_PACK.md)
- [ ] Validate timeout and escalation handling
- [ ] Validate stale-decision behavior
- [ ] Validate `http_request` skill success and failure outputs
- [ ] Validate governance failures appear in reports
- [ ] QA lead sign-off recorded
- [ ] Skill owner sign-off recorded

## Phase 1 Readiness Checklist

- [ ] All milestone sign-offs are complete
- [ ] No unresolved critical blockers
- [ ] Success metrics are reviewed in [SUCCESS_METRICS.md](SUCCESS_METRICS.md)
- [ ] Security constraints are checked against [SECURITY.md](SECURITY.md)
- [ ] Final release decision documented

## Ownership And Sign-Off

- Product owner: scope and release-go/no-go approval
- Orchestrator owner: lifecycle and status integrity approval
- QA lead: evidence, defect quality, and governance approval
- Integration owner: Jira sync behavior approval
- Skill owner: skill contract and runtime output approval

## Exit Condition

Phase 1 is release-ready only when all checklist items are complete and all required sign-offs are recorded.
