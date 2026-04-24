# Orqestly

**AI-Orchestrated Software Testing Platform**

Orqestly is an open-source platform that converts software requirements into executable test plans and autonomously validates real, running applications using AI agents.

> From requirements → to tests → to verified reality.

---

## 🚀 What Problem Does Orqestly Solve?

Traditional testing:

* Tests are written manually
* Often drift from actual requirements
* Rarely validate real production behavior

Orqestly:

* Parses requirements using AI
* Generates structured test plans
* Executes tests on live systems
* Compares expected vs actual behavior
* Produces structured defect reports

---

## 🧠 Core Concept

```text
Requirements + Specs → Plan → Execute → Review/Approve → Rerun/Close → Verified Reports
```

## 📘 System Requirements

The implementation source of truth is [SYSTEM_REQUIREMENTS.md](SYSTEM_REQUIREMENTS.md). It defines the first product scope, success metrics, risks, and phase mapping for Orqestly.

## 📚 Documentation Map

- [SYSTEM_REQUIREMENTS.md](SYSTEM_REQUIREMENTS.md) — product scope and requirements
- [TECHNICAL_SPECIFICATION.md](TECHNICAL_SPECIFICATION.md) — lifecycle contracts, approvals, statuses, memory, integrations, and skills
- [ARCHITECTURE.md](ARCHITECTURE.md) — system boundaries and component roles
- [SUCCESS_METRICS.md](SUCCESS_METRICS.md) — measurable success criteria
- [PHASE1_EXECUTION_PLAN.md](PHASE1_EXECUTION_PLAN.md) — milestone-based execution roadmap with ownership and delivery gates
- [STATUS_TRANSITION_TABLE.md](STATUS_TRANSITION_TABLE.md) — lifecycle status transition contracts
- [APPROVAL_GATE_MATRIX.md](APPROVAL_GATE_MATRIX.md) — human approval gates and decision policies
- [JIRA_FIELD_MAPPING.md](JIRA_FIELD_MAPPING.md) — deterministic defect and status mapping for Jira sync
- [MILESTONE2_REFERENCE_RUN.md](MILESTONE2_REFERENCE_RUN.md) — canonical end-to-end validation run for Milestone 2
- [MILESTONE3_GOVERNANCE_TEST_PACK.md](MILESTONE3_GOVERNANCE_TEST_PACK.md) — governance and skill validation test pack for Milestone 3
- [PHASE1_ROLLOUT_CHECKLIST.md](PHASE1_ROLLOUT_CHECKLIST.md) — release-readiness checklist with milestone sign-off gates
- [TECH_STACK_FREEZE.md](TECH_STACK_FREEZE.md) — frozen Phase 1 stack and technology decisions
- [SIGNOFF_TEMPLATE.md](SIGNOFF_TEMPLATE.md) — standard approval template for milestone and release gates
- [DEPLOYMENT_BASELINE.md](DEPLOYMENT_BASELINE.md) — container deployment assumptions and VPS infra connectivity baseline
- [docker-compose.yaml](docker-compose.yaml) — service wiring starter for API/worker with external infra and edge proxy network
- [OPERATIONS_RUNBOOK.md](OPERATIONS_RUNBOOK.md) — common operations, troubleshooting, and recovery procedures
- [SLO_AND_ALERTING.md](SLO_AND_ALERTING.md) — Phase 1 service-level targets and alert thresholds
- [ERROR_CLASSIFICATION.md](ERROR_CLASSIFICATION.md) — failure taxonomy for orchestration, integrations, and skills
- [EVIDENCE_RETENTION_POLICY.md](EVIDENCE_RETENTION_POLICY.md) — evidence storage, retention, and purge rules
- [SECURITY_OPERATIONS_BASELINE.md](SECURITY_OPERATIONS_BASELINE.md) — operational security controls and access baseline
- [CI_CD_RELEASE_CONTRACT.md](CI_CD_RELEASE_CONTRACT.md) — CI/CD build, image, and release rules
- [CAPACITY_PLAN_PHASE1.md](CAPACITY_PLAN_PHASE1.md) — initial workload and resource assumptions for Phase 1
- [DOC_GOVERNANCE_SLA.md](DOC_GOVERNANCE_SLA.md) — documentation ownership and review cadence
- [METRICS_COLLECTION_SPEC.md](METRICS_COLLECTION_SPEC.md) — how success metrics are collected and reported
- [CI_CD_PIPELINE_SPEC.md](CI_CD_PIPELINE_SPEC.md) — detailed GitHub CI/CD workflow blueprint
- [OBSERVABILITY_DASHBOARD_SPEC.md](OBSERVABILITY_DASHBOARD_SPEC.md) — concrete dashboard panels and alerts
- [RUNBOOK_CHECKLISTS.md](RUNBOOK_CHECKLISTS.md) — per-scenario operational checklists

---

## ⚙️ Key Features

* AI-powered requirement parsing
* Autonomous multi-agent testing system
* UI, API, and functional validation
* Lifecycle orchestration with explicit status tracking
* Human-in-the-loop approvals and governance gates
* Persistent orchestration memory across runs
* Integration adapters for management tools (for example Jira)
* Skill-based runtime (starting with HTTP request/API validation)
* Evidence collection (logs, screenshots)
* Structured defect generation (CSV/JSON)
* CI/CD integration ready

---

## 🧩 Architecture Overview

* **Orchestrator**: Coordinates agents and execution flow
* **Spec Parser**: Converts requirements into structured data
* **Test Planner**: Generates test scenarios
* **Agents**:

  * UI Agent
  * API Agent
  * Validation Agent
  * Evidence Collector
* **Reporting Engine**

---

## 📦 Project Structure (Planned)

```text
orqestly/
├── core/
├── agents/
├── spec-parser/
├── test-planner/
├── execution-engine/
├── reporting/
└── integrations/
```

---

## 🛠️ Tech Stack (Planned)

* Python
* LLM APIs (OpenAI or compatible)
* Playwright (UI automation)
* REST clients for API testing
* GitHub Actions (CI/CD)

---

## 🧪 Example Use Case

1. Provide requirements (Markdown / JSON)
2. Provide target system URL
3. Orqestly:

   * generates test cases
   * runs them automatically
   * reports defects

---

## 🤝 Contributing

We welcome contributions in:

* AI/LLM engineering
* Test automation
* DevOps & CI/CD
* Security testing

See `CONTRIBUTING.md` for details.

---

## 📜 License

This project is licensed under a custom license (see LICENSE file).

---

## 🔐 Security

Please report vulnerabilities via `SECURITY.md`.

---

## ⚠️ Disclaimer

Orqestly is an evolving system. Results depend on input quality and system behavior.

---

## 📬 Vision

Orqestly aims to redefine software testing by making it:

* autonomous
* requirement-driven
* production-aware

---

## ⭐ Support

If you find this project useful, consider starring the repo.
