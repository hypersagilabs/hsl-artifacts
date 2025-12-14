# Naming Conventions, Ports & Repository Structure

**Version:** 2.0  
**Last Updated:** 2025-12-14  
**Project:** RoboAgency - Humachine AI Studio

---

## Changelog

### Version 2.0 (2025-12-14)
- Updated service naming from `frontend_service` to `frontend` (actual implementation)
- Updated folder structure from `apps/frontend_service/` to `apps/frontend/`
- Added infra/docker/ structure with compose files
- Updated compose file reference from root to `infra/docker/compose.yml`
- Added status indicators (✅ implemented, 📋 planned) to folder structure
- Updated Docker container naming conventions

### Version 1.0 (2025-10-04)
- Initial conventions documentation

---

## Overview

This document establishes naming conventions, port assignments, directory structure, and coding standards for the autonomous AI agency project.

**Goal:** Maintain consistency, readability, and ease of onboarding for new developers.

---

## Table of Contents

1. [Naming Conventions](#naming-conventions)
2. [Port Assignments](#port-assignments)
3. [Repository Structure](#repository-structure)
4. [Code Style](#code-style)
5. [Git Workflow](#git-workflow)
6. [Environment Variables](#environment-variables)

---

## Naming Conventions

### Services

**Format:** `{purpose}_service`

| Service | Name | Description |
|---------|------|-------------|
| Frontend | `frontend` | SvelteKit web application (container: frontend) |
| Backend API | `orchestrator` | FastAPI + LangGraph orchestrator |
| Automation | `n8n` | n8n workflows |
| Auth | `auth` | Authentication service (optional) |

**Docker Container Names:**

```yaml
# infra/docker/compose.yml
services:
  frontend:       # Service name in compose
    container_name: frontend
  
  api:
    container_name: roboagency_api
  
  n8n:
    container_name: roboagency_n8n
```

---

### Files & Directories

**Python:**
- Modules: `snake_case.py` (e.g., `intake_parser.py`)
- Classes: `PascalCase` (e.g., `IntakeParser`)
- Functions: `snake_case()` (e.g., `parse_intake()`)
- Constants: `UPPER_SNAKE_CASE` (e.g., `MAX_RETRIES`)

**TypeScript/JavaScript:**
- Components: `PascalCase.svelte` (e.g., `IntakeForm.svelte`)
- Routes: `+page.svelte`, `+server.ts`
- Utils: `camelCase.ts` (e.g., `formatDate.ts`)
- Types: `PascalCase` (e.g., `LeadData`, `ProjectStatus`)

**SQL:**
- Tables: `snake_case` (e.g., `workflow_runs`)
- Columns: `snake_case` (e.g., `created_at`)
- Indexes: `idx_{table}_{column}` (e.g., `idx_leads_email`)

---

### API Endpoints

**Format:** `/v{version}/{resource}/{action}`

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/v1/intake` | Create lead |
| GET | `/v1/runs/{id}` | Get workflow status |
| POST | `/v1/workflows/{id}` | Start workflow |
| GET | `/v1/assets/{id}` | Get asset URL |
| GET | `/v1/health` | Health check |

**Rules:**
- Use plural for collections: `/v1/projects` not `/v1/project`
- Use nouns, not verbs: `/v1/users` not `/v1/get-users`
- Use kebab-case: `/v1/workflow-runs` not `/v1/workflowRuns`

---

### Database

**Tables:**
- Plural: `leads`, `projects`, `assets`
- Lowercase with underscores: `workflow_runs`

**Columns:**
- Timestamps: `created_at`, `updated_at`, `deleted_at`
- IDs: `id` (primary key), `{table}_id` (foreign key)
- Status: `status` (not `state` or `current_status`)

**Example:**
```sql
CREATE TABLE workflow_runs (
  id UUID PRIMARY KEY,
  project_id UUID REFERENCES projects(id),
  status VARCHAR(50),
  created_at TIMESTAMPTZ
);
```

---

### Environment Variables

**Format:** `{SERVICE}_{CATEGORY}_{NAME}`

```bash
# App config
NODE_ENV=production
PUBLIC_BASE_URL=https://yourdomain.tld

# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=roboagency
DB_USER=postgres
DB_PASSWORD=secret

# External services
OPENAI_API_KEY=sk-...
RESEND_API_KEY=re_...
SUPABASE_URL=https://...
SUPABASE_SERVICE_KEY=...

# Internal services
REDIS_URL=redis://redis:6379
QDRANT_URL=http://qdrant:6333
```

---

## Port Assignments

### Port Ranges

| Range | Purpose | Examples |
|-------|---------|----------|
| 3000-3999 | Frontend apps | 3000: SvelteKit |
| 5000-5999 | Backend APIs | 5001: FastAPI, 5002: Auth |
| 6000-6999 | Databases (dev) | 6333: Qdrant, 6379: Redis |
| 8000-8999 | Internal services | 8080: n8n |

### Assigned Ports

| Service | Port | URL |
|---------|------|-----|
| Frontend (SvelteKit) | 3000 | http://localhost:3000 |
| API (FastAPI) | 5001 | http://localhost:5001 |
| Orchestrator | 5002 | http://localhost:5002 |
| n8n | 8080 | http://localhost:8080 |
| PostgreSQL | 5432 | localhost:5432 |
| Redis | 6379 | localhost:6379 |
| Qdrant | 6333 | http://localhost:6333 |
| Caddy | 80, 443 | http://localhost |

**Production:**

All services are behind Caddy and accessed via host-based routing:

- `https://agency.yourdomain.tld` → Frontend (3000)
- `https://api.yourdomain.tld` → API (5001)
- `https://automations.yourdomain.tld` → n8n (5678)
- `https://grafana.yourdomain.tld` → Grafana (3000, optional)

---

## Repository Structure

### Monorepo Layout

```
roboagency/
├── apps/
│   ├── frontend/                  # SvelteKit app (✅ implemented)
│   │   ├── src/
│   │   │   ├── routes/
│   │   │   ├── lib/
│   │   │   └── app.html
│   │   ├── static/
│   │   ├── Dockerfile
│   │   └── package.json
│   │
│   ├── orchestrator/              # FastAPI + LangGraph (📋 planned)
│   │   ├── agents/
│   │   │   ├── orchestrator.py
│   │   │   ├── intake_parser.py
│   │   │   ├── product_doc_writer.py
│   │   │   └── ...
│   │   ├── api/
│   │   │   ├── main.py
│   │   │   ├── routes/
│   │   │   └── middleware/
│   │   ├── models/
│   │   ├── utils/
│   │   ├── tests/
│   │   ├── Dockerfile
│   │   └── requirements.txt
│   │
│   └── automation_service/        # n8n workflows
│       ├── workflows/
│       │   ├── lead_warmup.json
│       │   └── content_distribution.json
│       └── docker-compose.n8n.yml
│
├── packages/                       # Shared code
│   ├── shared/
│   │   ├── schemas/
│   │   ├── types/
│   │   └── utils/
│   └── ui/                         # Shared UI components
│
├── infra/                          # Infrastructure config
│   ├── docker/                     # Docker compose files (✅ implemented)
│   │   ├── compose.yml
│   │   ├── compose.dev.yml
│   │   └── compose.monitoring.yml
│   ├── caddy/                      # Caddy config (✅ implemented)
│   │   └── Caddyfile
│   └── monitoring/                 # Monitoring config (✅ implemented)
│       ├── prometheus/
│       └── grafana/
│   ├── caddy/
│   │   └── Caddyfile
│   ├── monitoring/
│   │   ├── prometheus/
│   │   │   └── prometheus.yml
│   │   └── grafana/
│   │       └── provisioning/
│   └── scripts/
│       ├── backup.sh
│       └── deploy.sh
│
├── docs/                           # Documentation
│   ├── ARCHITECTURE.md
│   ├── WORKFLOWS.md
│   ├── AGENTS.md
│   ├── AUTOMATIONS.md
│   ├── API_DOCUMENTATION.md
│   ├── DATA_MODEL.md
│   ├── CONVENTIONS.md
│   ├── SECURITY_POLICY.md
│   ├── RUNBOOK.md
│   └── ROADMAP.md
│
├── tests/                          # Integration tests
│   ├── e2e/
│   └── integration/
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── deploy-staging.yml
│       └── deploy-prod.yml
│
├── .env.example
├── .env.development
├── .gitignore
├── README.md
└── package.json                    # Root package.json (workspaces)
```

---

## Code Style

### Python (PEP 8)

```python
# Good
def parse_intake(data: dict) -> LeadData:
    """Parse and validate intake form data.
    
    Args:
        data: Raw form data from frontend
        
    Returns:
        LeadData: Validated lead object
        
    Raises:
        ValidationError: If data is invalid
    """
    if not data.get("email"):
        raise ValidationError("Email is required")
    
    return LeadData(
        email=data["email"],
        sector=data["sector"],
        idea=data["idea"]
    )

# Bad
def parseIntake(d):
    if not d.get('email'): raise ValidationError('Email required')
    return LeadData(email=d['email'],sector=d['sector'],idea=d['idea'])
```

**Formatting:**
- Max line length: 88 characters (Black formatter)
- Use type hints
- Docstrings for all public functions
- Use `black`, `isort`, `mypy` for linting

---

### TypeScript/JavaScript (Prettier + ESLint)

```typescript
// Good
interface LeadData {
  email: string;
  sector: string;
  idea: string;
  constraints?: Record<string, unknown>;
}

async function createLead(data: LeadData): Promise<Lead> {
  const response = await fetch('/api/intake', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });
  
  if (!response.ok) {
    throw new Error(`Failed to create lead: ${response.statusText}`);
  }
  
  return await response.json();
}

// Bad
async function createLead(data){
  const response = await fetch('/api/intake',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(data)})
  if(!response.ok)throw new Error(`Failed to create lead: ${response.statusText}`)
  return await response.json()
}
```

**Formatting:**
- Max line length: 100 characters
- Use TypeScript for all new code
- Prefer `async/await` over `.then()`
- Use Prettier for formatting

---

### SQL Style

```sql
-- Good
SELECT 
  p.id,
  p.name,
  p.status,
  l.email AS lead_email,
  COUNT(a.id) AS asset_count
FROM projects p
LEFT JOIN leads l ON p.lead_id = l.id
LEFT JOIN assets a ON p.id = a.project_id
WHERE p.status = 'completed'
  AND p.created_at >= NOW() - INTERVAL '7 days'
GROUP BY p.id, p.name, p.status, l.email
ORDER BY p.created_at DESC
LIMIT 10;

-- Bad
select p.id,p.name,p.status,l.email AS lead_email,COUNT(a.id) AS asset_count from projects p left join leads l on p.lead_id=l.id left join assets a on p.id=a.project_id where p.status='completed' and p.created_at>=NOW()-INTERVAL '7 days' group by p.id,p.name,p.status,l.email order by p.created_at desc limit 10;
```

---

## Git Workflow

### Branch Naming

| Type | Format | Example |
|------|--------|---------|
| Feature | `feat/{description}` | `feat/add-intake-form` |
| Bug fix | `fix/{description}` | `fix/email-validation` |
| Hotfix | `hotfix/{description}` | `hotfix/security-patch` |
| Chore | `chore/{description}` | `chore/update-deps` |
| Docs | `docs/{description}` | `docs/api-endpoints` |

### Commit Messages

**Format:** `{type}({scope}): {description}`

```bash
# Good
feat(intake): add sector validation
fix(api): handle null pointer in workflow
docs(readme): update setup instructions
chore(deps): upgrade fastapi to 0.104.1

# Bad
added stuff
fixed bug
update
WIP
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `chore`: Maintenance
- `refactor`: Code refactoring
- `test`: Tests
- `ci`: CI/CD changes

### Pull Request Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Checklist
- [ ] Tests pass locally
- [ ] Linting passes
- [ ] Documentation updated
- [ ] Screenshots attached (if UI change)

## Related Issues
Fixes #123
```

---

## Tilt Conventions

### Resource Labels

Organize services using labels for better visualization and selective startup:

| Label | Purpose | Services |
|-------|---------|----------|
| `core` | Primary application services | frontend, orchestrator, workers |
| `infrastructure` | Edge and gateway services | caddy |
| `data` | Data storage services | redis, qdrant, postgres |
| `automation` | Workflow automation | n8n |
| `observability` | Monitoring and logging | prometheus, grafana |

**Usage:**

```bash
# Start only core services
tilt up -- --labels=core

# Start core + infrastructure
tilt up -- --labels=core,infrastructure

# Start everything
tilt up
```

### Resource Dependencies

Define dependencies to ensure proper startup order:

```python
dc_resource('frontend', 
    resource_deps=['caddy'])

dc_resource('orchestrator', 
    resource_deps=['redis', 'qdrant'])

dc_resource('grafana', 
    resource_deps=['prometheus'])
```

### Development Workflow

1. **Start services:** `tilt up`
2. **View dashboard:** Opens automatically at `http://localhost:10350`
3. **Make code changes:** Tilt auto-rebuilds on file changes
4. **View logs:** Click service in Tilt dashboard
5. **Stop services:** `Ctrl+C` or `tilt down`

### Best Practices

- **Always use Tilt in development** for consistent experience
- **Keep Tiltfile simple** - complex logic goes in shell scripts
- **Document resource dependencies** in comments
- **Use labels** to group related services
- **Test with `tilt ci`** before committing (runs headless)

---

## Environment Variables

### File Structure

```
.env.example        # Template (committed)
.env.development    # Local dev (not committed)
.env.production     # Production secrets (not committed)
.env.test           # Test environment (not committed)
```

### .gitignore

```
# Environment variables
.env
.env.local
.env.development
.env.production
.env.test

# Never commit
*.env
!.env.example
```

### Naming Standards

```bash
# Good
DATABASE_URL=postgresql://...
OPENAI_API_KEY=sk-...
SMTP_HOST=smtp.resend.com
PUBLIC_BASE_URL=https://...

# Bad
db_url=...
openai=...
smtpHost=...
baseUrl=...
```

---

## Documentation Standards

### README Template

```markdown
# Service Name

Brief description

## Setup

```bash
npm install
npm run dev
```

## Environment Variables

See `.env.example`

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Health check |

## Tests

```bash
npm test
```

## Deployment

See `docs/DEPLOYMENT.md`
```

### Code Comments

```python
# Good: Explain WHY, not WHAT
# Use exponential backoff to avoid overwhelming the API during outages
await retry_with_backoff(call_llm)

# Bad: Obvious comment
# Increment counter by 1
counter += 1
```

---

## Next Steps

1. **Set up** linting tools (Black, Prettier, ESLint)
2. **Configure** pre-commit hooks
3. **Document** any deviations from these conventions
4. **Review** conventions quarterly

---

**Document Version:** 1.0  
**Last Review Date:** 2025-10-04  
**Next Review Date:** 2025-11-04

