# HyperSagi Labs Deployment Roadmap

**Version:** 1.0  
**Last Updated:** 2025-10-04  
**Project:** hsl-website

---

## Executive Summary

This roadmap outlines the complete deployment strategy for the HyperSagi Labs website, from local development to production-ready microservices architecture with Caddy edge gateway, host-based routing, and observability.

**Estimated Timeline:** 4-6 weeks  
**Complexity:** Medium to High  
**Key Technologies:** Docker, Caddy, Prometheus/Grafana, SvelteKit, Node.js

---

## Current State Assessment

### ✅ Completed
- [x] SvelteKit 2 application built and tested locally
- [x] Tailwind CSS v4 styling configured
- [x] Basic pages: Home, About, Services, Industries, Contact
- [x] Development environment setup (Node.js 20, npm)

### ⏳ In Progress
- [ ] Production build verification
- [ ] Environment variables management

### ❌ Not Started
- [ ] Docker containerization
- [ ] Domain and DNS setup
- [ ] Email service configuration
- [ ] Production deployment
- [ ] Monitoring and logging

---

## Phase 1: Containerization & Local Infrastructure (Week 1)

### Objectives
- Dockerize the SvelteKit application
- Set up local Docker Compose environment
- Configure Caddy as edge gateway
- Establish development-production parity

### Tasks

#### 1.1 Create Dockerfile for SvelteKit App
**Priority:** HIGH  
**Time:** 2-3 hours

```dockerfile
# Multi-stage build for optimal image size
# - Builder stage for dependencies and build
# - Production stage with minimal runtime
```

**Deliverables:**
- `Dockerfile` in project root
- `.dockerignore` file
- Build verification script

#### 1.2 Set Up Docker Compose
**Priority:** HIGH  
**Time:** 2-3 hours

**Services to define:**
- `web` - SvelteKit application (Node.js)
- `caddy` - Edge gateway and reverse proxy
- `postgres` (optional) - Database if needed later

**Deliverables:**
- `docker-compose.yml`
- `docker-compose.dev.yml` (development overrides)
- `docker-compose.prod.yml` (production overrides)

#### 1.3 Configure Caddy
**Priority:** HIGH  
**Time:** 2-4 hours

