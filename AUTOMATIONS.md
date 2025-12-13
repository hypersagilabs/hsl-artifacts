# Automations & n8n Workflows

**Version:** 1.0  
**Last Updated:** 2025-10-04  
**Project:** RoboAgency - Humachine AI Studio

---

## Overview

This document defines all automated workflows powered by n8n, the workflow automation platform. These workflows handle webhook ingestion, email campaigns, content distribution, and monitoring.

---

## Table of Contents

1. [n8n Setup](#n8n-setup)
2. [Workflow: Lead Warmup](#workflow-lead-warmup)
3. [Workflow: Content Distribution](#workflow-content-distribution)
4. [Workflow: Monitoring & Alerts](#workflow-monitoring--alerts)
5. [Email Configuration](#email-configuration)
6. [Integration Testing](#integration-testing)

---

## n8n Setup

### Docker Configuration

```yaml
# docker-compose.yml
services:
  n8n:
    image: n8nio/n8n:latest
    restart: unless-stopped
    ports:
      - "8080:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=${N8N_PASSWORD}
      - N8N_HOST=${N8N_HOST}
      - N8N_PROTOCOL=https
      - NODE_ENV=production
      - WEBHOOK_URL=https://webhooks.yourdomain.tld
    volumes:
      - ./data/n8n:/home/node/.n8n
    depends_on:
      - postgres
```

### Environment Variables

```bash
# .env.production
N8N_HOST=n8n.yourdomain.tld
N8N_PASSWORD=<strong_password>
WEBHOOK_URL=https://webhooks.yourdomain.tld
```

### Access

- **URL:** https://n8n.yourdomain.tld
- **Credentials:** Set in environment variables
- **Webhook Base:** https://webhooks.yourdomain.tld/webhook/

---

## Workflow: Lead Warmup

### Purpose

Automatically send a 3-email sequence after a lead completes the intake form and agents generate demo assets.

### Trigger

Webhook: `POST /webhook/lead_ready`

**Payload:**
```json
{
  "project_id": "abc123",
  "lead_email": "user@example.com",
  "lead_name": "John Doe",
  "demo_url": "https://assets.yourdomain.tld/videos/abc123.mp4",
  "proto_url": "https://assets.yourdomain.tld/prototypes/abc123/",
  "calendar_url": "https://cal.com/youragency"
}
```

### Workflow Diagram

```
Webhook Trigger
    ↓
Extract Data
    ↓
Send Email #1 (Demo)
    ↓
Wait 24 hours
    ↓
Send Email #2 (Prototype)
    ↓
Wait 48 hours
    ↓
Send Email #3 (Calendar)
```

### n8n JSON Export

```json
{
  "name": "Lead Warmup Campaign",
  "nodes": [
    {
      "parameters": {
        "path": "lead_ready",
        "responseMode": "responseNode",
        "options": {}
      },
      "name": "Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 1,
      "position": [250, 300]
    },
    {
      "parameters": {
        "functionCode": "// Extract and validate payload\nconst payload = $json.body;\n\nif (!payload.lead_email || !payload.demo_url) {\n  throw new Error('Missing required fields');\n}\n\nreturn [\n  {\n    json: {\n      email: payload.lead_email,\n      name: payload.lead_name || 'there',\n      demo_url: payload.demo_url,\n      proto_url: payload.proto_url,\n      calendar_url: payload.calendar_url,\n      project_id: payload.project_id\n    }\n  }\n];"
      },
      "name": "Extract & Validate",
      "type": "n8n-nodes-base.function",
      "typeVersion": 1,
      "position": [450, 300]
    },
    {
      "parameters": {
        "fromEmail": "hello@youragency.com",
        "toEmail": "={{ $json.email }}",
        "subject": "Your Custom Demo is Ready! 🎉",
        "text": "Hi {{ $json.name }},\n\nThanks for sharing your idea with us! We've created a custom demo video that shows how your vision could come to life.\n\n🎬 Watch your demo: {{ $json.demo_url }}\n\nThis video walks through the key features and user experience. Take a look and let us know what you think!\n\nBest regards,\nThe Team",
        "options": {
          "allowUnauthorizedCerts": false
        }
      },
      "name": "Email #1 - Demo Video",
      "type": "n8n-nodes-base.emailSend",
      "typeVersion": 2,
      "position": [650, 300],
      "credentials": {
        "smtp": {
          "id": "1",
          "name": "Resend SMTP"
        }
      }
    },
    {
      "parameters": {
        "amount": 24,
        "unit": "hours"
      },
      "name": "Wait 24 Hours",
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1,
      "position": [850, 300],
      "webhookId": "auto-generated"
    },
    {
      "parameters": {
        "fromEmail": "hello@youragency.com",
        "toEmail": "={{ $json.email }}",
        "subject": "Try Your Interactive Prototype",
        "text": "Hi {{ $json.name }},\n\nWant to see your idea in action? We've built an interactive prototype that you can click through and explore.\n\n🚀 Try it here: {{ $json.proto_url }}\n\nFeel free to share it with your team and get their feedback. It's a real working demo!\n\nCheers,\nThe Team",
        "options": {}
      },
      "name": "Email #2 - Prototype",
      "type": "n8n-nodes-base.emailSend",
      "typeVersion": 2,
      "position": [1050, 300],
      "credentials": {
        "smtp": {
          "id": "1",
          "name": "Resend SMTP"
        }
      }
    },
    {
      "parameters": {
        "amount": 48,
        "unit": "hours"
      },
      "name": "Wait 48 Hours",
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1,
      "position": [1250, 300],
      "webhookId": "auto-generated"
    },
    {
      "parameters": {
        "fromEmail": "hello@youragency.com",
        "toEmail": "={{ $json.email }}",
        "subject": "Let's Build This Together",
        "text": "Hi {{ $json.name }},\n\nIf you're ready to turn this vision into reality, let's chat!\n\n📅 Book a 30-minute call: {{ $json.calendar_url }}\n\nWe'll discuss:\n• Timeline and budget\n• Technical approach\n• Team and resources\n• Next steps\n\nLooking forward to working together!\n\nBest,\nThe Team",
        "options": {}
      },
      "name": "Email #3 - Schedule Call",
      "type": "n8n-nodes-base.emailSend",
      "typeVersion": 2,
      "position": [1450, 300],
      "credentials": {
        "smtp": {
          "id": "1",
          "name": "Resend SMTP"
        }
      }
    },
    {
      "parameters": {
        "operation": "update",
        "tableId": "projects",
        "columns": "status",
        "updateKey": "id",
        "options": {
          "queryName": "id"
        }
      },
      "name": "Update Project Status",
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [1650, 300],
      "credentials": {
        "supabaseApi": {
          "id": "2",
          "name": "Supabase"
        }
      }
    },
    {
      "parameters": {
        "respondWith": "json",
        "responseBody": "={{ {\"success\": true, \"project_id\": $json.project_id} }}"
      },
      "name": "Respond to Webhook",
      "type": "n8n-nodes-base.respondToWebhook",
      "typeVersion": 1,
      "position": [450, 500]
    }
  ],
  "connections": {
    "Webhook": {
      "main": [
        [
          {
            "node": "Extract & Validate",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Extract & Validate": {
      "main": [
        [
          {
            "node": "Email #1 - Demo Video",
            "type": "main",
            "index": 0
          },
          {
            "node": "Respond to Webhook",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Email #1 - Demo Video": {
      "main": [
        [
          {
            "node": "Wait 24 Hours",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Wait 24 Hours": {
      "main": [
        [
          {
            "node": "Email #2 - Prototype",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Email #2 - Prototype": {
      "main": [
        [
          {
            "node": "Wait 48 Hours",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Wait 48 Hours": {
      "main": [
        [
          {
            "node": "Email #3 - Schedule Call",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Email #3 - Schedule Call": {
      "main": [
        [
          {
            "node": "Update Project Status",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}
```

### Testing

```bash
# Test webhook locally
curl -X POST https://webhooks.yourdomain.tld/webhook/lead_ready \
  -H "Content-Type: application/json" \
  -d '{
    "project_id": "test123",
    "lead_email": "test@example.com",
    "lead_name": "Test User",
    "demo_url": "https://example.com/demo.mp4",
    "proto_url": "https://example.com/proto/",
    "calendar_url": "https://cal.com/test"
  }'
```

---

## Workflow: Content Distribution

### Purpose

Automatically distribute blog content to multiple platforms (Twitter, LinkedIn, Instagram, YouTube, Newsletter).

### Trigger

Webhook: `POST /webhook/cms/publish`

**Payload:**
```json
{
  "post_id": "xyz789",
  "title": "How AI Agents Transform Development",
  "url": "https://blog.youragency.com/ai-agents-dev",
  "excerpt": "A deep dive into...",
  "content": "<full HTML>",
  "tags": ["AI", "Development", "Automation"]
}
```

### Workflow Diagram

```
CMS Publish Webhook
    ↓
Extract Content
    ↓
Call LLM (Adapt for Platforms)
    ↓
    ├─→ Schedule Twitter Thread
    ├─→ Schedule LinkedIn Post
    ├─→ Schedule Instagram Post
    ├─→ Schedule YouTube Short
    └─→ Send Newsletter
```

### n8n JSON Export (Simplified)

```json
{
  "name": "Content Omnichannel Distribution",
  "nodes": [
    {
      "parameters": {
        "path": "cms/publish"
      },
      "name": "CMS Webhook",
      "type": "n8n-nodes-base.webhook"
    },
    {
      "parameters": {
        "functionCode": "const post = $json.body.post;\nconst prompt = `Adapt this blog post for multiple platforms:\n\nTitle: ${post.title}\nExcerpt: ${post.excerpt}\nURL: ${post.url}\n\nCreate:\n1. Twitter thread (5 tweets, 280 chars each)\n2. LinkedIn post (professional, 1300 chars)\n3. Instagram caption (engaging, with hashtags)\n4. YouTube Shorts description (150 chars)\n\nReturn as JSON.`;\n\nreturn [{json: {post, prompt}}];"
      },
      "name": "Prepare LLM Prompt"
    },
    {
      "parameters": {
        "model": "gpt-4",
        "messages": {
          "values": [
            {
              "role": "user",
              "content": "={{ $json.prompt }}"
            }
          ]
        },
        "options": {
          "temperature": 0.7
        }
      },
      "name": "OpenAI - Adapt Content",
      "type": "n8n-nodes-base.openAi"
    },
    {
      "parameters": {
        "functionCode": "const adaptations = JSON.parse($json.choices[0].message.content);\nreturn [{json: {...$json.post, ...adaptations}}];"
      },
      "name": "Parse Adaptations"
    },
    {
      "parameters": {},
      "name": "Twitter Thread",
      "type": "n8n-nodes-base.twitter"
    },
    {
      "parameters": {},
      "name": "LinkedIn Post",
      "type": "n8n-nodes-base.linkedIn"
    },
    {
      "parameters": {
        "fromEmail": "newsletter@youragency.com",
        "toEmail": "={{ $json.subscriber_emails }}",
        "subject": "={{ $json.title }}",
        "html": "={{ $json.content }}"
      },
      "name": "Send Newsletter"
    }
  ],
  "connections": {
    "CMS Webhook": {"main": [[{"node": "Prepare LLM Prompt"}]]},
    "Prepare LLM Prompt": {"main": [[{"node": "OpenAI - Adapt Content"}]]},
    "OpenAI - Adapt Content": {"main": [[{"node": "Parse Adaptations"}]]},
    "Parse Adaptations": {"main": [
      [{"node": "Twitter Thread"}],
      [{"node": "LinkedIn Post"}],
      [{"node": "Send Newsletter"}]
    ]}
  }
}
```

---

## Workflow: Monitoring & Alerts

### Purpose

Health check all services and send alerts on failures.

### Trigger

Schedule: Every 5 minutes

### Workflow Diagram

```
Schedule Trigger (5 min)
    ↓
    ├─→ Check Frontend (GET /)
    ├─→ Check API (GET /health)
    ├─→ Check n8n (GET /healthz)
    └─→ Check Database (SELECT 1)
    ↓
Evaluate Results
    ↓
If Any Failed → Send Alert (Email/Slack)
```

### n8n JSON Export (Simplified)

```json
{
  "name": "Service Health Monitoring",
  "nodes": [
    {
      "parameters": {
        "rule": {
          "interval": [
            {
              "field": "minutes",
              "minutesInterval": 5
            }
          ]
        }
      },
      "name": "Schedule",
      "type": "n8n-nodes-base.scheduleTrigger"
    },
    {
      "parameters": {
        "url": "https://yourdomain.tld",
        "options": {
          "timeout": 5000
        }
      },
      "name": "Check Frontend"
    },
    {
      "parameters": {
        "url": "https://api.yourdomain.tld/health"
      },
      "name": "Check API"
    },
    {
      "parameters": {
        "conditions": {
          "boolean": [
            {
              "value1": "={{ $json.statusCode }}",
              "operation": "equal",
              "value2": 200
            }
          ]
        }
      },
      "name": "If Failed"
    },
    {
      "parameters": {
        "fromEmail": "alerts@youragency.com",
        "toEmail": "admin@youragency.com",
        "subject": "🚨 Service Down Alert",
        "text": "Service health check failed:\n\n{{ $json.error }}"
      },
      "name": "Send Alert Email"
    }
  ]
}
```

---

## Email Configuration

### Resend SMTP Setup

**Credentials (n8n):**
- Host: `smtp.resend.com`
- Port: `587`
- User: `resend`
- Password: `re_xxxxxxxxxxxxx` (API key)
- From: `hello@youragency.com`

### DNS Records (Any Provider)

**SPF:**
```
TXT @ "v=spf1 include:_spf.google.com include:_spf.resend.com ~all"
```

**Note:** Adjust SPF record based on your email provider. If using Google Workspace, use `include:_spf.google.com`. If using Cloudflare Email Routing, use `include:_spf.mx.cloudflare.net`.

**DKIM:**
```
TXT resend._domainkey "v=DKIM1; k=rsa; p=MIGfMA0GCSq..."
```

**DMARC:**
```
TXT _dmarc "v=DMARC1; p=quarantine; rua=mailto:dmarc@youragency.com; pct=100; adkim=s; aspf=s"
```

### Email Templates

Store templates in `/apps/automation_service/templates/`:

```
templates/
├── lead_demo.html
├── lead_prototype.html
├── lead_calendar.html
└── newsletter_base.html
```

---

## Integration Testing

### Test Lead Warmup Workflow

```python
# tests/test_n8n_workflows.py
import httpx
import pytest

@pytest.mark.asyncio
async def test_lead_warmup_webhook():
    async with httpx.AsyncClient() as client:
        response = await client.post(
            "https://webhooks.yourdomain.tld/webhook/lead_ready",
            json={
                "project_id": "test123",
                "lead_email": "test@example.com",
                "lead_name": "Test User",
                "demo_url": "https://example.com/demo.mp4",
                "proto_url": "https://example.com/proto/",
                "calendar_url": "https://cal.com/test"
            }
        )
        
        assert response.status_code == 200
        data = response.json()
        assert data["success"] is True
        assert data["project_id"] == "test123"
```

### Manual Testing Checklist

- [ ] Webhook receives POST data correctly
- [ ] Email #1 sent immediately
- [ ] Email #2 sent after 24 hours
- [ ] Email #3 sent after 72 hours total
- [ ] All emails rendered correctly
- [ ] Links work and are not broken
- [ ] Project status updated in database

---

## Backup & Export

### Export All Workflows

```bash
# From n8n UI: Settings → Export
# Or via API:
curl -X GET https://n8n.yourdomain.tld/rest/workflows \
  -u admin:${N8N_PASSWORD} \
  -o workflows_backup.json
```

### Import Workflows

```bash
# Via UI: Settings → Import
# Or programmatically via API
```

---

## Next Steps

1. **Import** workflow JSON files into n8n
2. **Configure** SMTP credentials
3. **Test** webhooks with sample data
4. **Monitor** execution logs
5. **Optimize** timing based on engagement data

---

**Document Version:** 1.0  
**Last Review Date:** 2025-10-04  
**Next Review Date:** 2025-11-04

