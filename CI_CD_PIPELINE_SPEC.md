# CI/CD Pipeline Specification - Orqestly

## Purpose

This document defines the detailed GitHub CI/CD workflow for building, validating, publishing, and deploying Orqestly services as Docker images.

## Pipeline Goals

- Build repeatable images for each service component.
- Prevent contract or quality regressions from reaching the registry.
- Publish immutable image references for deployment.
- Deploy through Docker Compose against the existing VPS infrastructure.
- Verify service health and routing before release completion.

## Trigger Conditions

- pull request to main branches: validation only
- merge to main: build, test, publish, and deploy pipeline
- manual release tag: release publish and deployment promotion
- emergency hotfix: expedited validation with owner approval

## Pipeline Stages

### 1. Source Validation

- format checks
- lint checks
- type checks
- unit tests
- contract checks for roadmap docs and manifests when applicable

### 2. Build Preparation

- restore dependencies
- cache package artifacts
- compute version metadata from commit SHA and release tag

### 3. Image Build

- build API image
- build worker image
- apply consistent labels and version metadata
- ensure build uses frozen stack dependencies only

### 4. Security And Quality Checks

- verify no secret leakage in build artifacts
- confirm image metadata matches source revision
- run container-level smoke checks when supported

### 5. Registry Publish

- publish immutable image digests
- record image references in release artifact
- prevent overwrite of released tags

### 6. Compose Deployment

- update compose image references
- deploy API and worker to the VPS runtime
- reuse `infra-postgres`, `infra-redis`, and `edge-nginx`

### 7. Post-Deploy Verification

- run API health check
- run worker readiness check
- confirm proxy routing
- run reference smoke test

## Release Gates

- no publish without passing validation
- no deployment without approved digest
- no promotion without healthy post-deploy checks

## Rollback Rules

- preserve last known-good image set
- roll back on failed health checks or routing errors
- log rollback reason and owning role

## Artifacts Produced

- test logs
- build logs
- image digests
- deployment revision record
- health verification results
- rollback record if needed

## Ownership

- Orchestrator owner: workflow integrity
- Integration owner: registry and deploy wiring
- QA lead: validation and smoke tests
- Product owner: release approval

## Related Documents

- [TECH_STACK_FREEZE.md](TECH_STACK_FREEZE.md)
- [CI_CD_RELEASE_CONTRACT.md](CI_CD_RELEASE_CONTRACT.md)
- [DEPLOYMENT_BASELINE.md](DEPLOYMENT_BASELINE.md)
