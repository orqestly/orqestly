# Architecture Overview — Orqestly

## Purpose

This document defines the initial system boundaries for Orqestly. It describes how requirements move through a continuous orchestration cycle, which responsibilities belong to each component, and which parts are intentionally excluded from the first release.

## System Flow

Intake -> Parse -> Plan -> Execute -> Review/Approve -> Decide (Rerun or Close) -> Report

## Core Components

### 1. Requirements Intake

- Accepts Markdown or JSON requirements.
- Normalizes the input into a structured internal format.
- Preserves traceability from source text to derived checks.

### 2. Spec Parser

- Ingests multiple input artifacts (requirements documents, UI/UX specs, database schemas, and API collections).
- Extracts entities, flows, constraints, and assumptions from the input.
- Flags ambiguous statements for human review.
- Produces a schema-friendly representation for downstream components and status-aware planning.

### 3. Test Planner

- Converts parsed requirements into execution scenarios.
- Prioritizes the highest-value user paths first.
- Emits structured plans that can be reviewed before execution.

### 4. Orchestrator

- Coordinates the end-to-end run.
- Selects which agent or executor handles each step.
- Manages retries, state transitions, and result aggregation.
- Owns lifecycle transitions across `pending`, `in_progress`, `blocked`, `needs_review`, `failed`, and `done`.
- Provides deterministic stage transitions for intake, execution, review, and closure.

### 5. Approval Controller (Human-in-the-Loop)

- Opens approval gates at critical transitions.
- Supports approve, reject, request-changes, and defer decisions.
- Applies timeout and fallback policy for unattended decisions.
- Publishes decision history for traceability and auditability.

### 6. Agents

- UI Agent: interacts with browser-based flows.
- API Agent: validates service behavior and response contracts.
- Validation Agent: compares observed behavior to expected behavior.
- Evidence Collector: captures logs, screenshots, and execution metadata.

### 7. Reporting Engine

- Produces structured pass/fail summaries.
- Separates product defects from environment and execution failures.
- Exports evidence in a format suitable for triage.

### 8. Memory Layer

- Stores run-scoped context for active orchestration.
- Stores cross-run memory for prior approvals, defects, and validated patterns.
- Applies retention boundaries for sensitive artifacts and historical context.

### 9. Integration Adapters

- Syncs defects and status transitions with external management tools.
- Starts with Jira-style integration for issue creation and status updates.
- Maintains deterministic mapping between internal statuses and external issue states.

### 10. Skill Runtime

- Registers pluggable skills with explicit contracts.
- Routes planned steps to selected skills at runtime.
- Includes `http_request` as the baseline skill for API validation.
- Captures skill-level evidence and returns normalized results to the orchestrator.

## First Release Boundaries

### In Scope

- Form-based UI validation.
- Basic requirements parsing.
- Evidence capture.
- Structured defect reporting.
- Lifecycle orchestration with explicit state transitions.
- Human approval gates for governance and trust.
- Skill execution for API testing through `http_request`.
- Jira-style issue synchronization.

### Out of Scope

- Full autonomous remediation.
- Large-scale workflow orchestration.
- Complex cross-system discovery.
- Broad enterprise policy enforcement.

## Control Principles

- Keep each component narrowly responsible.
- Preserve traceability across every stage.
- Fail safely when the system cannot infer a requirement.
- Make execution results explainable and reviewable.

## Architectural Decisions That Support Success

- Use structured outputs between components instead of free-form text.
- Keep the first agent set small so quality is easier to control.
- Make evidence a first-class output, not an optional add-on.
- Treat ambiguous requirement interpretation as a review point, not an automation target.

## Relationship To Other Documents

- [SYSTEM_REQUIREMENTS.md](SYSTEM_REQUIREMENTS.md) defines the product scope.
- [TECHNICAL_SPECIFICATION.md](TECHNICAL_SPECIFICATION.md) defines lifecycle contracts, status transitions, memory rules, integration mappings, and skill interfaces.
- [SUCCESS_METRICS.md](SUCCESS_METRICS.md) defines how success is measured.
- [STRATEGY.md](STRATEGY.md) defines the phase roadmap.

This document is the implementation reference for system structure and component boundaries.
