# Security Policy & Compliance

**Version:** 1.0  
**Last Updated:** 2025-10-04  
**Project:** RoboAgency - Humachine AI Studio

---

## Overview

This document outlines the security measures, policies, and procedures to protect the autonomous AI agency infrastructure, data, and services.

**Security Philosophy:** Defense in depth with multiple layers of protection.

---

## Table of Contents

1. [Security Architecture](#security-architecture)
2. [Authentication & Authorization](#authentication--authorization)
3. [Data Protection](#data-protection)
4. [Network Security](#network-security)
5. [Application Security](#application-security)
6. [Infrastructure Security](#infrastructure-security)
7. [Incident Response](#incident-response)
8. [Compliance](#compliance)
9. [Security Checklist](#security-checklist)

---

## Security Architecture

### Defense Layers

```
┌────────────────────────────────────────────────┐
│         UFW Firewall (Layer 1: Perimeter)     │
│         - Ports 22, 80, 443 only              │
│         - Optional: CrowdSec/fail2ban          │
└────────────────────────────────────────────────┘
                     ↓
┌────────────────────────────────────────────────┐
│         Caddy (Layer 2: Edge Gateway)          │
│         - Automatic HTTPS                      │
│         - Security headers                     │
│         - Rate limiting                        │
│         - Host-based routing                  │
└────────────────────────────────────────────────┘
                     ↓
┌────────────────────────────────────────────────┐
│         Application (Layer 3: Logic)           │
│         - Input validation                     │
│         - Authentication                       │
│         - Authorization                        │
└────────────────────────────────────────────────┘
                     ↓
┌────────────────────────────────────────────────┐
│         Data (Layer 4: Storage)                │
│         - Encryption at rest                   │
│         - Access control                       │
│         - Audit logging                        │
└────────────────────────────────────────────────┘
```

---

## Authentication & Authorization

### API Authentication

**Method:** Bearer token (JWT)

```python
from fastapi import Security, HTTPException
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
import jwt

security = HTTPBearer()

def verify_token(credentials: HTTPAuthorizationCredentials = Security(security)):
    try:
        payload = jwt.decode(
            credentials.credentials,
            SECRET_KEY,
            algorithms=["HS256"]
        )
        return payload
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token expired")
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=401, detail="Invalid token")

# Protected endpoint
@app.get("/api/protected")
def protected_route(token: dict = Depends(verify_token)):
    return {"message": "Access granted"}
```

### API Key Management

**Storage:** Environment variables only, never in code or version control.

```bash
# .env.production
API_KEY=<generated-with-cryptographically-secure-random>
OPENAI_API_KEY=sk-...
SUPABASE_SERVICE_KEY=<secret>

# Never commit
git add .env.production  # ❌ NEVER DO THIS
```

**Rotation:** Rotate API keys every 90 days.

```python
# API key rotation script
import secrets

def generate_api_key():
    return secrets.token_urlsafe(32)

new_key = generate_api_key()
# Store in secure location and update .env.production
```

---

## Data Protection

### Encryption at Rest

**Database:**
```sql
-- Enable PostgreSQL encryption (if supported)
ALTER SYSTEM SET wal_encryption = on;

-- Encrypt sensitive columns
CREATE EXTENSION IF NOT EXISTS pgcrypto;

INSERT INTO sensitive_data (email, encrypted_field)
VALUES ('user@example.com', pgp_sym_encrypt('secret', 'encryption_key'));
```

**S3/Object Storage:**
```python
# Server-side encryption (SSE-S3)
s3.upload_file(
    'local_file.txt',
    'bucket-name',
    'key',
    ExtraArgs={'ServerSideEncryption': 'AES256'}
)
```

### Encryption in Transit

**SSL/TLS:**
- Caddy automatically uses TLS 1.2+ (prefers TLS 1.3)
- Strong cipher suites configured automatically
- HSTS enabled by default

**Caddy Configuration:**
Caddy automatically handles TLS with Let's Encrypt. No manual certificate management needed.

```caddyfile
{
  email {$ACME_EMAIL}
  admin 0.0.0.0:2019
}

agency.{$DOMAIN} {
  # Automatic HTTPS with HSTS
  header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
  reverse_proxy frontend:3000
}
```

### Data Retention

| Data Type | Retention | Deletion Method |
|-----------|-----------|-----------------|
| Lead data | 2 years | Soft delete, then purge |
| Project data | 5 years | Archive to cold storage |
| Assets | 2 years | S3 lifecycle policy |
| Logs | 90 days | Automated deletion |
| Backups | 1 year | Encrypted, then deleted |

### PII Handling

**Minimize collection:**
- Only collect necessary PII (email, name)
- No sensitive data (SSN, payment info) stored

**Access control:**
- PII accessible only to authorized services
- Audit all PII access

**User rights:**
- Right to access: Provide all user data
- Right to deletion: Purge all user data
- Right to portability: Export in JSON format

---

## Network Security

### Firewall (UFW)

```bash
# Server firewall configuration
ufw default deny incoming
ufw default allow outgoing

# Allow SSH (key-only)
ufw allow 22/tcp

# Allow HTTP/HTTPS
ufw allow 80/tcp
ufw allow 443/tcp

# Enable firewall
ufw enable
```

### Intrusion Detection (Choose One)

**Option A: CrowdSec (Recommended)**

CrowdSec provides community-driven threat intelligence and automatic remediation.

```bash
# Install CrowdSec
curl -s https://packagecloud.io/install/repositories/crowdsec/crowdsec/script.deb.sh | sudo bash
sudo apt install crowdsec

# Install Caddy bouncer (if using Caddy)
sudo cscli collections install crowdsecurity/caddy

# Enable and start
sudo systemctl enable crowdsec
sudo systemctl start crowdsec
```

**Option B: fail2ban (Alternative)**

```bash
# /etc/fail2ban/jail.local
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600

# Note: Caddy logs may need custom filter configuration
# Ensure logs contain real client IPs (not proxy IPs)
```

### VPN Access (Optional)

For production database access:

```bash
# Install WireGuard
apt install wireguard

# Configure VPN for database access
# Only allow database connections from VPN subnet
```

---

## Application Security

### Input Validation

**Always validate and sanitize user input:**

```python
from pydantic import BaseModel, EmailStr, validator

class IntakeRequest(BaseModel):
    email: EmailStr
    sector: str
    idea: str
    
    @validator('sector')
    def sector_must_be_valid(cls, v):
        allowed = ['healthtech', 'fintech', 'edtech', 'saas', 'ecommerce']
        if v not in allowed:
            raise ValueError(f'sector must be one of {allowed}')
        return v
    
    @validator('idea')
    def idea_length(cls, v):
        if len(v) < 50 or len(v) > 500:
            raise ValueError('idea must be 50-500 characters')
        return v
```

### SQL Injection Prevention

**Use parameterized queries:**

```python
# Good: Parameterized query
cursor.execute(
    "SELECT * FROM leads WHERE email = %s",
    (email,)
)

# Bad: String concatenation ❌
cursor.execute(
    f"SELECT * FROM leads WHERE email = '{email}'"
)
```

### XSS Prevention

**Content Security Policy:**

```caddyfile
agency.{$DOMAIN} {
  header {
    Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' https://trusted-cdn.com; style-src 'self' 'unsafe-inline';"
  }
  reverse_proxy frontend:3000
}
```

**Sanitize output:**

```typescript
// Svelte automatically escapes
<p>{userInput}</p>

// Raw HTML (use sparingly)
<p>{@html sanitizedHtml}</p>
```

### CSRF Protection

```python
from fastapi_csrf import CsrfProtect

csrf = CsrfProtect()

@app.post("/api/intake")
async def create_intake(request: Request, data: IntakeRequest):
    await csrf.validate_csrf(request)
    # Process request
```

### Rate Limiting

**Caddy Rate Limiting:**

Caddy supports rate limiting via plugins or application-level middleware. Recommended approach:

**Option 1: Application-level (Recommended)**
```python
# FastAPI with slowapi
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@app.post("/api/intake")
@limiter.limit("5/hour")
async def create_intake(request: Request):
    # Handle intake
    pass
```

**Option 2: Caddy Rate Limiting Plugin**
Install Caddy rate limiting plugin and configure in Caddyfile. See [Caddy rate limiting docs](https://caddyserver.com/docs/modules/http.handlers.rate_limit).

**Option 3: CrowdSec**
CrowdSec provides advanced rate limiting and DDoS protection as part of its intrusion detection system.

---

## Infrastructure Security

### SSH Hardening

```bash
# /etc/ssh/sshd_config
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2

# Restart SSH
systemctl restart sshd
```

### Docker Security

**Non-root user:**

```dockerfile
# Dockerfile
FROM python:3.11-slim

# Create non-root user
RUN useradd -m -u 1000 appuser

# Switch to non-root
USER appuser

CMD ["python", "app.py"]
```

**Image scanning:**

```bash
# Scan images for vulnerabilities
docker scan roboagency/api:latest

# Use Trivy for scanning
trivy image roboagency/api:latest
```

### Secrets Management

**Environment variables:**

```bash
# Production server only
chmod 600 .env.production
chown deploy:deploy .env.production
```

**Alternative: HashiCorp Vault**

```python
import hvac

client = hvac.Client(url='https://vault.example.com')
client.auth.approle.login(role_id, secret_id)

secret = client.secrets.kv.v2.read_secret_version(path='roboagency/api')
api_key = secret['data']['data']['api_key']
```

---

## Incident Response

### Incident Classification

| Severity | Definition | Response Time |
|----------|-----------|---------------|
| **Critical** | Data breach, service down | Immediate |
| **High** | Security vulnerability, major bug | < 1 hour |
| **Medium** | Performance degradation | < 4 hours |
| **Low** | Minor bug, cosmetic issue | < 24 hours |

### Incident Response Plan

1. **Detection**
   - Monitoring alerts (Sentry, UptimeRobot)
   - User reports
   - Security scans

2. **Triage**
   - Classify severity
   - Assign incident commander
   - Create incident channel (Slack, Discord)

3. **Containment**
   - Stop the breach (block IP, disable service)
   - Preserve evidence (logs, database snapshots)
   - Notify stakeholders

4. **Eradication**
   - Fix vulnerability
   - Deploy patch
   - Verify fix

5. **Recovery**
   - Restore services
   - Monitor for recurrence
   - Update documentation

6. **Post-Mortem**
   - Document timeline
   - Root cause analysis
   - Action items to prevent recurrence

### Contact Information

```
Security Team Lead: security@youragency.com
On-Call Engineer: +1-555-123-4567
Incident Response: https://status.youragency.com
```

### Breach Notification

**Timeline:**
- Internal notification: Immediate
- Customer notification: Within 72 hours (GDPR requirement)
- Public disclosure: As required by law

---

## Compliance

### GDPR (if serving EU users)

- [ ] Data processing agreement
- [ ] Privacy policy published
- [ ] Cookie consent banner
- [ ] Data export functionality
- [ ] Right to deletion implemented
- [ ] Data retention policy

### Security Audits

**Schedule:**
- Quarterly security reviews
- Annual penetration testing
- Continuous vulnerability scanning

**Tools:**
- OWASP ZAP (automated scanning)
- Burp Suite (manual testing)
- Nmap (network scanning)

---

## Security Checklist

### Pre-Launch

- [ ] All secrets stored securely (not in code)
- [ ] HTTPS enforced with valid certificate
- [ ] Security headers configured
- [ ] Input validation on all endpoints
- [ ] SQL injection prevention verified
- [ ] XSS protection enabled
- [ ] CSRF protection enabled
- [ ] Rate limiting configured
- [ ] Firewall enabled (UFW)
- [ ] CrowdSec or fail2ban configured
- [ ] SSH hardened (key-only)
- [ ] Automated backups configured
- [ ] Monitoring and alerting active
- [ ] Incident response plan documented
- [ ] Security contact published

### Quarterly Reviews

- [ ] Rotate API keys
- [ ] Update dependencies
- [ ] Review access logs
- [ ] Audit user permissions
- [ ] Test backup restoration
- [ ] Vulnerability scan
- [ ] Review firewall rules
- [ ] Update security documentation

### Ongoing

- [ ] Monitor security alerts
- [ ] Apply security patches within 7 days
- [ ] Review logs weekly
- [ ] Test incident response procedures
- [ ] Stay informed on security threats

---

## Reporting Security Issues

If you discover a security vulnerability, please email:

**security@youragency.com**

**DO NOT:**
- Open a public GitHub issue
- Post on social media
- Share with third parties

**We will:**
- Acknowledge within 24 hours
- Provide status updates
- Credit you (if desired) after fix is deployed

---

## Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)

---

**Document Version:** 2.0  
**Last Review Date:** 2025-01-XX  
**Next Review Date:** 2025-04-XX (Quarterly)

