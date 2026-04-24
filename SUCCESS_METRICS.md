# Success Metrics — Orqestly

## Purpose

This document defines what success looks like for the first release of Orqestly. The metrics are intentionally narrow so the team can prove value before expanding scope.

## Primary Success Metrics

### 1. Requirement Traceability

- Every generated validation step should map back to a source requirement.
- The first release should preserve traceability for the main user flow.

### 2. Execution Reliability

- Repeated runs on the same supported flow should produce consistent results.
- Transient failures should be clearly identified instead of being reported as product defects.

### 3. Defect Report Quality

- Reports should clearly explain what failed, where it failed, and what evidence supports the conclusion.
- Reports should reduce triage time by separating environment issues from application issues.

### 4. Scope Efficiency

- The platform should demonstrate value on a narrow class of form-based applications before expanding.
- The team should avoid broadening scope until the first supported path is stable.

### 5. Manual Effort Reduction

- The system should reduce the need to handwrite test cases for the supported flows.
- The team should be able to show that requirements can be converted into usable validation runs with less manual work.

### 6. Human-in-the-Loop Governance

- Approval gates should provide predictable response windows for pre-execution, ambiguity handling, triage, and closure.
- Decisions should be fully auditable with actor, timestamp, and rationale.

### 7. Lifecycle Status Integrity

- Orchestration runs and test units should maintain valid transitions across `pending`, `in_progress`, `blocked`, `needs_review`, `failed`, and `done`.
- The platform should reliably expose what is done, what is next, and what is pending.

### 8. Integration Effectiveness

- Defects should sync into management tools with deterministic field mapping.
- Status changes in orchestration should remain consistent with linked external issues.

### 9. Memory and Reuse Quality

- Cross-run memory should reduce repeated ambiguity and repeated misclassification.
- Historical context should improve rerun efficiency for known workflows.

### 10. Skill Runtime Quality

- The baseline `http_request` skill should consistently validate API behavior and produce triage-ready evidence.
- Skill outputs should remain normalized for downstream validation and reporting.

## Operational Success Signals

- A reviewer can understand why a test was generated.
- A developer can reproduce a failure using the captured evidence.
- A product owner can see how requirements were translated into validation.
- The first supported flow completes without requiring custom scripts for every run.
- Approval requests are resolved without blocking the orchestration cycle unexpectedly.
- Status dashboards remain consistent with actual execution and issue-tracker state.
- Reruns use prior memory and avoid redoing already validated decisions.

## Anti-Goals For Success Measurement

- Counting total number of features shipped without proving traceability.
- Measuring AI output volume instead of validation quality.
- Expanding into every possible test type before the core flow is stable.

## Suggested Baseline Targets

These targets should be refined once the first implementation is benchmarked:

- 100 percent traceability for supported main-flow steps.
- Low false-positive rate for supported scenarios.
- Clear classification of execution failures versus product defects.
- Reduced time to produce a triage-ready defect report.
- High status-transition validity across runs.
- High defect-to-issue synchronization success for integrated workflows.

## Relationship To Other Documents

- [SYSTEM_REQUIREMENTS.md](SYSTEM_REQUIREMENTS.md) defines the product goals and scope.
- [ARCHITECTURE.md](ARCHITECTURE.md) defines how the system is structured.
- [STRATEGY.md](STRATEGY.md) defines the phased roadmap.

This document should be used whenever scope decisions need to be checked against measurable outcomes.
