# Phase 1 Execution Plan - Orqestly

## Purpose

This plan turns the current requirements and architecture into an executable Phase 1 delivery roadmap with ownership, milestones, acceptance gates, and operating cadence.

## Phase 1 Outcome

Deliver a working orchestration platform for form-based UI validation with lifecycle status tracking, human approvals, evidence-based defect reporting, and baseline API skill execution.

## Scope For This Phase

- Multi-artifact intake and normalization for supported inputs.
- Orchestration cycle: intake -> plan -> execute -> review -> rerun or close.
- Human-in-the-loop approval gates at critical transitions.
- Status engine with auditable transitions.
- Evidence and reporting pipeline.
- Baseline `http_request` skill.
- Jira-style issue synchronization.

## Workstreams

### 1. Intake and Planning

- Implement normalized intake contract for requirements and related specs.
- Build traceable scenario planning output.
- Add ambiguity flagging for human review.

### 2. Lifecycle and State Engine

- Implement statuses: `pending`, `in_progress`, `blocked`, `needs_review`, `failed`, `done`.
- Enforce valid transition rules.
- Persist status history and transition reason metadata.

### 3. Human Approval Layer

- Implement approval gates: pre-execution, ambiguity, triage, closure.
- Implement actions: approve, reject, request_changes, defer.
- Add timeout and fallback policy per gate.

### 4. Skill Runtime and API Validation

- Implement pluggable skill contract.
- Implement `http_request` skill with deterministic execution and evidence capture.
- Normalize skill outputs for reporting and traceability.

### 5. Reporting and Integrations

- Implement evidence packaging and defect classification.
- Implement Jira-style issue creation and status sync mapping.
- Add sync logging and conflict reconciliation.

### 6. Container Packaging and Deployment

- Build one Docker image per component through GitHub CI/CD.
- Deploy and wire services via `docker-compose.yaml`.
- Reuse `infra-postgres` and `infra-redis` as external dependencies.
- Ensure API routing through `edge-nginx` reverse proxy.

### 7. Operations and Failure Handling

- Define runbook steps for API, worker, Redis, Postgres, and Jira failures.
- Define SLO targets and alert thresholds.
- Define error classification and recovery policy.

### 8. Security and Release Controls

- Define evidence retention and purge rules.
- Define security operations baseline for secrets, access, and audit logs.
- Define CI/CD release contract for build, publish, deploy, and rollback.

### 9. Capacity and Governance Controls

- Define Phase 1 capacity assumptions and scale triggers.
- Define documentation governance SLA and review cadence.
- Define metrics collection specification for milestone reporting.

### 10. CI/CD and Observability Blueprint

- Define detailed CI/CD workflow stages and release gates.
- Define dashboard panels and alert definitions.
- Define runbook checklists for common operational scenarios.

## Milestones

### Milestone 1: Contracts Locked

- Inputs, statuses, approvals, and skill contracts are stable.
- Exit gate: contract review approved.
- Contract artifacts:
	- [STATUS_TRANSITION_TABLE.md](STATUS_TRANSITION_TABLE.md)
	- [APPROVAL_GATE_MATRIX.md](APPROVAL_GATE_MATRIX.md)
	- [JIRA_FIELD_MAPPING.md](JIRA_FIELD_MAPPING.md)

### Milestone 2: Core Orchestration Running

- End-to-end cycle works on one supported reference flow.
- Exit gate: successful run from intake to report with auditable states.
- Reference run artifact:
	- [MILESTONE2_REFERENCE_RUN.md](MILESTONE2_REFERENCE_RUN.md)

### Milestone 3: Governance and Skill Completion

- Human approvals and `http_request` skill fully operational.
- Exit gate: approval events and skill evidence validated.
- Governance test artifact:
	- [MILESTONE3_GOVERNANCE_TEST_PACK.md](MILESTONE3_GOVERNANCE_TEST_PACK.md)

### Milestone 4: Integration and Readiness

