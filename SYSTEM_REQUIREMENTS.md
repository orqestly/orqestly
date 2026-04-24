# System Requirements — Orqestly

## Purpose

This document defines the product requirements for Orqestly's first implementation stage. It turns the current strategy and positioning into a buildable system request so the project can move from vision to execution with clear scope, measurable outcomes, and explicit success criteria.

## Product Goal

Orqestly will validate software behavior against human intent by converting requirements into executable validation flows, running them against real systems, and producing evidence-based defect reports.

## Problem To Solve

Current testing workflows are fragmented:

- Requirements are documented separately from test execution.
- Test cases drift from the original intent.
- Manual scripting creates maintenance overhead.
- Most tools validate staging or isolated scenarios instead of the running system.

Orqestly exists to close the gap between what was intended and what is actually working.

## Target Users

- QA engineers who need faster, requirement-driven validation.
- Engineering teams that want real-system confidence before release.
- Founders and product teams who need a lightweight way to verify critical flows.
- DevOps and platform teams that want validation integrated into delivery workflows.

## Scope

### In Scope for Phase 1

- Ingesting requirements and related specifications in Markdown or JSON form.
- Converting requirements into structured validation scenarios.
- Executing UI validation for form-based applications.
- Collecting evidence such as screenshots, logs, and structured results.
- Producing a report of passed checks, failed checks, and likely defects.
- Running a full orchestration cycle with explicit approval and status transitions.

### Out of Scope for Phase 1

- Full autonomous remediation.
- Broad enterprise workflow orchestration.
- Deep cross-system dependency discovery.
- Replacement of full test management suites.

## Functional Requirements

### Requirement Intake

- The system must accept a requirements document and a target application identifier or URL.
- The system must aggregate multiple specification artifacts in a single intake session and normalize them into a unified validation plan.
- The system must normalize requirements into a structured internal representation.
- The system must preserve traceability from each requirement to each generated validation step.

### Test Planning

- The system must derive validation scenarios from the supplied requirements.
- The system must prioritize core user flows before edge cases.
- The system must expose structured outputs suitable for review before execution.

### Execution

- The system must support browser-based UI validation for simple form-driven flows.
- The system must execute steps in a deterministic order when possible.
- The system must support retries for transient failures and mark unstable results clearly.
- The system must run as a cycle: intake -> planning -> execution -> review -> decision -> rerun or close.
- The system must maintain orchestration and test-unit statuses across the cycle.

### Lifecycle State Management

- The system must support at least these statuses: `pending`, `in_progress`, `blocked`, `needs_review`, `failed`, and `done`.
- The system must enforce valid state transitions and reject invalid status jumps.
- The system must expose current, next, and pending work states for every orchestration run.
- The system must persist status history for audit and debugging.

### Evidence and Reporting

- The system must capture artifacts that support defect triage.
- The system must generate a structured report of observed behavior versus expected behavior.
- The system must distinguish between assertion failures, environmental failures, and execution errors.

### Human Review and Safety

- The system must allow human review before execution when the generated interpretation is ambiguous.
- The system must not silently infer unsupported requirements.
- The system must keep a record of assumptions made during parsing and planning.
- The system must provide human approval gates for major transitions, including pre-execution approval, ambiguity resolution, defect triage approval, and closure approval.
- The system must support approve, reject, request-changes, and defer actions at each approval gate.
- The system must apply timeout and fallback behavior for unattended approval requests.

### Memory and Tool Integrations

- The system must support run-scoped memory for current execution context.
- The system must support cross-run memory for prior decisions, known failures, and validated patterns.
- The system must support configurable retention boundaries for memory and evidence artifacts.
- The system must support integration contracts for engineering management tools, starting with Jira-style issue synchronization.
- The system must map defect outputs to issue fields and sync lifecycle status updates.

### Skill Framework

- The orchestrator must support pluggable skills with defined input, output, and evidence contracts.
- The first required skill is `http_request` for API validation across request, response, status-code, and payload checks.
- The orchestrator must route task steps to the appropriate skill at runtime.
- Skill execution results must be traceable to requirements and included in reports.

## Non-Functional Requirements

- The system should be reproducible across runs when the input requirements do not change.
- The system should fail safely when credentials, permissions, or environment data are missing.
- The system should produce outputs that are easy to inspect, export, and compare.
- The system should avoid storing sensitive data unless explicitly required.

## Success Metrics

Orqestly succeeds if the first release can demonstrate:

- Clear requirement-to-test traceability for the main flow.
- Reliable execution on a narrow class of form-based applications.
- Useful defect reports that reduce triage time.
- A measurable reduction in manual test authoring for supported flows.
- Repeatable validation results for the same input and environment.
- Reliable approval turnaround across human-in-the-loop gates.
- Consistent and auditable status transitions through the orchestration cycle.
- Stable synchronization of defect and status data into management tools.

## Risks And Mitigations

- AI misinterprets requirements. Mitigation: structured parsing, constrained schemas, and human review for ambiguous inputs.
- UI automation becomes flaky. Mitigation: retry policy, stable selectors, and a narrow supported app class in Phase 1.
- Scope expands too early. Mitigation: keep the first release focused on one validation path and one application style.
- Results become hard to trust. Mitigation: strong evidence capture and explicit separation of environment failures from product failures.

## Phase Mapping

### Phase 1

- UI validation for form-based applications.
- Simple requirements parsing.
- Structured defect reporting.
- Evidence collection and traceability.

### Phase 2

- API validation.
- Workflow testing.
- Expanded agent capability.

### Phase 3

- Full orchestration.
- Multi-system validation.
- CI/CD integration.

## Ways This Idea Succeeds

To make the product succeed, the team should follow these operating principles:

- Start with one narrow, real use case and prove value before broadening scope.
- Treat requirements as the primary input, not test scripts.
- Make every generated check explainable and reviewable.
- Optimize for evidence and triage speed, not just pass/fail output.
- Keep security and data handling explicit from the beginning.
- Measure quality with a small set of concrete metrics instead of broad ambition.

## Open Decisions

- The first supported requirements format may need to be limited to a strict Markdown template or JSON schema.
- The first supported application type may need to be constrained to simple form-based flows only.
- The reporting format may need to be standardized before integration work begins.

## Relationship To Other Documents

- [STRATEGY.md](STRATEGY.md) defines the roadmap phases.
- [POSITIONING.md](POSITIONING.md) defines how Orqestly is differentiated.
- [MARKET_GAP.md](MARKET_GAP.md) explains why the product exists.
- [SECURITY.md](SECURITY.md) defines the security reporting process and scope.
- [TECHNICAL_SPECIFICATION.md](TECHNICAL_SPECIFICATION.md) defines lifecycle contracts, status transitions, memory model, integration mapping, and skill interfaces.

This document is the implementation-facing source of truth for the initial product scope.
