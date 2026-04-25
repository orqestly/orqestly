# Team Ownership RACI - Phase 1

## Purpose

Define the teams required to deliver Orqestly Phase 1 and the concrete outcomes each team is accountable for at milestone gates.

## Team Structure

- Product and Program Management
- Architecture and Technical Leadership
- Platform Backend (API and Orchestrator)
- Data Engineering and DBA
- Skill Runtime and Automation Engineering
- Integrations (Jira and Notifications)
- QA and Validation Engineering
- DevOps and SRE
- Security and Compliance
- Documentation and Enablement

## RACI Key

- R: Responsible for delivery execution.
- A: Accountable for final acceptance.
- C: Consulted during design and review.
- I: Informed of progress and decisions.

## Outcome Matrix By Team

| Team | Required Outcomes | Milestone Gate | Acceptance Evidence |
|---|---|---|---|
| Product and Program Management | Scope lock, milestone planning, dependency tracking, decision escalation | M1, M2, M3, M4 | Signed scope baseline, weekly risk log, gate decision records |
| Architecture and Technical Leadership | Stable contracts across lifecycle, approval, skill, reporting, integration boundaries | M1 | Approved architecture and contract review with zero open blockers |
| Platform Backend (API and Orchestrator) | End-to-end orchestration lifecycle from intake to closure with valid status transitions and approval hooks | M2, M3 | Reference run passes with auditable state history |
| Data Engineering and DBA | Schema, migrations, indexing, retention jobs, backup and restore readiness | M2, M4 | Migration baseline applied, retention job validated, restore drill within target |
| Skill Runtime and Automation Engineering | Pluggable skill runtime and baseline skill contract compliance with deterministic outputs | M3 | Skill test pack pass and evidence records linked per run |
| Integrations (Jira and Notifications) | Deterministic Jira sync mapping, retry handling, conflict resolution logging | M4 | Sync success criteria met on reference run with reconciliation logs |
| QA and Validation Engineering | Functional and governance validation packs with defect quality checks | M2, M3, M4 | Execution reports for reference and governance packs, defect quality review signed |
| DevOps and SRE | CI/CD gates, deploy verification, observability dashboards, incident and rollback readiness | M4 | Green release pipeline, deployment verification checklist, rollback rehearsal |
| Security and Compliance | Secrets and access controls, audit logging, evidence retention and redaction controls | M3, M4 | Security baseline checklist pass and auditability verification |
| Documentation and Enablement | Current, linked, implementation-ready documentation and handoff clarity | M1, M4 | No broken links, SLA review complete, implementation handoff approved |

## Cross-Team RACI For Core Deliverables

| Deliverable | Product/Program | Architecture | Backend | Data/DBA | Skill Runtime | Integrations | QA | DevOps/SRE | Security | Docs |
|---|---|---|---|---|---|---|---|---|---|---|
| Lifecycle and status engine | C | A | R | I | C | I | C | I | I | I |
| Human approval gates | C | A | R | I | I | I | C | I | C | I |
| Baseline skill runtime and http_request | I | C | C | I | A/R | I | C | I | I | I |
| Jira sync and reconciliation | I | C | C | I | I | A/R | C | I | I | I |
| Evidence retention and auditability | I | C | C | R | I | I | C | I | A | C |
| CI/CD and deployment readiness | I | C | C | I | I | I | C | A/R | C | I |
| Observability and runbook readiness | I | C | C | I | I | I | C | A/R | C | C |
| Documentation completeness and handoff | C | C | I | I | I | I | I | I | I | A/R |

## Milestone Exit Ownership

### M1 - Contracts Locked

- Accountable: Architecture and Technical Leadership
- Responsible: Product and Program Management, Documentation and Enablement
- Required evidence:
  - Contract set reviewed and approved.
  - No unresolved contract conflicts.

### M2 - Core Orchestration Running

- Accountable: Platform Backend
- Responsible: Backend, Data Engineering and DBA, QA
- Required evidence:
  - Reference run from intake to report is successful.
  - Status transitions are valid and fully auditable.

### M3 - Governance and Skill Completion

- Accountable: Skill Runtime and Automation Engineering
- Responsible: Skill Runtime, Backend, Security, QA
- Required evidence:
  - Approval gate behavior validated including timeout paths.
  - Baseline skill behavior and evidence contracts validated.

### M4 - Integration and Readiness

- Accountable: DevOps and SRE
- Responsible: Integrations, DevOps and SRE, QA, Security
- Required evidence:
  - Jira sync and reconciliation checks pass.
  - Release pipeline and rollback checks pass.
  - Operational checklist and alert routing pass.

## Handoff Outcomes Required From Every Team

- Product and Program Management: approved scope baseline and milestone gate decisions.
- Architecture and Technical Leadership: approved technical contract baseline for implementation.
- Platform Backend: validated orchestration flow and lifecycle API contract readiness.
- Data Engineering and DBA: migration-ready schema and tested recovery procedure.
- Skill Runtime and Automation Engineering: baseline skill set ready with deterministic result contracts.
- Integrations: Jira mapping behavior validated and reconciliation process documented.
- QA and Validation Engineering: test packs ready for CI gate enforcement.
- DevOps and SRE: release pipeline and operational readiness checks validated.
- Security and Compliance: evidence retention and access controls validated.
- Documentation and Enablement: linked and complete implementation handoff package.

## References

- [PHASE1_EXECUTION_PLAN.md](PHASE1_EXECUTION_PLAN.md)
- [PHASE1_ROLLOUT_CHECKLIST.md](PHASE1_ROLLOUT_CHECKLIST.md)
- [SIGNOFF_TEMPLATE.md](SIGNOFF_TEMPLATE.md)
- [TECHNICAL_SPECIFICATION.md](TECHNICAL_SPECIFICATION.md)
- [CI_CD_PIPELINE_SPEC.md](CI_CD_PIPELINE_SPEC.md)
- [RUNBOOK_CHECKLISTS.md](RUNBOOK_CHECKLISTS.md)
