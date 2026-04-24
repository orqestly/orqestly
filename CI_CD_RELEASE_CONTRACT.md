# CI/CD Release Contract - Orqestly

## Purpose

This document defines how GitHub CI/CD builds, validates, tags, and publishes Orqestly component images for Docker Compose deployment.

## Release Goals

- Build one image per component.
- Validate build artifacts before publish.
- Tag releases in a reproducible way.
- Deploy via Docker Compose using published images.
- Verify post-deploy health and routing.

## Pipeline Stages

### 1. Source Validation

- Lint, type check, and unit test the repository.
- Reject builds on formatting or contract violations.

### 2. Image Build

- Build component images for API, worker, and any supporting service images.
- Use deterministic build context and pinned dependencies.

### 3. Image Tagging

- Tag images by branch, commit SHA, and release version when applicable.
- Avoid mutable tags for release promotion.

### 4. Registry Publish

- Publish images to the configured registry.
- Capture image digest for deployment references.

### 5. Deployment Update

- Update `docker-compose.yaml` image references to the approved release digest/tag.
- Reuse `infra-postgres`, `infra-redis`, and `edge-nginx` as external dependencies.

### 6. Post-Deploy Verification

- Run health checks for API and worker services.
- Validate edge proxy routing.
- Execute a small smoke test before release completion.

## Release Gates

- No publish without passing tests.
- No deployment without image digest approval.
- No release without successful post-deploy verification.

## Rollback Contract

- Preserve previous known-good image digest.
- Roll back to the last successful image set on failed health checks.
- Log rollback reason and affected services.

## Artifact Requirements

- build logs
- test logs
- image digests
- deployment version record
- post-deploy health results

## Ownership

- Orchestrator owner: build/deploy flow integrity
- Product owner: release approval
- Integration owner: registry and deploy connectivity
- QA lead: smoke-test and verification gate

## Related Documents

- [TECH_STACK_FREEZE.md](TECH_STACK_FREEZE.md)
- [DEPLOYMENT_BASELINE.md](DEPLOYMENT_BASELINE.md)
- [PHASE1_ROLLOUT_CHECKLIST.md](PHASE1_ROLLOUT_CHECKLIST.md)
- [CI_CD_PIPELINE_SPEC.md](CI_CD_PIPELINE_SPEC.md)
