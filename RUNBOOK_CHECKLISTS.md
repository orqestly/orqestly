# Runbook Checklists - Orqestly

## Purpose

This document provides scenario-based operational checklists for common Orqestly deployment and runtime situations.

## Checklist 1: Pre-Deployment

- [ ] confirm CI pipeline passed
- [ ] confirm image digests approved
- [ ] confirm infra-postgres and infra-redis are healthy
- [ ] confirm edge-nginx route is available
- [ ] confirm environment variables are set
- [ ] confirm backup image references exist

## Checklist 2: Post-Deployment Smoke Test

- [ ] confirm API health endpoint returns success
- [ ] confirm worker is reachable and responsive
- [ ] confirm compose services are on the expected network
- [ ] confirm proxy routing reaches API
- [ ] execute a small reference run
- [ ] confirm evidence bundle is created

## Checklist 3: Approval Gate Stuck

- [ ] identify gate type
- [ ] identify assigned approver
- [ ] verify timeout settings
- [ ] apply escalation policy
- [ ] capture reason for delay
- [ ] resume only after decision is recorded

## Checklist 4: Worker Failure

- [ ] review worker logs
- [ ] check Redis connectivity
- [ ] inspect task queue backlog
- [ ] validate worker image version
- [ ] rerun failed task after fix
- [ ] record incident summary

## Checklist 5: Jira Sync Failure

- [ ] verify Jira credentials
- [ ] validate field mapping
- [ ] retry sync once after correction
- [ ] capture request and response payloads
- [ ] log unresolved conflict
- [ ] escalate if failure persists

## Checklist 6: Rollback

- [ ] identify last known-good image digest
- [ ] stop promotion of current release
- [ ] revert compose image references
- [ ] confirm services recover
- [ ] rerun smoke tests
- [ ] document rollback reason

## Checklist 7: Security Incident

- [ ] treat as critical
- [ ] revoke exposed credentials
- [ ] capture affected evidence references
- [ ] notify security owner
- [ ] preserve audit trail
- [ ] verify no further exposure

## Related Documents

- [OPERATIONS_RUNBOOK.md](OPERATIONS_RUNBOOK.md)
- [ERROR_CLASSIFICATION.md](ERROR_CLASSIFICATION.md)
- [CI_CD_RELEASE_CONTRACT.md](CI_CD_RELEASE_CONTRACT.md)
