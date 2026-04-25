# Failover & High Availability Specification

**Version**: 1.0  
**Last Updated**: April 2026  
**Owner**: Platform Engineering  
**Status**: Locked for Phase 1

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Phase 1 Baseline (Single VPS)](#phase-1-baseline-single-vps)
3. [Failover Procedures](#failover-procedures)
4. [Phase 2 High Availability Architecture](#phase-2-high-availability-architecture)
5. [Disaster Recovery](#disaster-recovery)
6. [Testing & Validation](#testing--validation)
7. [Escalation & Runbooks](#escalation--runbooks)

---

## Executive Summary

**Phase 1 Deployment Model**: Single VPS with local persistence (infra-postgres, infra-redis, edge-nginx).

**RTO/RPO Targets**:
- **RTO (Recovery Time Objective)**: < 15 minutes
- **RPO (Recovery Point Objective)**: < 5 minutes
- **Uptime SLO**: 99.5% monthly availability

**Phase 1 Limitations**:
- No multi-region replication
- No active-active clustering
- Manual failover to secondary VPS (if deployed)
- Single point of failure: VPS hardware/network

**Phase 2 Roadmap** (Post-MVP):
- Multi-zone PostgreSQL replicas
- Redis Sentinel for automatic failover
- Load balancer across multiple API nodes
- Geo-distributed edge caches
- Automated disaster recovery scripts

---

## Phase 1 Baseline (Single VPS)

### Current Infrastructure

```
┌─────────────────────────────────────────┐
│         Single VPS (infra-*)            │
├─────────────────────────────────────────┤
│ ┌─────────────────────────────────────┐ │
│ │   edge-nginx (reverse proxy)        │ │
│ │   Port 443 (TLS)                    │ │
│ └─────────────────────────────────────┘ │
│            ↓                             │
│ ┌─────────────────────────────────────┐ │
│ │   Docker Compose                    │ │
│ │  ┌──────────────┐ ┌──────────────┐ │ │
│ │  │ orqestly-api │ │ orqestly-wrk │ │ │
│ │  │  :8000       │ │              │ │ │
│ │  └──────────────┘ └──────────────┘ │ │
│ │  (6 replicas via FastAPI + Uvicorn) │ │
│ │  (4 worker processes via Celery)    │ │
│ └─────────────────────────────────────┘ │
│            ↓          ↓                  │
│ ┌──────────────────────────────────────┐ │
│ │  infra-postgres:5432                 │ │
│ │  (Single PostgreSQL 17 instance)     │ │
│ └──────────────────────────────────────┘ │
│            ↓                             │
│ ┌──────────────────────────────────────┐ │
│ │  infra-redis:6379                    │ │
│ │  (Single Redis instance)             │ │
│ │  (Celery broker + cache)             │ │
│ └──────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

### Availability Assumptions

| Component | Availability | Notes |
|---|---|---|
| VPS Host | 99.9% | GCP/AWS/Azure SLA |
| edge-nginx | 99.95% | Stateless, auto-restart |
| PostgreSQL | 99.95% | Local instance, single replica backups |
| Redis | 99.9% | In-memory, recoverable from AOF |
| API Service | 99.5% | Target SLO (health checks + auto-restart) |
| Worker Service | 99.5% | Target SLO (dead letter queue + replay) |

**Composite Availability** (Phase 1): 99.5% × 99.5% × 99.9% ≈ **99.4%** monthly uptime

---

## Failover Procedures

### Scenario 1: API Service Crash

**Detection**:
- edge-nginx health check fails (http://orqestly-api:8000/health)
- Kubernetes liveness probe OR manual monitoring alert

**Recovery** (Automated):
```bash
# If using Docker Compose with health checks
docker-compose up -d orqestly-api  # Restart container
```

**Recovery** (Manual):
```bash
# SSH to VPS
ssh admin@infra-prod

# Check container status
docker ps | grep orqestly-api

# Restart if needed
docker-compose restart orqestly-api

# Verify
curl http://localhost:8000/health
```

**Expected Recovery Time**: < 2 minutes

**Checkpoint**: Verify no in-flight requests lost (Celery tasks should be requeued)

---

### Scenario 2: Worker Service Hung or Crashed

**Detection**:
- Celery task queue depth increases (unprocessed tasks accumulate)
- Worker heartbeat missing from Redis
- Alert: `worker_task_queue_depth > threshold` for > 5 minutes

**Recovery** (Automated):
```bash
docker-compose restart orqestly-worker
```

**Recovery** (Manual):
```bash
# SSH to VPS
ssh admin@infra-prod

# Check Celery stats
docker-compose exec orqestly-worker celery -A app.tasks inspect active

# Restart workers
docker-compose restart orqestly-worker

# Inspect queue for stuck tasks
docker-compose exec redis redis-cli LLEN celery
```

**Expected Recovery Time**: < 3 minutes

**Checkpoint**: Verify dead-letter queue is empty or < 5 messages; check evidence_retention_policy for recovery bounds

---

### Scenario 3: PostgreSQL Connection Pool Exhaustion

**Detection**:
- API logs: `psycopg2.pool.PoolError: connection pool exhausted`
- Alert: `db_connection_pool_available < 5` for > 2 minutes

**Immediate Mitigation** (No data loss):
```bash
# SSH to VPS
ssh admin@infra-prod

# Restart API service (will recycle connections)
docker-compose restart orqestly-api

# If database is unreachable, see Scenario 4
```

**Preventive Action**:
- Increase connection pool size in docker-compose.yaml (SQLALCHEMY_POOL_SIZE)
- Review and optimize long-running queries (check OPERATIONS_RUNBOOK.md)
- Implement connection timeout and recycling policy

**Expected Recovery Time**: < 2 minutes

---

### Scenario 4: PostgreSQL Data Corruption or Disk Full

**Detection**:
- Application logs: `ERROR: no space left on device` OR integrity check failures
- `df -h` shows disk > 95% utilization
- Alert: `disk_usage_percent > 90%` for any data directory

**Recovery** (No Full Restore Needed):
1. **Emergency Purge**: Delete oldest evidence records (preserves active runs)
   ```bash
   docker-compose exec orqestly-api python -c "
   from app.tasks import purge_expired_evidence
   asyncio.run(purge_expired_evidence())
   "
   ```

2. **Verify Database Health**:
   ```bash
   docker-compose exec infra-postgres psql -U orqestly -d orqestly -c "VACUUM ANALYZE;"
   docker-compose exec infra-postgres psql -U orqestly -d orqestly -c "SELECT pg_database.datname, pg_size_pretty(pg_database_size(pg_database.datname)) FROM pg_database;"
   ```

3. **If Disk Still Critical**: Restore from backup (see Disaster Recovery)

**Expected Recovery Time**: 5-15 minutes (depending on corruption severity)

---

### Scenario 5: Redis Memory Pressure or Eviction

**Detection**:
- Redis logs: `WARNING: Memory usage is > maxmemory limit`
- Celery tasks rejected: `OOM: unable to append to AOF`
- Alert: `redis_memory_used_percent > 85%` for > 2 minutes

**Immediate Mitigation** (Tasks preserved in database):
```bash
# SSH to VPS
ssh admin@infra-prod

# Check Redis stats
docker-compose exec redis redis-cli INFO memory

# Force eviction of least-used keys (LRU policy already set)
docker-compose exec redis redis-cli CONFIG SET maxmemory-policy allkeys-lru

# Restart workers to purge in-memory caches
docker-compose restart orqestly-worker orqestly-api
```

**Preventive Action**:
- Increase Redis memory in docker-compose.yaml
- Implement Celery result backend in PostgreSQL (not Redis)
- Reduce cache TTL for low-importance data

**Expected Recovery Time**: < 3 minutes

---

### Scenario 6: Network Partition (VPS Isolation)

**Detection**:
- Network packets timeout/lost (monitored via edge-nginx or icmp from external host)
- Jira webhook deliveries fail (retries exhaust)
- Alert: `network_latency_p99 > 5000ms` for > 1 minute OR external connectivity lost

**Recovery** (Automatic at VPS level):
- VPS provider's network fabric automatically recovers
- Services continue running locally; inbound requests timeout

**Recovery** (Manual intervention if needed):
```bash
# From your local machine, SSH may fail; if VPS has secondary network:
ssh -i key admin@infra-prod-secondary

# Check network interfaces
ip link show
ip route show

# Restart network service (OS-dependent)
sudo systemctl restart networking  # Linux
```

**Expected Recovery Time**: 5-10 minutes (provider SLA depends)

**Checkpoint**: Once recovered, retry Jira sync failures and retry Slack notifications

---

## Phase 2 High Availability Architecture

### Multi-Zone PostgreSQL Replication

```
┌─────────────────────────────────────────────────────┐
│         Multi-Zone HA Deployment (Phase 2)          │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Zone A (Primary)     Zone B (Standby)              │
│  ┌───────────────┐    ┌───────────────┐            │
│  │ PostgreSQL    │    │ PostgreSQL    │            │
│  │ (Primary)     │───→│ (Hot Standby) │            │
│  │ Port 5432     │    │ Streaming Rep │            │
│  └───────────────┘    └───────────────┘            │
│       ↑                       ↑                     │
│       └───────────────────────┘                    │
│       WAL Archive to S3 (PITR)                     │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │      Cloud Load Balancer                     │  │
│  │  (Route to Primary, Monitor failover)        │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  API Layer (Multi-zone, Auto-scale)                │
│  ┌──────────────┐    ┌──────────────┐             │
│  │ orqestly-api │    │ orqestly-api │             │
│  │ Zone A       │    │ Zone B       │             │
│  └──────────────┘    └──────────────┘             │
│       (+ Zone C)                                    │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │      Redis Sentinel (3-node cluster)         │  │
│  │  (Auto-failover, master detection)           │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### HA Configuration (Post-Phase-1)

**PostgreSQL**:
- Streaming replication to standby in different zone
- Automated failover via patroni or WAL-G
- RTO: < 5 minutes, RPO: < 1 minute

**Redis Sentinel**:
- 3-node Redis Sentinel cluster
- Automatic master/replica promotion
- Celery workers auto-discover new master

**API Layer**:
- Stateless Uvicorn instances across zones
- Load balancer health checks every 5 seconds
- Auto-scaling based on request rate

**Celery Workers**:
- Distributed across zones
- Dead-letter queue for failed tasks
- Task replay on worker restart

---

## Disaster Recovery

### Full VPS Loss Scenario

**Objective**: Restore all services to new VPS within RTO < 15 minutes

**Prerequisites** (Must be in place):
- ✅ Daily database backups stored in S3 (separate from VPS)
- ✅ WAL archives for PITR
- ✅ Infrastructure-as-code (Terraform/CloudFormation) for rapid VPS provisioning
- ✅ Docker images stored in container registry (ECR/GCR)
- ✅ Configuration stored in secrets manager (AWS Secrets Manager / Azure Key Vault)

### Recovery Procedure (Automated)

```bash
#!/bin/bash
# disaster_recovery/restore_from_backup.sh

set -e

TARGET_VPS_IP=$1
BACKUP_DATE=$2  # YYYYMMDD format

echo "Starting disaster recovery for VPS ${TARGET_VPS_IP}"

# 1. Provision new VPS (assumes IaC via Terraform)
echo "Step 1: Provisioning new VPS..."
terraform apply -var="vps_ip=${TARGET_VPS_IP}"

# 2. Restore PostgreSQL from backup
echo "Step 2: Restoring PostgreSQL..."
aws s3 cp "s3://orqestly-backups/backup-${BACKUP_DATE}.dump" /tmp/
pg_restore -Fc -U orqestly -d orqestly /tmp/backup-${BACKUP_DATE}.dump

# 3. Restore Redis state (AOF)
echo "Step 3: Restoring Redis..."
aws s3 cp "s3://orqestly-backups/redis-aof-${BACKUP_DATE}" /var/lib/redis/appendonly.aof
systemctl restart redis

# 4. Deploy latest Docker images
echo "Step 4: Deploying services..."
docker-compose pull
docker-compose up -d

# 5. Verify health
echo "Step 5: Verifying health..."
sleep 10
curl -f http://localhost:8000/health || exit 1

echo "Disaster recovery completed. Services online."
```

### PITR (Point-in-Time Recovery)

If a more recent backup is needed:

```bash
# Restore to specific timestamp (e.g., before data corruption)
pg_basebackup -h localhost -U orqestly \
  -D /var/lib/postgresql/recovery_wals \
  -t '2026-04-25 14:30:00 UTC'
```

---

## Testing & Validation

### Monthly Failover Drill

**Purpose**: Ensure all procedures are current and team can execute under time pressure

**Schedule**: First Thursday of each month, 14:00-15:30 UTC

**Scenarios** (Rotate monthly):
1. API service crash + restart
2. PostgreSQL connection failure + recovery
3. Redis memory pressure + eviction
4. Full backup + PITR restore
5. Disk full + emergency purge

**Validation Checklist**:
- [ ] Services return to healthy state within RTO < 15 min
- [ ] No data loss (verify evidence retention)
- [ ] Approval gates resume processing
- [ ] Jira sync queue drains within 30 min
- [ ] All dashboards display current metrics
- [ ] Audit log captures incident and recovery

**Incident Commander**: Platform Engineering lead  
**Participant Roles**: SRE, Database Admin, Application Support

### Synthetic Monitoring

```python
# monitoring/synthetic_tests.py

import httpx
import asyncio
from datetime import datetime

async def synthetic_check():
    """Periodic end-to-end health check."""
    
    # 1. API availability
    try:
        async with httpx.AsyncClient() as client:
            resp = await client.get("https://orqestly.internal/health", timeout=5)
            assert resp.status_code == 200
    except Exception as e:
        alert(f"API health check failed: {e}")
    
    # 2. Database connectivity
    async with database.connection() as conn:
        result = await conn.execute("SELECT 1")
        assert result is not None
    
    # 3. Redis connectivity
    try:
        redis.ping()
    except Exception as e:
        alert(f"Redis health check failed: {e}")
    
    # 4. Jira integration
    try:
        jira_client.search_issues(jql="project = TEST LIMIT 1")
    except Exception as e:
        alert(f"Jira integration health check failed: {e}")

# Run every 5 minutes
scheduler.schedule_job(synthetic_check, 'interval', minutes=5)
```

---

## Escalation & Runbooks

### Escalation Path

| Severity | Condition | Primary | Backup | Timeline |
|---|---|---|---|---|
| **Critical** | Zero uptime OR data loss risk | SRE on-call | Platform Lead | Page immediately |
| **Major** | > 20% request latency increase | SRE on-call | Eng Manager | Alert within 5 min |
| **Minor** | Degradation < 10% OR non-customer-facing | NOC monitoring | SRE | Alert within 15 min |

### Quick Reference Runbooks

See **OPERATIONS_RUNBOOK.md** for:
- API service restart procedure
- Database recovery from corruption
- Evidence purge (emergency disk free)
- Jira sync failure recovery
- Redis memory pressure mitigation
- Network partition recovery

See **RUNBOOK_CHECKLISTS.md** for:
- Pre-deployment failover test checklist
- Post-incident review checklist
- Rollback procedure checklist

---

## References

- **OPERATIONS_RUNBOOK.md** — Detailed recovery procedures
- **SLO_AND_ALERTING.md** — SLO targets and alert thresholds
- **DEPLOYMENT_BASELINE.md** — Infrastructure topology
- **docker-compose.yaml** — Service configuration and health checks
- **EVIDENCE_RETENTION_POLICY.md** — Data retention for recovery bounds

