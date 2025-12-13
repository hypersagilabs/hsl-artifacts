# API Documentation

**Version:** 1.0  
**Last Updated:** 2025-10-04  
**Project:** RoboAgency - Humachine AI Studio

---

## Overview

This document defines the REST API contracts for the autonomous AI agency platform. All APIs follow RESTful principles and return JSON responses.

**Base URL:** `https://api.yourdomain.tld/v1`

---

## Table of Contents

1. [Authentication](#authentication)
2. [API Endpoints](#api-endpoints)
3. [Error Handling](#error-handling)
4. [Rate Limiting](#rate-limiting)
5. [Webhooks](#webhooks)
6. [SDK & Client Libraries](#sdk--client-libraries)

---

## Authentication

### API Key Authentication

```bash
# Include API key in header
curl -H "Authorization: Bearer YOUR_API_KEY" \
  https://api.yourdomain.tld/v1/intake
```

### Service-to-Service Authentication

Internal services use JWT tokens with shared secret:

```python
import jwt

token = jwt.encode(
    {"service": "orchestrator", "exp": datetime.utcnow() + timedelta(hours=1)},
    SECRET_KEY,
    algorithm="HS256"
)
```

---

## API Endpoints

### 1. POST /intake

Create a new lead and project from intake form submission.

**Request:**

```http
POST /v1/intake
Content-Type: application/json

{
  "sector": "healthtech",
  "idea": "A telemedicine app for rural areas with limited internet connectivity",
  "email": "user@example.com",
  "constraints": {
    "budget": "50k",
    "timeline": "3 months",
    "tech_stack": "mobile-first"
  }
}
```

**Response (201 Created):**

```json
{
  "lead_id": "550e8400-e29b-41d4-a716-446655440000",
  "project_id": "660e8400-e29b-41d4-a716-446655440001",
  "status": "queued",
  "estimated_completion": "2025-10-04T15:30:00Z",
  "message": "Your project has been queued for processing"
}
```

**Validation Rules:**
- `sector` (required): Must be one of predefined list
- `idea` (required): 50-500 characters
- `email` (required): Valid email format
- `constraints` (optional): Object with optional fields

**Error Response (400 Bad Request):**

```json
{
  "error": "validation_error",
  "message": "Invalid sector provided",
  "details": {
    "field": "sector",
    "allowed_values": ["healthtech", "fintech", "edtech", "saas", "ecommerce"]
  }
}
```

---

### 2. POST /workflows/{workflow_id}

Start an agent workflow for a project.

**Request:**

```http
POST /v1/workflows/proto_v1
Content-Type: application/json
Authorization: Bearer YOUR_API_KEY

{
  "lead_id": "550e8400-e29b-41d4-a716-446655440000",
  "options": {
    "priority": "high",
    "include_video": true,
    "target_platforms": ["web", "mobile"]
  }
}
```

**Response (202 Accepted):**

```json
{
  "run_id": "770e8400-e29b-41d4-a716-446655440002",
  "workflow_id": "proto_v1",
  "status": "running",
  "started_at": "2025-10-04T14:00:00Z",
  "estimated_completion": "2025-10-04T14:10:00Z"
}
```

---

### 3. GET /runs/{run_id}

Get the status and results of a workflow run.

**Request:**

```http
GET /v1/runs/770e8400-e29b-41d4-a716-446655440002
Authorization: Bearer YOUR_API_KEY
```

**Response (200 OK):**

```json
{
  "run_id": "770e8400-e29b-41d4-a716-446655440002",
  "workflow_id": "proto_v1",
  "status": "completed",
  "started_at": "2025-10-04T14:00:00Z",
  "completed_at": "2025-10-04T14:08:32Z",
  "duration_seconds": 512,
  "artifacts": [
    {
      "type": "prd",
      "url": "https://assets.yourdomain.tld/projects/abc123/prd.md",
      "created_at": "2025-10-04T14:02:15Z"
    },
    {
      "type": "wireframe",
      "url": "https://assets.yourdomain.tld/projects/abc123/wireframe.json",
      "created_at": "2025-10-04T14:04:20Z"
    },
    {
      "type": "prototype",
      "url": "https://assets.yourdomain.tld/projects/abc123/index.html",
      "created_at": "2025-10-04T14:06:45Z"
    },
    {
      "type": "demo_video",
      "url": "https://assets.yourdomain.tld/projects/abc123/demo.mp4",
      "created_at": "2025-10-04T14:08:30Z"
    }
  ],
  "metrics": {
    "validation_score": 0.92,
    "estimated_cost_usd": 2.45,
    "steps_completed": 8
  }
}
```

**Status Values:**
- `queued` - Waiting to start
- `running` - Currently processing
- `completed` - Finished successfully
- `failed` - Encountered error
- `cancelled` - Manually stopped

---

### 4. POST /webhooks/agent/{event}

Internal webhook for agent-to-service communication.

**Request:**

```http
POST /v1/webhooks/agent/artifacts_ready
Content-Type: application/json
X-Internal-Token: SERVICE_TOKEN

{
  "project_id": "abc123",
  "event": "artifacts_ready",
  "artifacts": [
    {"type": "prd", "url": "..."},
    {"type": "prototype", "url": "..."}
  ],
  "timestamp": "2025-10-04T14:08:32Z"
}
```

**Response (200 OK):**

```json
{
  "acknowledged": true,
  "next_action": "trigger_n8n_campaign"
}
```

**Event Types:**
- `artifacts_ready` - All artifacts generated
- `step_completed` - Individual step finished
- `error_occurred` - Workflow error
- `human_review_required` - Needs manual review

---

### 5. GET /assets/{asset_id}

Retrieve a signed URL for an asset.

**Request:**

```http
GET /v1/assets/abc123/prd.md
Authorization: Bearer YOUR_API_KEY
```

**Response (200 OK):**

```json
{
  "asset_id": "abc123/prd.md",
  "url": "https://assets.yourdomain.tld/abc123/prd.md?signature=...",
  "expires_at": "2025-10-04T15:00:00Z",
  "content_type": "text/markdown",
  "size_bytes": 12845
}
```

---

### 6. GET /health

Service health check endpoint.

**Request:**

```http
GET /v1/health
```

**Response (200 OK):**

```json
{
  "status": "healthy",
  "version": "1.0.0",
  "uptime_seconds": 86400,
  "dependencies": {
    "database": {
      "status": "up",
      "latency_ms": 12
    },
    "redis": {
      "status": "up",
      "latency_ms": 3
    },
    "qdrant": {
      "status": "up",
      "latency_ms": 8
    },
    "s3": {
      "status": "up",
      "latency_ms": 45
    }
  }
}
```

**Unhealthy Response (503 Service Unavailable):**

```json
{
  "status": "unhealthy",
  "version": "1.0.0",
  "dependencies": {
    "database": {
      "status": "down",
      "error": "connection timeout"
    }
  }
}
```

---

## Error Handling

### Error Response Format

All errors follow this structure:

```json
{
  "error": "error_code",
  "message": "Human-readable error message",
  "details": {
    "field": "value",
    "additional": "context"
  },
  "request_id": "req_abc123xyz"
}
```

### HTTP Status Codes

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | OK | Successful GET/PUT/DELETE |
| 201 | Created | Successful POST |
| 202 | Accepted | Async operation started |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Missing/invalid auth |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Unexpected error |
| 503 | Service Unavailable | Temporarily down |

### Common Error Codes

```json
{
  "validation_error": "Invalid input data",
  "authentication_error": "Invalid API key",
  "rate_limit_exceeded": "Too many requests",
  "resource_not_found": "Requested resource doesn't exist",
  "workflow_failed": "Agent workflow encountered error",
  "service_unavailable": "Dependent service is down"
}
```

---

## Rate Limiting

### Limits

| Endpoint | Limit | Window |
|----------|-------|--------|
| POST /intake | 10 requests | 1 hour |
| GET /runs/* | 100 requests | 1 minute |
| GET /assets/* | 1000 requests | 1 hour |
| All other endpoints | 60 requests | 1 minute |

### Headers

```http
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1633046400
```

### Rate Limit Exceeded Response

```json
{
  "error": "rate_limit_exceeded",
  "message": "You have exceeded the rate limit",
  "details": {
    "limit": 60,
    "window_seconds": 60,
    "retry_after": 23
  }
}
```

---

## Webhooks

### Outbound Webhooks

The system can send webhooks to your configured URL on certain events.

**Configuration:**

```json
POST /v1/webhooks/subscribe
{
  "url": "https://your-service.com/webhooks",
  "events": ["workflow_completed", "workflow_failed"],
  "secret": "your_webhook_secret"
}
```

**Webhook Payload:**

```json
{
  "event": "workflow_completed",
  "timestamp": "2025-10-04T14:08:32Z",
  "data": {
    "run_id": "770e8400-e29b-41d4-a716-446655440002",
    "project_id": "abc123",
    "status": "completed",
    "artifacts": [...]
  },
  "signature": "sha256=..."
}
```

**Signature Verification:**

```python
import hmac
import hashlib

def verify_webhook(payload: bytes, signature: str, secret: str) -> bool:
    expected_signature = "sha256=" + hmac.new(
        secret.encode(),
        payload,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected_signature, signature)
```

---

## SDK & Client Libraries

### Python SDK

```python
# Installation
pip install roboagency-sdk

# Usage
from roboagency import Client

client = Client(api_key="YOUR_API_KEY")

# Create lead
lead = client.intake.create(
    sector="healthtech",
    idea="Telemedicine app for rural areas",
    email="user@example.com"
)

# Start workflow
run = client.workflows.start("proto_v1", lead_id=lead.id)

# Poll for completion
while run.status == "running":
    time.sleep(10)
    run = client.runs.get(run.id)

# Download artifacts
for artifact in run.artifacts:
    client.assets.download(artifact.url, f"output/{artifact.type}")
```

### JavaScript/TypeScript SDK

```typescript
// Installation
npm install @roboagency/sdk

// Usage
import { RoboAgency } from '@roboagency/sdk';

const client = new RoboAgency({ apiKey: 'YOUR_API_KEY' });

// Create lead
const lead = await client.intake.create({
  sector: 'healthtech',
  idea: 'Telemedicine app for rural areas',
  email: 'user@example.com'
});

// Start workflow
const run = await client.workflows.start('proto_v1', {
  leadId: lead.id
});

// Wait for completion
const completed = await run.waitForCompletion();
console.log(completed.artifacts);
```

---

## OpenAPI Specification

Full OpenAPI 3.0 spec available at:

**URL:** `https://api.yourdomain.tld/v1/openapi.json`

**Example:**

```yaml
openapi: 3.0.0
info:
  title: RoboAgency API
  version: 1.0.0
  description: Autonomous AI Agency API
servers:
  - url: https://api.yourdomain.tld/v1
paths:
  /intake:
    post:
      summary: Create new lead
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/IntakeRequest'
      responses:
        '201':
          description: Lead created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/IntakeResponse'
components:
  schemas:
    IntakeRequest:
      type: object
      required:
        - sector
        - idea
        - email
      properties:
        sector:
          type: string
          enum: [healthtech, fintech, edtech, saas, ecommerce]
        idea:
          type: string
          minLength: 50
          maxLength: 500
        email:
          type: string
          format: email
```

---

## Testing

### cURL Examples

```bash
# Create lead
curl -X POST https://api.yourdomain.tld/v1/intake \
  -H "Content-Type: application/json" \
  -d '{
    "sector": "healthtech",
    "idea": "Telemedicine app for rural areas",
    "email": "test@example.com"
  }'

# Check workflow status
curl https://api.yourdomain.tld/v1/runs/770e8400 \
  -H "Authorization: Bearer YOUR_API_KEY"
```

### Postman Collection

Download the Postman collection:

**URL:** `https://api.yourdomain.tld/v1/postman.json`

---

## Next Steps

1. **Generate** API keys for testing
2. **Import** OpenAPI spec into Postman
3. **Test** all endpoints
4. **Integrate** with frontend
5. **Monitor** API usage and errors

---

**Document Version:** 1.0  
**Last Review Date:** 2025-10-04  
**Next Review Date:** 2025-11-04

