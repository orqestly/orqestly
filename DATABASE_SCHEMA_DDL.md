# Database Schema DDL

**Version**: 1.0  
**Last Updated**: April 2026  
**Owner**: Platform Engineering  
**Status**: Locked for Phase 1

---

## Table of Contents

1. [Overview](#overview)
2. [Schema Design Principles](#schema-design-principles)
3. [DDL Statements](#ddl-statements)
4. [Migrations](#migrations)
5. [Indexes & Performance](#indexes--performance)
6. [Retention & Purging](#retention--purging)
7. [Backup & Recovery](#backup--recovery)

---

## Overview

This document defines the PostgreSQL 17 schema for Orqestly. The schema is designed to:
- Support the orchestration lifecycle (intake → parse → plan → execute → review/approve → decide → report)
- Track multi-artifact validation, approval gates, and cross-run memory
- Enforce evidence retention policies with tiered boundaries (run/cross-run/defect/audit)
- Enable deterministic Jira synchronization with conflict resolution
- Provide audit trails for compliance and forensic investigation

**Database Connection**:
```
Host: infra-postgres (VPS)
Port: 5432
Database: orqestly
Timezone: UTC
```

---

## Schema Design Principles

### 1. **Normalization & Integrity**
- 3NF: All tables normalized to eliminate redundancy
- Foreign key constraints enforce referential integrity
- Check constraints validate enum values (status, approval type, etc.)
- NOT NULL constraints on critical fields

### 2. **Immutability & Audit**
- Core run/artifact/approval records are write-once
- `created_at` timestamp on all tables (server-generated UTC)
- `updated_at` only on mutable tables (evidence, Jira sync state)
- Audit log table captures all transitions with actor identity

### 3. **Retention & Lifecycle**
- Evidence categorized by type (run_log, artifact, approval_decision, skill_execution, jira_sync_state)
- `retention_class` column (run/cross_run/defect/audit) determines purge boundaries
- `expires_at` computed at insert time based on retention policy
- Batch purge job removes records where `expires_at <= now()`

### 4. **Jira Integration**
- Dedicated `jira_sync_state` table tracks deterministic state of each Orqestly run
- Idempotent sync operations: `external_id` + `external_type` unique key prevents duplicates
- Conflict resolution via `last_sync_ts` and `conflict_resolution_strategy`
- Sync errors logged separately with retry metadata

### 5. **Cross-Run Memory & Decision History**
- `cross_run_memory` table stores reusable patterns, decision history, learned heuristics
- Scoped by `memory_scope` (global/project/artifact_type/skill) and `memory_key`
- TTL configurable per scope; searchable by embeddings or exact match
- Linked to originating runs for traceability

---

## DDL Statements

### Phase 1 Base Tables

```sql
-- ============================================================================
-- RUNS TABLE
-- ============================================================================
-- Core orchestration run record. Immutable except for terminal state updates.

CREATE TABLE runs (
    id BIGSERIAL PRIMARY KEY,
    run_uuid UUID NOT NULL UNIQUE DEFAULT gen_random_uuid(),
    
    -- Lifecycle state machine
    status VARCHAR(32) NOT NULL CHECK (status IN (
        'pending', 'in_progress', 'blocked', 'needs_review', 'failed', 'done'
    )),
    
    -- Intake metadata
    title VARCHAR(1024) NOT NULL,
    description TEXT,
    ingestion_source VARCHAR(64) NOT NULL CHECK (ingestion_source IN (
        'api', 'jira', 'webhook', 'manual', 'automation'
    )),
    external_id VARCHAR(256),  -- Reference to upstream system (e.g., Jira issue key)
    external_type VARCHAR(64),  -- Type of external reference (e.g., 'jira_issue')
    
    -- Artifacts (JSON array of artifact UUIDs)
    artifact_uuids UUID[] NOT NULL DEFAULT '{}',
    
    -- Orchestration timeline (UTC timestamps)
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    started_at TIMESTAMP,
    review_requested_at TIMESTAMP,
    approved_at TIMESTAMP,
    completed_at TIMESTAMP,
    
    -- Approval gate metadata
    approval_required BOOLEAN NOT NULL DEFAULT false,
    approval_type VARCHAR(32),  -- pre_execution, ambiguity, triage, closure
    approval_assigned_to VARCHAR(256),  -- Email of assigned approver
    approval_timeout_seconds INT,
    approval_escalation_to VARCHAR(256),  -- Escalation email if timeout
    
    -- Closure decision
    decision VARCHAR(64),  -- approve, reject, hold, defer
    decision_reason TEXT,
    decision_metadata JSONB DEFAULT '{}',
    
    -- Result summary
    test_result_summary JSONB DEFAULT '{}',  -- Aggregated results
    error_classification VARCHAR(64),  -- transient_infra, product_defect, etc.
    error_detail TEXT,
    
    -- Cross-run linkage
    parent_run_uuid UUID REFERENCES runs(run_uuid) ON DELETE SET NULL,
    
    -- Metadata
    environment_tag VARCHAR(128),  -- dev, staging, prod, etc.
    tags JSONB DEFAULT '{}',
    
    CONSTRAINT external_id_requires_type CHECK (
        (external_id IS NULL AND external_type IS NULL) OR
        (external_id IS NOT NULL AND external_type IS NOT NULL)
    )
);

CREATE INDEX idx_runs_status ON runs(status);
CREATE INDEX idx_runs_created_at ON runs(created_at DESC);
CREATE INDEX idx_runs_external_id ON runs(external_id, external_type) WHERE external_id IS NOT NULL;
CREATE INDEX idx_runs_approval_assigned_to ON runs(approval_assigned_to) WHERE approval_required = true;

-- ============================================================================
-- ARTIFACTS TABLE
-- ============================================================================
-- Immutable record of each artifact ingested during intake phase.

CREATE TABLE artifacts (
    id BIGSERIAL PRIMARY KEY,
    artifact_uuid UUID NOT NULL UNIQUE DEFAULT gen_random_uuid(),
    run_uuid UUID NOT NULL REFERENCES runs(run_uuid) ON DELETE CASCADE,
    
    -- Artifact identity
    artifact_type VARCHAR(64) NOT NULL CHECK (artifact_type IN (
        'requirement', 'specification', 'schema', 'api_contract', 'test_plan', 'code', 'other'
    )),
    artifact_name VARCHAR(512) NOT NULL,
    artifact_hash VARCHAR(64),  -- SHA256 for change detection
    
    -- Storage
    content_encoding VARCHAR(32) DEFAULT 'utf8',  -- utf8, gzip, etc.
    content_type VARCHAR(128),  -- application/json, text/plain, etc.
    content_bytes INT,  -- Size in bytes for quota enforcement
    storage_uri VARCHAR(1024),  -- S3/blob reference or inline below
    content_inline BYTEA,  -- For small artifacts (<100KB)
    
    -- Validation state
    parse_status VARCHAR(32) NOT NULL CHECK (parse_status IN (
        'pending', 'parsing', 'valid', 'invalid', 'unrecognized'
    )),
    parse_error TEXT,
    parsed_metadata JSONB DEFAULT '{}',
    
    -- Traceability
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    retention_class VARCHAR(16) NOT NULL DEFAULT 'run' CHECK (retention_class IN (
        'run', 'cross_run', 'defect', 'audit'
    )),
    expires_at TIMESTAMP NOT NULL,
    
    UNIQUE(run_uuid, artifact_type, artifact_name)
);

CREATE INDEX idx_artifacts_run_uuid ON artifacts(run_uuid);
CREATE INDEX idx_artifacts_type ON artifacts(artifact_type);
CREATE INDEX idx_artifacts_expires_at ON artifacts(expires_at);

-- ============================================================================
-- APPROVAL_GATES TABLE
-- ============================================================================
-- Event log of approval gate transitions.

CREATE TABLE approval_gates (
    id BIGSERIAL PRIMARY KEY,
    run_uuid UUID NOT NULL REFERENCES runs(run_uuid) ON DELETE CASCADE,
    
    -- Gate metadata
    gate_type VARCHAR(32) NOT NULL CHECK (gate_type IN (
        'pre_execution', 'ambiguity', 'triage', 'closure'
    )),
    gate_sequence INT NOT NULL,  -- Ordinal position in run lifecycle
    
    -- Gate state machine
    gate_status VARCHAR(32) NOT NULL CHECK (gate_status IN (
        'pending', 'open', 'approved', 'rejected', 'escalated', 'timeout'
    )),
    
    -- Assignment & timing
    assigned_to VARCHAR(256) NOT NULL,
    assigned_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    timeout_seconds INT,
    timeout_at TIMESTAMP,
    
    -- Action taken
    action VARCHAR(32),  -- approve, reject, hold, defer, escalate
    action_at TIMESTAMP,
    action_by VARCHAR(256),
    action_reason TEXT,
    
    -- Escalation
    escalated_to VARCHAR(256),
    escalation_reason TEXT,
    escalation_resolved_at TIMESTAMP,
    
    -- Metadata
    context_snapshot JSONB DEFAULT '{}',  -- Run state at gate time for decision context
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_approval_gates_run_uuid ON approval_gates(run_uuid);
CREATE INDEX idx_approval_gates_assigned_to ON approval_gates(assigned_to);
CREATE INDEX idx_approval_gates_gate_status ON approval_gates(gate_status);

-- ============================================================================
-- EVIDENCE TABLE
-- ============================================================================
-- Immutable log of execution evidence: test results, metrics, skill output, logs.

CREATE TABLE evidence (
    id BIGSERIAL PRIMARY KEY,
    evidence_uuid UUID NOT NULL UNIQUE DEFAULT gen_random_uuid(),
    run_uuid UUID NOT NULL REFERENCES runs(run_uuid) ON DELETE CASCADE,
    
    -- Evidence classification
    evidence_type VARCHAR(64) NOT NULL CHECK (evidence_type IN (
        'run_log', 'artifact_validation', 'approval_decision',
        'skill_execution', 'test_result', 'jira_sync_state',
        'orchestrator_decision', 'exception', 'metric'
    )),
    
    -- Content
    evidence_name VARCHAR(512) NOT NULL,
    content_encoding VARCHAR(32) DEFAULT 'utf8',
    content_type VARCHAR(128),
    content_bytes INT,
    storage_uri VARCHAR(1024),
    content_inline JSONB,  -- For structured evidence (test results, metrics)
    
    -- Retention & lifecycle
    retention_class VARCHAR(16) NOT NULL CHECK (retention_class IN (
        'run', 'cross_run', 'defect', 'audit'
    )),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP NOT NULL,
    
    -- Traceability
    source_component VARCHAR(64),  -- orchestrator, skill_runtime, approval_engine, etc.
    correlation_id VARCHAR(256),  -- For grouping related evidence
    
    -- Redaction metadata
    has_pii BOOLEAN DEFAULT false,
    has_secrets BOOLEAN DEFAULT false,
    redaction_applied_at TIMESTAMP
);

CREATE INDEX idx_evidence_run_uuid ON evidence(run_uuid);
CREATE INDEX idx_evidence_type ON evidence(evidence_type);
CREATE INDEX idx_evidence_expires_at ON evidence(expires_at);
CREATE INDEX idx_evidence_correlation_id ON evidence(correlation_id);

-- ============================================================================
-- JIRA_SYNC_STATE TABLE
-- ============================================================================
-- Deterministic tracking of Jira integration state for conflict resolution.

CREATE TABLE jira_sync_state (
    id BIGSERIAL PRIMARY KEY,
    sync_uuid UUID NOT NULL UNIQUE DEFAULT gen_random_uuid(),
    run_uuid UUID NOT NULL REFERENCES runs(run_uuid) ON DELETE CASCADE,
    
    -- External identity
    jira_issue_key VARCHAR(64) NOT NULL,  -- e.g., TEST-123
    jira_issue_id VARCHAR(64),  -- Jira server-assigned numeric ID
    
    -- Sync metadata
    sync_direction VARCHAR(16) NOT NULL CHECK (sync_direction IN (
        'pull', 'push', 'bidirectional'
    )),
    last_sync_ts TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    -- Field mapping state (JSON snapshot of last successful sync)
    field_mapping_state JSONB NOT NULL DEFAULT '{}',
    
    -- Conflict tracking
    conflict_detected BOOLEAN DEFAULT false,
    conflict_type VARCHAR(64),  -- remote_update_after_sync, field_mismatch, etc.
    conflict_resolution_strategy VARCHAR(32) DEFAULT 'remote_wins' CHECK (
        conflict_resolution_strategy IN ('remote_wins', 'local_wins', 'manual_review')
    ),
    conflict_resolved_at TIMESTAMP,
    conflict_notes TEXT,
    
    -- Sync errors (for retry logic)
    last_error_code VARCHAR(64),
    last_error_message TEXT,
    last_error_ts TIMESTAMP,
    retry_count INT DEFAULT 0,
    next_retry_at TIMESTAMP,
    
    -- Metadata
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    UNIQUE(run_uuid, jira_issue_key)
);

CREATE INDEX idx_jira_sync_state_run_uuid ON jira_sync_state(run_uuid);
CREATE INDEX idx_jira_sync_state_issue_key ON jira_sync_state(jira_issue_key);
CREATE INDEX idx_jira_sync_state_next_retry_at ON jira_sync_state(next_retry_at) WHERE next_retry_at IS NOT NULL;

-- ============================================================================
-- CROSS_RUN_MEMORY TABLE
-- ============================================================================
-- Reusable patterns, decision history, learned heuristics from previous runs.

CREATE TABLE cross_run_memory (
    id BIGSERIAL PRIMARY KEY,
    memory_uuid UUID NOT NULL UNIQUE DEFAULT gen_random_uuid(),
    
    -- Memory scope and key
    memory_scope VARCHAR(64) NOT NULL CHECK (memory_scope IN (
        'global', 'project', 'artifact_type', 'skill', 'approver'
    )),
    memory_scope_value VARCHAR(256),  -- e.g., project_id, artifact_type name, skill_name
    memory_key VARCHAR(512) NOT NULL,  -- Semantic key or hash
    
    -- Content
    memory_type VARCHAR(64) NOT NULL CHECK (memory_type IN (
        'pattern', 'decision_history', 'heuristic', 'learned_mapping', 'error_resolution'
    )),
    memory_data JSONB NOT NULL,  -- Structured payload (varies by type)
    
    -- Embeddings for semantic search (optional)
    embedding_vector VECTOR(1536),  -- OpenAI embedding or similar
    
    -- Lifecycle
    first_observed_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    last_used_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usage_count INT DEFAULT 1,
    
    -- Retention
    ttl_seconds INT,  -- NULL = permanent
    expires_at TIMESTAMP,
    
    -- Originating run (for traceability)
    originating_run_uuid UUID REFERENCES runs(run_uuid) ON DELETE SET NULL,
    
    -- Confidence & quality
    confidence_score DECIMAL(3,2) DEFAULT 1.0 CHECK (confidence_score >= 0 AND confidence_score <= 1.0),
    is_validated BOOLEAN DEFAULT false,
    
    UNIQUE(memory_scope, memory_scope_value, memory_key)
);

CREATE INDEX idx_cross_run_memory_scope ON cross_run_memory(memory_scope, memory_scope_value);
CREATE INDEX idx_cross_run_memory_expires_at ON cross_run_memory(expires_at);
CREATE INDEX idx_cross_run_memory_last_used ON cross_run_memory(last_used_at DESC);

-- ============================================================================
-- SKILL_EXECUTION TABLE
-- ============================================================================
-- Execution record for each skill invocation during orchestration.

CREATE TABLE skill_execution (
    id BIGSERIAL PRIMARY KEY,
    execution_uuid UUID NOT NULL UNIQUE DEFAULT gen_random_uuid(),
    run_uuid UUID NOT NULL REFERENCES runs(run_uuid) ON DELETE CASCADE,
    
    -- Skill identity
    skill_name VARCHAR(256) NOT NULL,
    skill_version VARCHAR(32) NOT NULL,
    skill_class VARCHAR(32),  -- http_request, test_execution, validation, etc.
    
    -- Execution state
    execution_status VARCHAR(32) NOT NULL CHECK (execution_status IN (
        'pending', 'running', 'success', 'failed', 'timeout', 'skipped'
    )),
    
    -- Input & output
    input_payload JSONB NOT NULL,
    output_payload JSONB,
    error_message TEXT,
    
    -- Timing
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    duration_ms INT,
    
    -- Retry metadata
    attempt INT DEFAULT 1,
    max_retries INT DEFAULT 0,
    
    -- Traceability
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    retention_class VARCHAR(16) DEFAULT 'run',
    expires_at TIMESTAMP,
    
    correlation_id VARCHAR(256)
);

CREATE INDEX idx_skill_execution_run_uuid ON skill_execution(run_uuid);
CREATE INDEX idx_skill_execution_skill_name ON skill_execution(skill_name);
CREATE INDEX idx_skill_execution_status ON skill_execution(execution_status);

-- ============================================================================
-- AUDIT_LOG TABLE
-- ============================================================================
-- Immutable compliance log of all state transitions and sensitive operations.

CREATE TABLE audit_log (
    id BIGSERIAL PRIMARY KEY,
    audit_log_uuid UUID NOT NULL UNIQUE DEFAULT gen_random_uuid(),
    
    -- Event context
    event_type VARCHAR(64) NOT NULL,  -- run_created, approval_approved, evidence_redacted, etc.
    entity_type VARCHAR(64) NOT NULL,  -- run, artifact, approval_gate, jira_sync_state, etc.
    entity_uuid UUID,
    
    -- Actor (service account, user email, system identity)
    actor_identity VARCHAR(256) NOT NULL,
    actor_type VARCHAR(32) NOT NULL CHECK (actor_type IN ('system', 'user', 'api_token', 'service_account')),
    
    -- State change
    before_state JSONB,
    after_state JSONB,
    change_description TEXT,
    
    -- Metadata
    timestamp TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    ip_address INET,
    user_agent VARCHAR(512),
    
    -- Retention (audit logs are permanent)
    retention_class VARCHAR(16) DEFAULT 'audit'
);

CREATE INDEX idx_audit_log_timestamp ON audit_log(timestamp DESC);
CREATE INDEX idx_audit_log_entity ON audit_log(entity_type, entity_uuid);
CREATE INDEX idx_audit_log_actor ON audit_log(actor_identity);

-- ============================================================================
-- CONFIGURATION TABLE
-- ============================================================================
-- Global configuration, feature flags, SLO targets, retention boundaries.

CREATE TABLE configuration (
    id BIGSERIAL PRIMARY KEY,
    config_key VARCHAR(256) NOT NULL UNIQUE,
    config_value JSONB NOT NULL,
    config_type VARCHAR(32),  -- string, number, boolean, json
    description TEXT,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_by VARCHAR(256)
);

CREATE INDEX idx_configuration_key ON configuration(config_key);

-- Initial configuration seed
INSERT INTO configuration (config_key, config_value, config_type, description)
VALUES
    ('retention_policy.run_days', '{"value": 30}', 'number', 'Run evidence retention in days'),
    ('retention_policy.cross_run_days', '{"value": 90}', 'number', 'Cross-run memory retention in days'),
    ('retention_policy.defect_days', '{"value": 180}', 'number', 'Defect evidence retention in days'),
    ('retention_policy.audit_days', '{"value": 365}', 'number', 'Audit log retention in days'),
    ('approval_timeout_seconds', '{"value": 3600}', 'number', 'Default approval timeout in seconds'),
    ('slo_api_availability_percent', '{"value": 99.5}', 'number', 'API availability SLO target'),
    ('slo_worker_availability_percent', '{"value": 99.5}', 'number', 'Worker availability SLO target'),
    ('max_artifact_size_bytes', '{"value": 104857600}', 'number', 'Max artifact size: 100MB'),
    ('jira_sync_batch_size', '{"value": 50}', 'number', 'Batch size for Jira sync operations'),
    ('enable_cross_run_memory', '{"value": true}', 'boolean', 'Enable cross-run pattern learning')
ON CONFLICT (config_key) DO NOTHING;

```

---

## Migrations

### Alembic-Based Version Control

Create migrations with Alembic:

```bash
# Initialize (one-time)
alembic init alembic

# Create new migration
alembic revision --autogenerate -m "Initial schema"

# Apply migration
alembic upgrade head

# Rollback
alembic downgrade -1
```

### Migration File Template

```python
"""Create base schema for Phase 1.

Revision ID: 001
Revises: 
Create Date: 2026-04-25 00:00:00.000000

"""
from alembic import op
import sqlalchemy as sa
from sqlalchemy.dialects import postgresql

revision = '001'
down_revision = None
branch_labels = None
depends_on = None

def upgrade():
    # Apply migration
    pass

def downgrade():
    # Rollback
    pass
```

---

## Indexes & Performance

### Query Patterns & Index Strategy

| Query Pattern | Index | Cardinality | Rationale |
|---|---|---|---|
| List runs by status | `idx_runs_status` | Medium | Approval dashboard filters |
| Recent runs (dashboard) | `idx_runs_created_at DESC` | High | Temporal queries |
| Jira issue lookup | `idx_runs_external_id` | Low | Deduplicate incoming webhooks |
| Pending approvals | `idx_approval_gates_assigned_to` | Low | Approver dashboard |
| Artifact expiration | `idx_artifacts_expires_at` | Medium | Nightly purge job |
| Evidence correlation | `idx_evidence_correlation_id` | Medium | Cross-evidence queries |
| Jira sync retry queue | `idx_jira_sync_state_next_retry_at` | Low | Retry scheduler |

### Index Maintenance

```sql
-- Weekly ANALYZE to update planner statistics
ANALYZE;

-- Monthly VACUUM to reclaim space and prune dead tuples
VACUUM ANALYZE;

-- Monitor index bloat
SELECT schemaname, tablename, indexname, idx_size
FROM pg_indexes
ORDER BY idx_size DESC;
```

---

## Retention & Purging

### Automated Purge Job

```python
# scheduled_tasks/purge_expired_evidence.py

import asyncio
from datetime import datetime
import sqlalchemy as sa
from app.db import database, engine

async def purge_expired_evidence():
    """Delete evidence records where expires_at <= now()."""
    query = sa.delete(evidence).where(
        evidence.c.expires_at <= sa.func.now()
    )
    result = await database.execute(query)
    logger.info(f"Purged {result.rowcount} expired evidence records")

# Cron: Run nightly at 02:00 UTC
```

### Retention Policy Configuration

| Category | Default TTL | Configurable | Use Case |
|---|---|---|---|
| run | 30 days | ✅ | Run logs, transient artifacts |
| cross_run | 90 days | ✅ | Memory patterns, decision history |
| defect | 180 days | ✅ | Root cause analysis, defect trends |
| audit | 365 days | ✅ | Compliance, forensics, legal holds |

### Legal Hold & Override

```sql
-- Flag record for permanent retention (legal hold)
ALTER TABLE evidence ADD COLUMN legal_hold BOOLEAN DEFAULT false;
CREATE INDEX idx_evidence_legal_hold ON evidence(legal_hold) WHERE legal_hold = true;

-- Purge job respects legal hold
-- DELETE FROM evidence WHERE expires_at <= now() AND legal_hold = false;
```

---

## Backup & Recovery

### Backup Strategy

**Daily Full Backup** (02:00 UTC):
```bash
pg_dump -Fc -U orqestly orqestly > backup-$(date +%Y%m%d).dump
```

**Point-in-Time Recovery (PITR)**:
```bash
# PostgreSQL WAL archiving to S3
wal_level = replica
archive_mode = on
archive_command = 'aws s3 cp %p s3://orqestly-backups/wal/%f'
```

**Restore from Backup**:
```bash
# Full restore
pg_restore -Fc -U orqestly -d orqestly backup-20260425.dump

# PITR to specific timestamp
pg_basebackup -h localhost -U orqestly -D /path/to/recovery \
  -t '2026-04-25 14:30:00 UTC'
```

### Recovery Objectives

| Metric | Target | SLO |
|---|---|---|
| RTO (Recovery Time Objective) | < 15 minutes | Part of 99.5% API availability |
| RPO (Recovery Point Objective) | < 5 minutes | WAL archiving frequency |
| Backup Retention | 30 days + PITR 7 days | Regulatory compliance |

---

## References

- **TECHNICAL_SPECIFICATION.md** — Data model contracts
- **ARCHITECTURE.md** — System flow and component boundaries
- **EVIDENCE_RETENTION_POLICY.md** — Retention class definitions and purge rules
- **SLO_AND_ALERTING.md** — Availability targets and incident response SLOs
- **SECURITY_OPERATIONS_BASELINE.md** — Database encryption and access control