- Jira mapping and sync behavior verified.
- Exit gate: defect publication and status sync pass for reference run.
- Deployment gate: compose services are healthy, infra connectivity is verified, and edge proxy routing is validated.

## Ownership Model

- Product owner: scope decisions and acceptance priorities.
- Orchestrator owner: lifecycle engine and transition integrity.
- QA lead: evidence quality and defect taxonomy.
- Integration owner: Jira mapping and sync reliability.
- Skill owner: `http_request` behavior and contract compliance.

## Delivery Cadence

- Weekly planning and risk review.
- Daily execution updates by workstream owners.
- Weekly demo on a fixed reference scenario.
- Weekly metrics check against success indicators.

## Definition Of Done

- Lifecycle runs from intake to closure with no undefined states.
- Human approvals are captured with actor, decision, and timestamps.
- Defects include traceable evidence and requirement linkage.
- Jira issues are created and updated with deterministic mapping.
- `http_request` skill produces consistent, triage-ready outputs.

## Risks And Controls

- Ambiguous requirements slow execution. Control: early review gate and strict normalization.
- Flaky execution creates false signals. Control: retry policy and explicit unstable classification.
- Status drift between systems. Control: sync audit log and reconciliation checks.
- Scope creep. Control: milestone gates and strict phase boundary.

## Dependencies

- [SYSTEM_REQUIREMENTS.md](SYSTEM_REQUIREMENTS.md)
- [ARCHITECTURE.md](ARCHITECTURE.md)
- [TECHNICAL_SPECIFICATION.md](TECHNICAL_SPECIFICATION.md)
- [SUCCESS_METRICS.md](SUCCESS_METRICS.md)
- [TECH_STACK_FREEZE.md](TECH_STACK_FREEZE.md)
- [STATUS_TRANSITION_TABLE.md](STATUS_TRANSITION_TABLE.md)
- [APPROVAL_GATE_MATRIX.md](APPROVAL_GATE_MATRIX.md)
- [JIRA_FIELD_MAPPING.md](JIRA_FIELD_MAPPING.md)
- [MILESTONE2_REFERENCE_RUN.md](MILESTONE2_REFERENCE_RUN.md)
- [MILESTONE3_GOVERNANCE_TEST_PACK.md](MILESTONE3_GOVERNANCE_TEST_PACK.md)
- [PHASE1_ROLLOUT_CHECKLIST.md](PHASE1_ROLLOUT_CHECKLIST.md)
- [SIGNOFF_TEMPLATE.md](SIGNOFF_TEMPLATE.md)
- [DEPLOYMENT_BASELINE.md](DEPLOYMENT_BASELINE.md)
- [docker-compose.yaml](docker-compose.yaml)
- [OPERATIONS_RUNBOOK.md](OPERATIONS_RUNBOOK.md)
- [SLO_AND_ALERTING.md](SLO_AND_ALERTING.md)
- [ERROR_CLASSIFICATION.md](ERROR_CLASSIFICATION.md)
- [EVIDENCE_RETENTION_POLICY.md](EVIDENCE_RETENTION_POLICY.md)
- [SECURITY_OPERATIONS_BASELINE.md](SECURITY_OPERATIONS_BASELINE.md)
- [CI_CD_RELEASE_CONTRACT.md](CI_CD_RELEASE_CONTRACT.md)
- [CAPACITY_PLAN_PHASE1.md](CAPACITY_PLAN_PHASE1.md)
- [DOC_GOVERNANCE_SLA.md](DOC_GOVERNANCE_SLA.md)
- [METRICS_COLLECTION_SPEC.md](METRICS_COLLECTION_SPEC.md)
- [CI_CD_PIPELINE_SPEC.md](CI_CD_PIPELINE_SPEC.md)
- [OBSERVABILITY_DASHBOARD_SPEC.md](OBSERVABILITY_DASHBOARD_SPEC.md)
- [RUNBOOK_CHECKLISTS.md](RUNBOOK_CHECKLISTS.md)

This plan is the execution layer for Phase 1 and should be updated when milestone gates are passed.
