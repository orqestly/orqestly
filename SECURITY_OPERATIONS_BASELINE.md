# Security Operations Baseline - Orqestly

## Purpose

This document defines the minimum operational security controls for Phase 1 deployment and runtime usage.

## Security Objectives

- Protect secrets and credentials.
- Limit access to operational and evidence data.
- Preserve audit integrity.
- Reduce data leakage through logs, artifacts, and reports.

## Secrets Management

- Store secrets outside the repository.
- Inject secrets through deployment environment management.
- Rotate credentials on a documented schedule.
- Revoke credentials immediately on compromise.

## Access Model

- Role-based access must apply to approvals, reports, and evidence.
- Admin access must be limited to minimum required operators.
- External integration tokens must be scoped to the minimum required permissions.

## Data Protection

- Encrypt sensitive data at rest where supported by platform controls.
- Use TLS for all network communication between exposed services.
- Mask secrets and PII in logs, screenshots, and exports.

## Audit Logging

- Log security-relevant actions, including sign-in, approval decisions, sync actions, and purge events.
- Protect audit logs from tampering.
- Retain audit logs longer than operational evidence when required for compliance.

## Incident Handling

- Treat secret exposure as a critical incident.
- Treat unauthorized access to evidence as a security incident.
- Treat repeated sync failures with credential errors as a security alert.

## Deployment Checks

- Verify environment variables before deployment.
- Verify no secret values are committed into image layers or compose files.
- Verify runtime containers only receive required credentials.

## Compliance Notes

- Baseline should support future SOC 2-style evidence requests.
- Keep audit and retention policies explicit for future compliance expansion.

## Related Documents

- [TECH_STACK_FREEZE.md](TECH_STACK_FREEZE.md)
- [DEPLOYMENT_BASELINE.md](DEPLOYMENT_BASELINE.md)
- [EVIDENCE_RETENTION_POLICY.md](EVIDENCE_RETENTION_POLICY.md)
