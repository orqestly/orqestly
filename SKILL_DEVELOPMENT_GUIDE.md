# Skill Development Guide

**Version**: 1.0  
**Last Updated**: April 2026  
**Owner**: Platform Engineering  
**Status**: Locked for Phase 1

---

## Table of Contents

1. [Overview](#overview)
2. [Skill Architecture](#skill-architecture)
3. [Base Skill Interface](#base-skill-interface)
4. [Built-In Skills](#built-in-skills)
5. [Custom Skill Development](#custom-skill-development)
6. [Skill Lifecycle](#skill-lifecycle)
7. [Testing & Validation](#testing--validation)
8. [Packaging & Deployment](#packaging--deployment)
9. [Skill Registry](#skill-registry)

---

## Overview

Skills are extensible, composable execution units that Orqestly invokes during the **execute** phase of the orchestration lifecycle. They receive parsed artifacts and execution context, perform domain-specific operations (API calls, test execution, validation), and return standardized results.

### Skill Responsibilities

1. **Accept Input**: Receive task payload with parsed artifacts and execution parameters
2. **Execute**: Perform work in isolation (no side effects on Orqestly state)
3. **Return Results**: Structured output with success/failure status and evidence
4. **Report Metrics**: Execution duration, resource usage, retry behavior
5. **Handle Failures**: Graceful degradation with clear error classification

### Skill Benefits

- **Decoupled Architecture**: Skills are independent; system remains functional if skill fails
- **Horizontal Scaling**: Each skill type can scale independently via Celery
- **Versioning**: Skills can coexist (v1, v2) during backward-compatibility transitions
- **Auditability**: All skill invocations logged with inputs/outputs for forensics
- **Reusability**: Common skills (http_request, validation) available across all runs

---

## Skill Architecture

### Execution Flow

```
Orchestrator
    │
    ├─ Parse Artifacts
    │
    ├─ Generate Task Plan
    │  (Task 1: http_request to API endpoint)
    │  (Task 2: validate_response against schema)
    │  (Task 3: extract_metrics from response)
    │
    ├─ Dispatch to Celery
    │  │
    │  ├─ Skill Runtime (Worker Process)
    │  │  │
    │  │  ├─ Load Skill: http_request
    │  │  │
    │  │  ├─ Validate Input
    │  │  │  (Check required fields, type constraints)
    │  │  │
    │  │  ├─ Execute Skill
    │  │  │  (Send HTTP request, parse response)
    │  │  │
    │  │  ├─ Capture Output
    │  │  │  (Response body, status code, latency)
    │  │  │
    │  │  ├─ Classify Result
    │  │  │  (success / timeout / invalid_schema / etc.)
    │  │  │
    │  │  └─ Return to Orchestrator
    │  │     {
    │  │       "status": "success",
    │  │       "output": {...},
    │  │       "execution_duration_ms": 245,
    │  │       "evidence": [...]
    │  │     }
    │  │
    │  └─ (Repeat for tasks 2, 3)
    │
    ├─ Aggregate Results
    │
    ├─ Store Evidence
    │
    └─ Proceed to Review/Approve Phase
```

### Skill Execution Context

```python
@dataclass
class SkillExecutionContext:
    """Context passed to every skill invocation."""
    
    run_uuid: UUID
    execution_uuid: UUID
    
    # Ingested artifacts (parsed and validated)
    artifacts: Dict[str, Artifact]  # {artifact_name: Artifact}
    
    # Configuration and cross-run patterns
    config: Dict[str, Any]  # Global configuration
    memory: Dict[str, Any]  # Cross-run memory (if relevant)
    
    # Service clients and connections
    jira_client: JiraClient
    logger: Logger
    metrics_collector: MetricsCollector
    
    # Execution metadata
    retry_count: int
    max_retries: int
    timeout_seconds: int
```

---

## Base Skill Interface

### Python Skill Template

```python
# custom_skills/my_skill.py

from typing import Any, Dict, List
from pydantic import BaseModel, Field
from app.skill_runtime import BaseSkill, SkillResult, SkillExecutionContext
import logging

logger = logging.getLogger(__name__)

class MySkillInput(BaseModel):
    """Validate and document input schema."""
    
    param1: str = Field(..., description="Required parameter")
    param2: int = Field(default=10, description="Optional parameter with default")
    timeout_ms: int = Field(default=5000, description="Execution timeout")

class MySkillOutput(BaseModel):
    """Validate and document output schema."""
    
    result: str
    execution_time_ms: int
    metadata: Dict[str, Any] = {}

class MySkill(BaseSkill):
    """
    Custom skill implementation.
    
    Description:
        Brief explanation of what this skill does.
    
    Example Input:
        {
            "param1": "value",
            "param2": 20
        }
    
    Example Output:
        {
            "status": "success",
            "output": {
                "result": "output_value",
                "execution_time_ms": 145,
                "metadata": {...}
            },
            "evidence": [...]
        }
    """
    
    name = "my_skill"
    version = "1.0.0"
    skill_class = "custom"  # http_request, validation, test_execution, custom
    required_capabilities = []  # Infra requirements (e.g., ["browser", "network"])
    
    def validate_input(self, input_data: Dict[str, Any]) -> MySkillInput:
        """
        Validate and type-cast input.
        
        Raises:
            ValueError: If input violates schema
        """
        return MySkillInput(**input_data)
    
    async def execute(
        self,
        context: SkillExecutionContext,
        input_data: MySkillInput
    ) -> SkillResult:
        """
        Execute skill logic.
        
        Args:
            context: Execution context with artifacts, config, clients
            input_data: Validated input
        
        Returns:
            SkillResult with status, output, evidence, metrics
        """
        
        try:
            logger.info(f"Executing {self.name} with param1={input_data.param1}")
            
            # 1. Perform work
            start_time = time.time()
            output = await self._do_work(context, input_data)
            execution_time_ms = int((time.time() - start_time) * 1000)
            
            # 2. Validate output schema
            typed_output = MySkillOutput(
                result=output["result"],
                execution_time_ms=execution_time_ms,
                metadata=output.get("metadata", {})
            )
            
            # 3. Capture evidence
            evidence = [
                {
                    "type": "skill_execution_output",
                    "data": typed_output.dict()
                }
            ]
            
            # 4. Return success
            return SkillResult(
                status="success",
                output=typed_output.dict(),
                evidence=evidence,
                execution_duration_ms=execution_time_ms,
                retry_eligible=False
            )
        
        except TimeoutError as e:
            logger.error(f"Timeout in {self.name}: {e}")
            return SkillResult(
                status="timeout",
                error_message=str(e),
                execution_duration_ms=input_data.timeout_ms,
                retry_eligible=True
            )
        
        except Exception as e:
            logger.exception(f"Unexpected error in {self.name}: {e}")
            return SkillResult(
                status="failed",
                error_message=str(e),
                error_classification="product_defect",
                retry_eligible=True,
                execution_duration_ms=0
            )
    
    async def _do_work(
        self,
        context: SkillExecutionContext,
        input_data: MySkillInput
    ) -> Dict[str, Any]:
        """Implement core skill logic here."""
        # Example: Call API, validate response, return result
        return {
            "result": f"Processed {input_data.param1}",
            "metadata": {"processed_at": datetime.now().isoformat()}
        }
    
    @staticmethod
    def get_schema() -> Dict[str, Any]:
        """Export JSON schema for API documentation."""
        return {
            "input": MySkillInput.schema(),
            "output": MySkillOutput.schema()
        }
```

### SkillResult Contract

```python
@dataclass
class SkillResult:
    """Standardized result returned by all skills."""
    
    # Execution status
    status: str  # success, failed, timeout, skipped, invalid_input
    
    # Output (if success)
    output: Dict[str, Any] = None
    
    # Error (if failed)
    error_message: str = None
    error_classification: str = None  # See ERROR_CLASSIFICATION.md
    
    # Evidence for audit trail
    evidence: List[Dict[str, Any]] = field(default_factory=list)
    
    # Metrics
    execution_duration_ms: int = 0
    retry_eligible: bool = False
    
    # Cross-run learning
    memory_to_store: Dict[str, Any] = None  # Patterns for future runs
```

---

## Built-In Skills

### 1. http_request (Core Skill)

**Purpose**: Execute HTTP requests against API endpoints

**Input Schema**:
```python
class HttpRequestInput(BaseModel):
    method: str  # GET, POST, PUT, DELETE, PATCH
    url: str
    headers: Dict[str, str] = {}
    query_params: Dict[str, Any] = {}
    body: Optional[Union[Dict, str]] = None
    timeout_ms: int = 5000
    verify_ssl: bool = True
    follow_redirects: bool = False
    max_retries: int = 3
```

**Output Schema**:
```python
class HttpRequestOutput(BaseModel):
    status_code: int
    headers: Dict[str, str]
    body: Union[Dict, str]
    latency_ms: int
    error: Optional[str] = None
```

**Capabilities**:
- ✅ Connection pooling (reuse across requests)
- ✅ Automatic retry with exponential backoff
- ✅ SSL/TLS certificate verification
- ✅ Proxy support
- ✅ Request/response logging (sanitized secrets)
- ✅ Timeout enforcement

**Example Task**:
```yaml
- skill_name: http_request
  skill_version: "1.0"
  input:
    method: POST
    url: "https://api.example.com/v1/jobs"
    headers:
      Authorization: "Bearer ${JIRA_API_TOKEN}"
      Content-Type: "application/json"
    body:
      project: "TEST"
      summary: "Automated run results"
```

---

### 2. validate_against_schema (Core Skill)

**Purpose**: Validate structured data (JSON, YAML, OpenAPI) against a schema

**Input Schema**:
```python
class ValidateAgainstSchemaInput(BaseModel):
    schema_format: str  # json_schema, openapi, graphql, xml_schema
    schema_content: Union[Dict, str]  # Schema definition
    data_to_validate: Union[Dict, str]  # Data under test
    strict_mode: bool = True  # Reject unknown properties
```

**Output Schema**:
```python
class ValidateAgainstSchemaOutput(BaseModel):
    is_valid: bool
    violations: List[Dict[str, Any]]  # List of validation errors
    warnings: List[Dict[str, Any]] = []
```

**Capabilities**:
- ✅ JSON Schema validation (draft 7+)
- ✅ OpenAPI schema validation
- ✅ GraphQL schema validation
- ✅ Custom validation rules (Jinja2 expressions)

---

### 3. extract_from_artifact (Core Skill)

**Purpose**: Extract structured data from semi-structured artifacts

**Input Schema**:
```python
class ExtractFromArtifactInput(BaseModel):
    artifact_name: str  # Reference to ingested artifact
    extraction_rules: List[Dict[str, Any]]  # JSONPath, regex, XPath
    output_format: str  # json, csv, markdown
```

**Output Schema**:
```python
class ExtractFromArtifactOutput(BaseModel):
    extracted_data: Dict[str, Any]
    extraction_coverage_percent: int  # % of rules that matched
```

---

## Custom Skill Development

### Step-by-Step: Create "validate_performance_metrics" Skill

**Goal**: Validate that API response times meet SLO targets

**1. Define Input/Output Schemas**

```python
# custom_skills/validate_performance_metrics.py

from pydantic import BaseModel, Field

class ValidatePerformanceMetricsInput(BaseModel):
    test_results_artifact: str = Field(
        ..., 
        description="Name of artifact containing HTTP response times"
    )
    slo_p50_ms: int = Field(default=100, description="SLO target for P50 latency")
    slo_p99_ms: int = Field(default=500, description="SLO target for P99 latency")
    pass_threshold_percent: int = Field(
        default=95, 
        description="% of requests that must meet SLO"
    )

class ValidatePerformanceMetricsOutput(BaseModel):
    passed: bool
    p50_ms: float
    p99_ms: float
    p95_ms: float
    pass_percent: float
    violations: List[Dict[str, Any]] = []
```

**2. Implement Skill Class**

```python
class ValidatePerformanceMetrics(BaseSkill):
    name = "validate_performance_metrics"
    version = "1.0.0"
    skill_class = "validation"
    
    async def execute(
        self,
        context: SkillExecutionContext,
        input_data: ValidatePerformanceMetricsInput
    ) -> SkillResult:
        try:
            # Retrieve artifact
            artifact = context.artifacts.get(input_data.test_results_artifact)
            if not artifact:
                return SkillResult(
                    status="failed",
                    error_message=f"Artifact not found: {input_data.test_results_artifact}",
                    error_classification="product_defect"
                )
            
            # Parse test results
            import json
            results = json.loads(artifact.content_inline.decode())
            latencies = [r["latency_ms"] for r in results.get("requests", [])]
            
            if not latencies:
                return SkillResult(
                    status="failed",
                    error_message="No latency data in test results",
                    error_classification="product_defect"
                )
            
            # Calculate percentiles
            import statistics
            latencies_sorted = sorted(latencies)
            p50 = statistics.median(latencies_sorted)
            p95 = latencies_sorted[int(len(latencies_sorted) * 0.95)]
            p99 = latencies_sorted[int(len(latencies_sorted) * 0.99)]
            
            # Check SLO compliance
            pass_count = sum(1 for l in latencies if l <= input_data.slo_p99_ms)
            pass_percent = (pass_count / len(latencies)) * 100
            
            passed = (
                pass_percent >= input_data.pass_threshold_percent
                and p50 <= input_data.slo_p50_ms
            )
            
            # Build output
            output = ValidatePerformanceMetricsOutput(
                passed=passed,
                p50_ms=p50,
                p99_ms=p99,
                p95_ms=p95,
                pass_percent=pass_percent,
                violations=[
                    {
                        "type": "slo_violation",
                        "metric": "p99_latency",
                        "threshold_ms": input_data.slo_p99_ms,
                        "actual_ms": p99
                    }
                ] if p99 > input_data.slo_p99_ms else []
            )
            
            return SkillResult(
                status="success",
                output=output.dict(),
                execution_duration_ms=int(
                    (time.time() - start_time) * 1000
                )
            )
        
        except Exception as e:
            return SkillResult(
                status="failed",
                error_message=str(e),
                error_classification="product_defect"
            )
```

**3. Register Skill**

```python
# app/skill_registry.py

SKILL_REGISTRY = {
    "http_request": HttpRequestSkill(),
    "validate_against_schema": ValidateAgainstSchemaSkill(),
    "validate_performance_metrics": ValidatePerformanceMetrics(),
}

def get_skill(skill_name: str, skill_version: str):
    skill = SKILL_REGISTRY.get(skill_name)
    if not skill:
        raise ValueError(f"Unknown skill: {skill_name}")
    if skill.version != skill_version:
        raise ValueError(
            f"Skill version mismatch: {skill_name} "
            f"(requested {skill_version}, available {skill.version})"
        )
    return skill
```

**4. Use in Task Plan**

```yaml
# In task plan generated by planner:
- task_id: validate_perf
  skill_name: validate_performance_metrics
  skill_version: "1.0.0"
  input:
    test_results_artifact: "api_load_test_results.json"
    slo_p50_ms: 100
    slo_p99_ms: 500
    pass_threshold_percent: 95
  dependencies: ["execute_load_test"]
```

---

## Skill Lifecycle

### 1. Development

- Write skill class extending `BaseSkill`
- Define input/output schemas with Pydantic
- Implement `execute()` method
- Add unit tests (see Testing & Validation)

### 2. Testing

- Unit tests in `tests/skills/test_my_skill.py`
- Integration tests in staging environment
- Load tests to verify scaling behavior

### 3. Registration

- Add to SKILL_REGISTRY in `app/skill_registry.py`
- Document in this guide

### 4. Deployment

- Add to Docker image in CI/CD pipeline
- Deploy via docker-compose pull + restart
- No downtime (workers pick up new skill on restart)

### 5. Versioning

- Semantic versioning: MAJOR.MINOR.PATCH
- Breaking changes = MAJOR version bump
- Old versions coexist during transition period

### 6. Deprecation

- Mark old version as deprecated in registry
- Log warning when deprecated skill invoked
- Remove after 2 release cycles

---

## Testing & Validation

### Unit Test Template

```python
# tests/skills/test_validate_performance_metrics.py

import pytest
from app.skills import ValidatePerformanceMetrics, SkillExecutionContext
from unittest.mock import MagicMock
import json

@pytest.fixture
def mock_context():
    context = MagicMock(spec=SkillExecutionContext)
    context.artifacts = {}
    return context

@pytest.mark.asyncio
async def test_validate_performance_metrics_pass(mock_context):
    """Test skill returns success when SLO met."""
    
    # Setup artifact with latency data
    artifact = MagicMock()
    artifact.content_inline = json.dumps({
        "requests": [
            {"latency_ms": 50},
            {"latency_ms": 75},
            {"latency_ms": 100},
            {"latency_ms": 125},
            {"latency_ms": 150},
        ]
    }).encode()
    mock_context.artifacts["test_results"] = artifact
    
    # Execute skill
    skill = ValidatePerformanceMetrics()
    result = await skill.execute(
        mock_context,
        {
            "test_results_artifact": "test_results",
            "slo_p50_ms": 100,
            "slo_p99_ms": 200,
            "pass_threshold_percent": 90
        }
    )
    
    # Assert
    assert result.status == "success"
    assert result.output["passed"] == True
    assert result.output["p50_ms"] == 100

@pytest.mark.asyncio
async def test_validate_performance_metrics_fail(mock_context):
    """Test skill returns failure when SLO violated."""
    # Similar setup but with latency data that violates SLO
    ...

@pytest.mark.asyncio
async def test_validate_performance_metrics_missing_artifact(mock_context):
    """Test skill handles missing artifact gracefully."""
    skill = ValidatePerformanceMetrics()
    result = await skill.execute(
        mock_context,
        {
            "test_results_artifact": "nonexistent",
            "slo_p50_ms": 100,
            "slo_p99_ms": 200,
            "pass_threshold_percent": 90
        }
    )
    assert result.status == "failed"
    assert "not found" in result.error_message
```

### Integration Test

```bash
# Run against staging environment
pytest tests/skills/integration/test_validate_performance_metrics.py -v -s

# Run load test
locust -f tests/skills/load/locustfile.py --headless -u 100 -r 10 -t 5m
```

---

## Packaging & Deployment

### Docker Image Build

Ensure Dockerfile includes new skill:

```dockerfile
FROM python:3.12-slim

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY app/ /app/
COPY custom_skills/ /app/custom_skills/

WORKDIR /app
CMD ["celery", "-A", "app.tasks", "worker"]
```

### Deployment Steps

```bash
# 1. Build Docker image
docker build -t orqestly:latest .

# 2. Push to registry
docker push <registry>/orqestly:latest

# 3. Update docker-compose.yaml with new image
version: '3.8'
services:
  orqestly-api:
    image: <registry>/orqestly:latest
    ...
  orqestly-worker:
    image: <registry>/orqestly:latest
    ...

# 4. Deploy with zero downtime
docker-compose pull
docker-compose up -d orqestly-worker  # Restart workers first
docker-compose up -d orqestly-api     # Then API
```

---

## Skill Registry

### Phase 1 Skills (Locked)

| Skill | Version | Status | Owner | Priority |
|---|---|---|---|---|
| http_request | 1.0.0 | ✅ Stable | Platform Eng | Required |
| validate_against_schema | 1.0.0 | ✅ Stable | Platform Eng | Required |
| extract_from_artifact | 1.0.0 | ✅ Stable | Platform Eng | Required |

### Phase 1+ Extensions (Planned)

| Skill | Version | Status | Owner | Target Release |
|---|---|---|---|---|
| validate_performance_metrics | 1.0.0 | 🚧 In Development | QA Eng | M2 |
| browser_automation | 1.0.0 | 📋 Backlog | QA Eng | M3 |
| database_query | 1.0.0 | 📋 Backlog | Platform Eng | M3 |
| custom_validator | 1.0.0 | 📋 Backlog | Users | M4 |

---

## References

- **TECHNICAL_SPECIFICATION.md** — Skill framework and execution model
- **ERROR_CLASSIFICATION.md** — Error classification taxonomy
- **OPERATIONS_RUNBOOK.md** — Troubleshooting skill execution failures
- **METRICS_COLLECTION_SPEC.md** — Skill performance metrics