**Configuration needs:**
- Host-based routing (agency.domain, api.domain, automations.domain)
- Reverse proxy to SvelteKit app and backend services
- Automatic HTTPS (Let's Encrypt)
- Compression (gzip, zstd)
- Security headers
- Rate limiting
- Internal metrics endpoint (admin port 2019)

**Deliverables:**
- `infra/caddy/Caddyfile`
- Caddy data/config volumes for certificate storage

#### 1.4 Environment Variables Management
**Priority:** HIGH  
**Time:** 1-2 hours

**Files to create:**
- `.env.example` (template with all variables)
- `.env.development` (local development)
- `.env.production` (production - not in git)

**Variables needed:**
```bash
# App
NODE_ENV=production
PUBLIC_URL=https://hypersagilabs.com

# Email (SMTP)
SMTP_HOST=
SMTP_PORT=
SMTP_USER=
SMTP_PASS=
SMTP_FROM=
CONTACT_EMAIL=

# Database (future)
DATABASE_URL=

# Caddy
ACME_EMAIL=your-email@example.com
DOMAIN=yourdomain.tld

# Monitoring (future)
GRAFANA_ADMIN_PASSWORD=
```

#### 1.5 Testing & Verification
**Priority:** HIGH  
**Time:** 2-3 hours

**Tests:**
- [ ] Build Docker image successfully
- [ ] Run container locally
- [ ] Access via `http://localhost`
- [ ] Caddy routing works correctly
- [ ] Static assets served properly
- [ ] API routes functional
- [ ] Hot reload in development mode

---

## Phase 2: Domain & DNS Setup (Week 1-2)

### Objectives
- Register or configure domain name
- Configure DNS A records for subdomains
- Note: MX/SPF/DKIM/DMARC remain unchanged (Google Workspace compatibility)

### Tasks

#### 2.1 Domain Registration/Transfer
**Priority:** HIGH  
**Time:** 1 hour + waiting time

**Steps:**
1. Register `yourdomain.tld` (if not already owned)
2. Choose DNS provider (Cloudflare, Namecheap, Google Domains, etc.)
3. Verify domain ownership

**Note:** Cloudflare is optional - any DNS provider works. We only need DNS management, not WAF/CDN features.

#### 2.2 DNS Configuration
**Priority:** HIGH  
**Time:** 1 hour

**DNS A Records to create (point to your server IP):**
```
A     agency          <your-server-ip>
A     api             <your-server-ip>
A     automations     <your-server-ip>
A     grafana         <your-server-ip>  (optional)
```

**Important:** Do NOT modify existing MX/SPF/DKIM/DMARC records if using Google Workspace for email. These remain unchanged.

**Existing email records (do not change):**
```
MX    @               10 mail.google.com
TXT   @               "v=spf1 include:_spf.google.com ~all"
TXT   _dmarc          "v=DMARC1; p=quarantine; rua=mailto:dmarc@yourdomain.tld"
```

#### 2.3 DNS Propagation
**Priority:** HIGH  
**Time:** Wait 24-48 hours

**Steps:**
1. Verify DNS propagation using `dig` or online tools
2. Confirm A records resolve correctly
3. Test subdomain accessibility (will work after Caddy is configured)

---

## Phase 3: Server Provisioning & Initial Deployment (Week 2)

### Objectives
- Provision production server
- Set up security and firewall
- Deploy initial application version
- Configure SSL/TLS

### Tasks

#### 3.1 Server Selection & Provisioning
**Priority:** HIGH  
**Time:** 2-3 hours

**Recommended Providers:**
- **DigitalOcean** (recommended for simplicity)
  - Droplet: $12/mo (2 vCPU, 2GB RAM, 50GB SSD)
- **Hetzner** (best value)
  - CX21: €5.83/mo (2 vCPU, 4GB RAM, 40GB SSD)
- **AWS EC2** (enterprise grade)
  - t3.small: ~$15/mo

**Specifications:**
- OS: Ubuntu 22.04 LTS
- RAM: 2GB minimum (4GB recommended)
- Storage: 40GB minimum
- Location: Choose closest to target audience

#### 3.2 Server Initial Setup
**Priority:** HIGH  
**Time:** 2-3 hours

**Setup checklist:**
```bash
# Update system
apt update && apt upgrade -y

# Create non-root user
adduser deploy
usermod -aG sudo deploy

# Set up SSH keys (disable password auth)
# Configure firewall (UFW)
ufw allow OpenSSH
ufw allow 80/tcp
ufw allow 443/tcp
ufw enable

# Install Docker & Docker Compose
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
usermod -aG docker deploy

# Install Docker Compose
apt install docker-compose-plugin

# Install fail2ban (security)
apt install fail2ban
systemctl enable fail2ban
```

#### 3.3 SSL/TLS Certificate Setup
**Priority:** HIGH  
**Time:** Automatic (handled by Caddy)

**Caddy Automatic HTTPS:**
Caddy automatically obtains and renews Let's Encrypt certificates. No manual certificate management needed.

**Configuration:**
1. Set `ACME_EMAIL` environment variable in `.env.production`
2. Ensure DNS A records are properly configured
3. Caddy will automatically:
   - Obtain Let's Encrypt certificates on first start
   - Renew certificates automatically
   - Handle HTTP to HTTPS redirects
   - Store certificates in Docker volume (`caddy_data`)

**No additional steps required** - Caddy handles everything automatically.

#### 3.4 Deploy Application
**Priority:** HIGH  
**Time:** 2-3 hours

**Deployment steps:**
```bash
# Clone repository
cd /opt
git clone https://github.com/yourusername/hsl-website.git
cd hsl-website

# Set up environment
cp .env.example .env.production
nano .env.production  # Configure variables

# Build and start services
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d --build

# Verify deployment
docker compose ps
docker compose logs -f web
```

#### 3.5 Caddy Production Configuration
**Priority:** HIGH  
**Time:** 30 minutes

**Key configurations (in Caddyfile):**
- Host-based routing for subdomains
- Automatic HTTPS (handled automatically)
- Security headers (built-in)
- Compression (gzip, zstd)
- Rate limiting (optional, via Caddy plugins)
- Internal metrics endpoint (admin port 2019, not exposed publicly)

---

## Phase 4: Email Service Configuration (Week 2-3)

### Objectives
- Set up transactional email service
- Configure SMTP for contact form
- Set up email forwarding for team
- Implement SPF, DKIM, DMARC

### Tasks

#### 4.1 Choose Email Service Provider
**Priority:** HIGH  
**Time:** 1 hour

**Recommended Providers:**

| Provider | Free Tier | Pricing | Best For |
|----------|-----------|---------|----------|
| **Resend** | 3,000/mo | $20/mo (50k) | Modern API, great DX |
| **SendGrid** | 100/day | $19.95/mo (50k) | Established, reliable |
| **Mailgun** | 5,000/mo | $35/mo (50k) | Developer-friendly |
| **Amazon SES** | 62,000/mo (EC2) | Pay-as-you-go | Cheapest at scale |

**Recommendation:** Start with **Resend** or **Mailgun** for ease of use.

#### 4.2 Configure SMTP in Application
**Priority:** HIGH  
**Time:** 1-2 hours

**Steps:**
1. Sign up for email service
2. Verify domain ownership
3. Get SMTP credentials
4. Update `.env.production`:
```bash
SMTP_HOST=smtp.resend.com
SMTP_PORT=587
SMTP_USER=resend
SMTP_PASS=re_xxxxxxxxxxxxx
SMTP_FROM=hello@hypersagilabs.com
CONTACT_EMAIL=hello@hypersagilabs.com
```
5. Test contact form submission

#### 4.3 DNS Records for Email Authentication
**Priority:** HIGH  
**Time:** 1 hour

**Records to add in DNS (any provider):**

**SPF Record:**
```
TXT @ "v=spf1 include:_spf.google.com include:_spf.resend.com ~all"
```

**DKIM Record:**
(Provided by your email service)
```
TXT resend._domainkey "v=DKIM1; k=rsa; p=MIGfMA0GCSq..."
```

**DMARC Record:**
```
TXT _dmarc "v=DMARC1; p=quarantine; rua=mailto:dmarc@yourdomain.tld; pct=100; adkim=s; aspf=s"
```

**Note:** If using Google Workspace, keep existing MX and SPF records. Only add Resend to SPF.

#### 4.4 Email Forwarding Setup
**Priority:** MEDIUM  
**Time:** 30 minutes

**Options:**
1. **Google Workspace** (if already configured) - Use existing email forwarding
2. **DNS Provider** - Many DNS providers offer email forwarding
3. **Third-party service** - Use a dedicated email forwarding service

#### 4.5 Testing & Verification
**Priority:** HIGH  
**Time:** 1 hour

**Tests:**
- [ ] Send test email via contact form
- [ ] Verify email deliverability
- [ ] Check spam score (mail-tester.com)
- [ ] Verify SPF, DKIM, DMARC (mxtoolbox.com)
- [ ] Test email forwarding

---

## Phase 5: Security Hardening (Week 3)

### Objectives
- Configure server-level security (UFW firewall)
- Harden SSH access
- Set up optional intrusion detection (CrowdSec or fail2ban)
- Configure Caddy security headers

### Tasks

#### 5.1 UFW Firewall Configuration
**Priority:** HIGH  
**Time:** 30 minutes

**Configuration:**
```bash
# Review current rules
ufw status verbose

# Ensure only necessary ports are open
ufw default deny incoming
ufw default allow outgoing

# Allow SSH (be careful - ensure SSH keys work first!)
ufw allow 22/tcp

# Allow HTTP/HTTPS (Caddy)
ufw allow 80/tcp
ufw allow 443/tcp

# Enable firewall
ufw enable

# Verify
ufw status
```

**Important:** Only open ports 22, 80, and 443. All other services (Redis, Qdrant, Prometheus, Grafana) are on internal Docker networks and not exposed.

#### 5.2 SSH Hardening
**Priority:** HIGH  
**Time:** 30 minutes

**Configuration (`/etc/ssh/sshd_config`):**
```bash
# Disable root login
PermitRootLogin no

# Disable password authentication (use keys only)
PasswordAuthentication no
PubkeyAuthentication yes

# Limit authentication attempts
MaxAuthTries 3

# Disconnect idle sessions
ClientAliveInterval 300
ClientAliveCountMax 2

# Restart SSH service
sudo systemctl restart sshd
```

**Before applying:** Ensure your SSH key works and you can log in without password!

#### 5.3 Intrusion Detection (Choose One)

**Option A: CrowdSec (Recommended)**
**Priority:** MEDIUM  
**Time:** 1-2 hours

**Benefits:**
- Community-driven threat intelligence
- Automatic remediation
- Optional WAF-like features
- Better than fail2ban for modern threats

**Installation:**
```bash
# Install CrowdSec
curl -s https://packagecloud.io/install/repositories/crowdsec/crowdsec/script.deb.sh | sudo bash
sudo apt install crowdsec

# Install Caddy bouncer
sudo cscli collections install crowdsecurity/caddy
sudo systemctl enable crowdsec
```

**Option B: fail2ban (Alternative)**
**Priority:** MEDIUM  
**Time:** 1 hour

**Configuration:**
```bash
# Install fail2ban
sudo apt install fail2ban

# Configure jail
sudo nano /etc/fail2ban/jail.local
```

```ini
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
```

```bash
sudo systemctl enable fail2ban
sudo systemctl restart fail2ban
```

**Note:** Ensure logs contain real client IPs (not proxy IPs) for fail2ban to work correctly.

#### 5.4 Caddy Security Headers
**Priority:** HIGH  
**Time:** 30 minutes

**Configure in Caddyfile:**
```caddyfile
{
  email {$ACME_EMAIL}
  admin 0.0.0.0:2019
}

agency.{$DOMAIN} {
  encode gzip zstd
  
  # Security headers
  header {
    X-Frame-Options "SAMEORIGIN"
    X-Content-Type-Options "nosniff"
    X-XSS-Protection "1; mode=block"
    Referrer-Policy "strict-origin-when-cross-origin"
    Permissions-Policy "geolocation=(), microphone=(), camera=()"
  }
  
  reverse_proxy frontend:3000
}
```

#### 5.5 Rate Limiting (Optional)
**Priority:** MEDIUM  
**Time:** 1 hour

**Caddy Rate Limiting:**
Caddy supports rate limiting via plugins or middleware. For basic rate limiting, consider:
- Application-level rate limiting (in FastAPI/SvelteKit)
- Caddy rate limiting plugins
- Or rely on CrowdSec for advanced rate limiting

**Example (application-level in FastAPI):**
```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@app.post("/api/intake")
@limiter.limit("5/hour")
async def create_intake(request: Request):
    # Handle intake
    pass
```

---

## Phase 6: Microservices Architecture (Week 3-4)

### Objectives
- Design microservices architecture
- Separate concerns into independent services
- Set up service communication
- Implement API gateway pattern

### Tasks

#### 6.1 Architecture Design
**Priority:** HIGH  
**Time:** 3-4 hours

**Proposed Microservices:**

```
┌─────────────────────────────────────────────────────────┐
│                    Caddy (Edge Gateway)                 │
│              Host-based routing + HTTPS                 │
└─────────────────────────────────────────────────────────┘
           │              │              │
           ↓              ↓              ↓
    ┌──────────┐   ┌──────────┐   ┌──────────┐
    │   Web    │   │  Contact │   │   CMS    │
    │ Service  │   │ Service  │   │ Service  │
    │(SvelteKit)   │ (Node.js)│   │(Headless)│
    └──────────┘   └──────────┘   └──────────┘
                          │
                          ↓
                   ┌──────────┐
                   │  Email   │
                   │ Service  │
                   │ (SMTP)   │
                   └──────────┘
```

**Services:**

1. **Web Service (SvelteKit)** - Current app
   - Purpose: Frontend and SSR
   - Port: 3000
   - Dependencies: None

2. **Contact Service (Node.js/Express)**
   - Purpose: Handle form submissions, validation
   - Port: 3001
   - Dependencies: Email Service, Database (future)
   - API: `/api/contact/submit`

3. **Email Service (Microservice)**
   - Purpose: Send transactional emails
   - Port: 3002
   - Dependencies: SMTP provider
   - API: `/api/email/send`

4. **CMS Service (Future - Strapi/Payload)**
   - Purpose: Manage blog content, case studies
   - Port: 3003
   - Dependencies: Database
   - API: `/api/cms/*`

#### 6.2 Create Contact Microservice
**Priority:** HIGH  
**Time:** 4-6 hours

**Directory structure:**
```
services/
├── web/                    # Existing SvelteKit app
├── contact/
│   ├── src/
│   │   ├── index.js       # Express server
│   │   ├── routes/
│   │   ├── controllers/
│   │   ├── validators/
│   │   └── utils/
│   ├── Dockerfile
│   ├── package.json
│   └── .env.example
├── email/
│   └── ... (similar structure)
└── docker-compose.microservices.yml
```

**Contact Service API:**
```javascript
POST /api/contact/submit
{
  "name": "string",
  "email": "string",
  "company": "string?",
  "industry": "string?",
  "service": "string?",
  "message": "string"
}
```

#### 6.3 Update Docker Compose for Microservices
**Priority:** HIGH  
**Time:** 2-3 hours

**New docker-compose.microservices.yml:**
```yaml
version: '3.8'

services:
  caddy:
    image: caddy:2
    container_name: caddy
    ports:
      - "80:80"
      - "443:443"
    environment:
      - ACME_EMAIL=${ACME_EMAIL}
      - DOMAIN=${DOMAIN}
    volumes:
      - ./infra/caddy/Caddyfile:/etc/caddy/Caddyfile:ro
      - caddy_data:/data
      - caddy_config:/config
    networks: [edge]
    restart: unless-stopped
    depends_on:
      - web
      - contact
      - email

  web:
    build: ./services/web
    environment:
      - NODE_ENV=production
      - CONTACT_API_URL=http://contact:3001
    expose:
      - "3000"
    networks: [edge]

  contact:
    build: ./services/contact
    environment:
      - NODE_ENV=production
      - EMAIL_API_URL=http://email:3002
      - DATABASE_URL=${DATABASE_URL}
    expose:
      - "3001"
    networks: [edge]

  email:
    build: ./services/email
    environment:
      - NODE_ENV=production
      - SMTP_HOST=${SMTP_HOST}
      - SMTP_PORT=${SMTP_PORT}
      - SMTP_USER=${SMTP_USER}
      - SMTP_PASS=${SMTP_PASS}
    expose:
      - "3002"
    networks: [edge]

networks:
  edge:

volumes:
  caddy_data:
  caddy_config:
```

#### 6.4 API Gateway Configuration (Caddy)
**Priority:** HIGH  
**Time:** 1-2 hours

**Caddy host-based routing configuration:**
```caddyfile
{
  email {$ACME_EMAIL}
  admin 0.0.0.0:2019
}

# Web application (SvelteKit)
agency.{$DOMAIN} {
  encode gzip zstd
  reverse_proxy web:3000
}

# API service
api.{$DOMAIN} {
  encode gzip zstd
  reverse_proxy contact:3001
}

# Automations (n8n)
automations.{$DOMAIN} {
  encode gzip zstd
  reverse_proxy n8n:5678
}
```

**Note:** Host-based routing is cleaner than path-based routing. Each service gets its own subdomain.

#### 6.5 Service Communication & API Contracts
**Priority:** MEDIUM  
**Time:** 2-3 hours

**Implement:**
- REST API contracts between services
- Error handling and retry logic
- Health check endpoints for each service
- Logging and monitoring hooks

---

## Phase 7: CI/CD Pipeline (Week 4)

### Objectives
- Set up automated deployment
- Implement testing pipeline
- Configure staging environment
- Enable zero-downtime deployments

### Tasks

#### 7.1 Git Repository Setup
**Priority:** HIGH  
**Time:** 1 hour

**Structure:**
```
.github/
└── workflows/
    ├── ci.yml              # Run tests on PR
    ├── deploy-staging.yml  # Deploy to staging
    └── deploy-prod.yml     # Deploy to production
```

#### 7.2 GitHub Actions CI Pipeline
**Priority:** HIGH  
**Time:** 2-3 hours

**CI workflow (ci.yml):**
- Run linting (ESLint, Prettier)
- Run type checking (TypeScript)
- Run unit tests
- Run integration tests
- Build Docker images
- Security scanning (Trivy)

#### 7.3 Staging Environment Setup
**Priority:** MEDIUM  
**Time:** 2-3 hours

**Create:**
- `staging.hypersagilabs.com` subdomain
- Separate Docker Compose stack
- Staging database (if applicable)
- Staging email (catch-all)

#### 7.4 Production Deployment Workflow
**Priority:** HIGH  
**Time:** 3-4 hours

**Deployment steps:**
1. Build Docker images
2. Push to container registry
3. SSH to production server
4. Pull latest images
5. Run database migrations (if any)
6. Rolling restart services (zero downtime)
7. Health check verification
8. Rollback on failure

#### 7.5 Secrets Management
**Priority:** HIGH  
**Time:** 1-2 hours

**Options:**
- GitHub Secrets (simple)
- HashiCorp Vault (enterprise)
- AWS Secrets Manager (AWS-based)

**Store:**
- Server SSH keys
- Docker registry credentials
- Database credentials
- SMTP credentials
- Caddy configuration

---

## Phase 8: Observability (Prometheus + Grafana) (Week 4)

### Objectives
- Set up Prometheus for metrics collection
- Configure Grafana for visualization
- Scrape Caddy metrics (internal network only)
- Set up application health monitoring

### Tasks

#### 8.1 Prometheus Setup
**Priority:** HIGH  
**Time:** 2-3 hours

**Create docker-compose.monitoring.yml:**
```yaml
services:
  prometheus:
    image: prom/prometheus:v2.45.0
    container_name: prometheus
    command: ['--config.file=/etc/prometheus/prometheus.yml']
    volumes:
      - ./infra/monitoring/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus
    networks: [observability]
    restart: unless-stopped

  grafana:
    image: grafana/grafana:10.0.3
    container_name: grafana
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_ADMIN_PASSWORD}
    volumes:
      - ./infra/monitoring/grafana/provisioning:/etc/grafana/provisioning
      - ./infra/monitoring/grafana/dashboards:/var/lib/grafana/dashboards
      - grafana_data:/var/lib/grafana
    networks: [observability]
    restart: unless-stopped
    depends_on: [prometheus]

networks:
  observability:
    external: false

volumes:
  prometheus_data:
  grafana_data:
```

**Deliverables:**
- `docker-compose.monitoring.yml`
- `infra/monitoring/prometheus/prometheus.yml`
- Prometheus data volume

#### 8.2 Prometheus Configuration
**Priority:** HIGH  
**Time:** 1-2 hours

**Create `infra/monitoring/prometheus/prometheus.yml`:**
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  # Caddy metrics (internal admin port)
  - job_name: "caddy"
    metrics_path: /metrics
    static_configs:
      - targets: ["caddy:2019"]
    
  # Application health endpoints (future)
  - job_name: "frontend"
    metrics_path: /health
    static_configs:
      - targets: ["frontend:3000"]
    
  - job_name: "orchestrator"
    metrics_path: /health
    static_configs:
      - targets: ["orchestrator:5001"]
```

**Important:** Caddy admin port (2019) is only accessible via the `observability` network. It is NOT exposed to the internet.

#### 8.3 Grafana Setup
**Priority:** HIGH  
**Time:** 1-2 hours

**Configure Grafana datasource:**
Create `infra/monitoring/grafana/provisioning/datasources/prometheus.yml`:
```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
```

**Import Caddy dashboard:**
1. Access Grafana (via `grafana.yourdomain.tld` or port-forward)
2. Import Caddy dashboard (search Grafana dashboard library for "Caddy")
3. Configure dashboard to use Prometheus datasource

**Optional:** Expose Grafana via Caddy (behind auth):
```caddyfile
grafana.{$DOMAIN} {
  basicauth /* {
    {$GRAFANA_USER} {$GRAFANA_PASS_HASH}
  }
  reverse_proxy grafana:3000
}
```

#### 8.4 Caddy Metrics Configuration
**Priority:** HIGH  
**Time:** 30 minutes

**Update Caddyfile to enable metrics:**
```caddyfile
{
  email {$ACME_EMAIL}
  
  # Enable metrics
  servers {
    metrics
  }
  
  # Admin API (internal only - not exposed publicly)
  admin 0.0.0.0:2019
}
```

**Verify metrics endpoint:**
```bash
# From within observability network
curl http://caddy:2019/metrics
```

#### 8.5 Additional Monitoring Tools
**Priority:** MEDIUM  
**Time:** 1-2 hours

**Error Tracking:**
- **Sentry** - Error tracking and performance monitoring (recommended)

**Uptime Monitoring:**
- **UptimeRobot** (Free, 50 monitors) - External uptime checks

**Web Analytics:**
- **Plausible Analytics** (€9/mo) - Privacy-friendly analytics
- Or **Umami** (Self-hosted, free)

**Implementation:**
```javascript
// Add to app.html
<script defer data-domain="yourdomain.tld" src="https://plausible.io/js/script.js"></script>
```

---

## Phase 9: Security Hardening (Week 4)

### Objectives
- Implement security best practices
- Set up automated backups
- Configure security monitoring
- Document security procedures

### Tasks

#### 9.1 Security Audit
**Priority:** HIGH  
**Time:** 2-3 hours

**Checklist:**
- [ ] All dependencies up to date
- [ ] No secrets in code/repo
- [ ] Environment variables secured
- [ ] Database credentials rotated
- [ ] SSH keys only (no password auth)
- [ ] Firewall configured (UFW)
- [ ] fail2ban configured
- [ ] Security headers implemented
- [ ] HTTPS enforced
- [ ] CORS configured properly

#### 9.2 Automated Backups
**Priority:** HIGH  
**Time:** 2-3 hours

**Backup strategy:**

**Application Code:**
- Version controlled in Git
- Multiple remotes (GitHub + GitLab)

**Database (when implemented):**
- Daily automated backups
- Retention: 7 days full, 4 weeks weekly, 12 months monthly
- Storage: S3 or DigitalOcean Spaces
- Encryption at rest

**Configuration:**
- Backup `infra/caddy/Caddyfile`
- Backup `.env` files (encrypted)
- Caddy certificates stored in Docker volume (automatically backed up with volume backup)

#### 9.3 Security Monitoring
**Priority:** MEDIUM  
**Time:** 1-2 hours

**Implement:**
- Prometheus alerting rules (if configured)
- CrowdSec/fail2ban monitoring
- SSH access logging
- Suspicious activity alerts

#### 9.4 Incident Response Plan
**Priority:** MEDIUM  
**Time:** 2 hours

**Document:**
- Security contact information
- Escalation procedures
- Backup restoration procedures
- Rollback procedures
- Communication templates

---

## Phase 10: Documentation & Handoff (Week 4)

### Objectives
- Complete technical documentation
- Create runbooks for operations
- Document architecture decisions
- Prepare maintenance procedures

### Tasks

#### 10.1 Technical Documentation
**Priority:** HIGH  
**Time:** 4-6 hours

**Documents to create:**
- Architecture overview diagram
- API documentation
- Database schema (when applicable)
- Environment setup guide
- Deployment procedures
- Troubleshooting guide

#### 10.2 Operational Runbooks
**Priority:** HIGH  
**Time:** 3-4 hours

**Runbooks:**
1. **Deployment Runbook**
   - Pre-deployment checklist
   - Deployment steps
   - Verification steps
   - Rollback procedure

2. **Incident Response Runbook**
   - Incident classification
   - Response procedures
   - Communication plan
   - Post-mortem template

3. **Maintenance Runbook**
   - Server updates
   - Dependency updates
   - Database maintenance
   - SSL certificate renewal

4. **Backup & Recovery Runbook**
   - Backup verification
   - Restoration procedures
   - Disaster recovery plan

#### 10.3 Update README and Contributing Guides
**Priority:** MEDIUM  
**Time:** 2 hours

**Update:**
- README.md with project overview
- CONTRIBUTING.md with development guidelines
- CODE_OF_CONDUCT.md
- CHANGELOG.md

---

## Success Metrics

### Phase 1-3 (Infrastructure)
- [ ] Application accessible via HTTPS
- [ ] SSL Labs score: A or higher
- [ ] All services running in Docker
- [ ] Nginx properly routing traffic
- [ ] Zero security vulnerabilities in Docker images

### Phase 4-5 (Email & Cloudflare)
- [ ] Contact form successfully sending emails
- [ ] Email deliverability > 95%
- [ ] Spam score < 2/10
- [ ] Cloudflare WAF blocking malicious traffic
- [ ] Page load time < 2 seconds (LCP)

### Phase 6-7 (Microservices & CI/CD)
- [ ] All services independently deployable
- [ ] Zero-downtime deployments working
- [ ] CI/CD pipeline < 5 minutes
- [ ] Test coverage > 70%

### Phase 8-10 (Monitoring & Documentation)
- [ ] Uptime > 99.9%
- [ ] Error rate < 0.1%
- [ ] Mean time to recovery (MTTR) < 15 minutes
- [ ] All documentation complete and reviewed

---

## Cost Estimate

### Monthly Operating Costs

| Service | Tier | Monthly Cost | Annual Cost |
|---------|------|--------------|-------------|
| **Server** (Hetzner CX21) | 2 vCPU, 4GB RAM | €5.83 | €70 |
| **Domain** (Any registrar) | .com | €8-12 | €96-144 |
| **DNS** (Cloudflare free tier) | Optional | $0 | $0 |
| **Email** (Resend) | 50k emails/mo | $20 | $240 |
| **Monitoring** (Sentry) | Free tier | $0 | $0 |
| **Observability** (Prometheus/Grafana) | Self-hosted | $0 | $0 |
| **Analytics** (Plausible) | 10k pageviews | €9 | €108 |
| **Backups** (S3/Spaces) | 100GB | $5 | $60 |
| **Uptime Monitoring** (UptimeRobot) | Free | $0 | $0 |
| **Total (Minimum)** | - | **€48** | **€576** |

**Note:** Cloudflare is optional (DNS only, free tier). No WAF/CDN costs needed.

### One-Time Costs
- Domain registration: €10-15 (first year)
- SSL certificate: $0 (Caddy automatic Let's Encrypt)

---

## Risk Assessment

### High Priority Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Server downtime | Medium | High | Use monitoring, automated failover |
| Data breach | Low | Critical | Security hardening, regular audits |
| Email deliverability issues | Medium | Medium | Proper SPF/DKIM/DMARC, reputable provider |
| DDoS attack | Medium | High | UFW firewall, Caddy rate limiting, optional CrowdSec |
| Deployment failure | Low | High | Staging environment, rollback procedures |

### Mitigation Strategies
1. Implement comprehensive monitoring and alerting
2. Regular security audits and updates
3. Multiple backup strategies
4. Documented incident response procedures
5. Staging environment for testing changes

---

## Next Steps

### Immediate Actions (This Week)
1. ✅ Review and approve this roadmap
2. 📋 Set up project tracking (GitHub Projects or similar)
3. 🔧 Begin Phase 1: Dockerization
4. 🌐 Register/verify domain ownership
5. ☁️ Create Cloudflare account

### Week 1 Deliverables
- [ ] Dockerfile and docker-compose.yml created
- [ ] Local Docker environment working
- [ ] Caddy configured as edge gateway
- [ ] Domain DNS pointed to Cloudflare

### Quick Wins (Can be done in parallel)
- Set up GitHub repository if not already done
- Create accounts with service providers (Cloudflare, email, monitoring)
- Document current architecture and technical decisions
- Set up basic monitoring (UptimeRobot)

---

## Resources

### Documentation Links
- [SvelteKit Deployment](https://kit.svelte.dev/docs/adapter-node)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Caddy Documentation](https://caddyserver.com/docs/)
- [Caddyfile Syntax](https://caddyserver.com/docs/caddyfile)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)

### Useful Tools
- [SSL Labs Test](https://www.ssllabs.com/ssltest/)
- [Security Headers](https://securityheaders.com/)
- [Mail Tester](https://www.mail-tester.com/)
- [MXToolbox](https://mxtoolbox.com/)
- [Lighthouse CI](https://github.com/GoogleChrome/lighthouse-ci)

### Community Support
- [SvelteKit Discord](https://discord.gg/svelte)
- [Docker Community](https://www.docker.com/community/)
- [Caddy Community](https://caddy.community/)

---

## Appendix

### Glossary of Terms
- **API Gateway**: Single entry point for all client requests
- **CDN**: Content Delivery Network
- **CSP**: Content Security Policy
- **DKIM**: DomainKeys Identified Mail
- **DMARC**: Domain-based Message Authentication, Reporting & Conformance
- **SPF**: Sender Policy Framework
- **SSL/TLS**: Secure Sockets Layer / Transport Layer Security
- **WAF**: Web Application Firewall

### Contact Information
For questions about this roadmap:
- Email: [your-email]
- GitHub: [your-github]
- Project Repository: [repo-url]

---

**Document Version:** 2.0  
**Last Review Date:** 2025-01-XX  
**Next Review Date:** 2025-02-XX

