# Quick Start Guide - HyperSagi Labs Deployment

**TL;DR Version of the Deployment Roadmap**

---

## The Right Order to Do Things

### 🎯 Phase 1: Get Docker Working (Week 1, Days 1-2)
**What:** Containerize your app  
**Why:** Makes everything else easier

```bash
# This is your first priority
1. Create Dockerfile
2. Create docker-compose.yml
3. Test locally: docker compose up
```

**Done when:** You can access your site at `http://localhost` via Docker

---

### 🌐 Phase 2: Get Your Domain (Week 1, Day 3)
**What:** Register domain + configure DNS  
**Why:** DNS takes time to propagate (24-48 hours)

```bash
# Do this ASAP - it takes time to propagate
1. Buy domain (any registrar)
2. Configure DNS A records for subdomains:
   - agency.yourdomain.tld → your-server-ip
   - api.yourdomain.tld → your-server-ip
   - automations.yourdomain.tld → your-server-ip
3. Wait 24-48 hours for propagation
```

**Note:** Cloudflare is optional (DNS only, free tier). Any DNS provider works.

**Done when:** DNS A records resolve correctly

---

### 🖥️ Phase 3: Get a Server (Week 1, Days 4-5)
**What:** Provision a server  
**Why:** Need somewhere to host your app

**Recommended:**
- Hetzner CX21: €5.83/mo (best value)
- DigitalOcean Droplet: $12/mo (easier)

```bash
# Once you have server
1. SSH into server
2. Install Docker
3. Set up firewall (UFW)
4. Deploy your Docker app
```

**Done when:** You can SSH into your server and run Docker

---

### 🔒 Phase 4: Set Up SSL (Week 1, Day 5)
**What:** Enable HTTPS  
**Why:** Required for production

**Caddy automatic HTTPS:**
1. Set `ACME_EMAIL` in `.env.production`
2. Ensure DNS A records are configured
3. Start Caddy - it automatically:
   - Obtains Let's Encrypt certificates
   - Renews certificates automatically
   - Handles HTTP to HTTPS redirects

**No manual certificate management needed!**

**Done when:** Your site loads with `https://` automatically

---

### 📧 Phase 5: Set Up Email (Week 2, Days 1-2)
**What:** Configure email for contact form  
**Why:** Your contact form needs to work

**Recommended:** Resend.com (3,000 emails/month free)

```bash
1. Sign up for Resend
2. Verify domain
3. Get SMTP credentials
4. Add to .env.production
5. Add DNS records (SPF, DKIM, DMARC)
```

**Done when:** Contact form successfully sends emails

---

### 🛡️ Phase 6: Lock Down Security (Week 2, Days 3-4)
**What:** Server-level security  
**Why:** Protect against attacks

```bash
1. Configure UFW firewall (ports 22, 80, 443 only)
2. Harden SSH (keys only, disable password auth)
3. Set up CrowdSec or fail2ban (optional but recommended)
4. Configure security headers in Caddyfile
5. Set up application-level rate limiting
```

**Done when:** Security score is A+ on securityheaders.com

---

### 🏗️ Phase 7: Microservices (Week 3) - OPTIONAL FOR NOW
**What:** Split app into smaller services  
**Why:** Better organization and scalability

**You can skip this initially** and come back to it later.

```bash
# Only do this if you need it
1. Create separate service for contact form
2. Create separate service for email
3. Use Caddy for host-based routing (agency.domain, api.domain)
```

**Done when:** Each service runs independently

---

### 🚀 Phase 8: CI/CD (Week 3-4)
**What:** Automated deployments  
**Why:** Deploy faster with less risk

```bash
1. Set up GitHub Actions
2. Create staging environment
3. Automate Docker builds
4. Automate deployments
```

**Done when:** Git push automatically deploys to server

---

### 📊 Phase 9: Monitoring (Week 4)
**What:** Know when things break  
**Why:** Sleep better at night

**Free tools:**
- UptimeRobot (uptime monitoring)
- Sentry (error tracking)
- Prometheus + Grafana (self-hosted metrics)

**Done when:** You get alerted before users complain

---

## Absolute Minimum to Go Live

If you need to launch ASAP, here's the bare minimum:

```
Week 1:
✅ Dockerize app (Days 1-2)
✅ Buy domain + Cloudflare setup (Day 3)
✅ Get server (Day 4)
✅ Deploy + SSL (Day 5)

Week 2:
✅ Email setup (Days 1-2)
✅ Basic security (Days 3-4)
✅ Test everything (Day 5)

= LIVE! 🎉
```

Everything else can be added incrementally.

---

## What NOT to Do

❌ Don't start with microservices  
❌ Don't overcomplicate the first deployment  
❌ Don't skip SSL/HTTPS  
❌ Don't forget to back up  
❌ Don't expose internal services (Redis, Qdrant, Prometheus) to the internet

---

## Cost Breakdown (Realistic)

**Minimum viable production:**
- Server: €5.83/mo (Hetzner)
- Domain: €10/year
- Email: $0 (Resend free tier)
- DNS: $0 (any provider, Cloudflare free tier optional)
- **Total: €6/mo + €10/year = €82/year**

**Recommended production:**
- Server: €5.83/mo
- Domain: €10/year
- Email: $20/mo (Resend paid)
- Analytics: €9/mo (Plausible)
- **Total: €35/mo = €420/year**

**Note:** No Cloudflare Pro needed - Caddy handles SSL/TLS automatically.

---

## Emergency Contacts / Resources

- **Server down?** Check UptimeRobot alerts
- **SSL issues?** Test at ssllabs.com
- **Email not working?** Test at mail-tester.com
- **Need help?** SvelteKit Discord, /r/docker, /r/selfhosted

---

## Your First Day Checklist

Day 1 (Today):
- [ ] Read full roadmap (artifacts/DEPLOYMENT_ROADMAP.md)
- [ ] Create Dockerfile
- [ ] Create docker-compose.yml
- [ ] Test Docker locally
- [ ] Start domain registration
- [ ] Choose DNS provider (any provider works)

Day 2:
- [ ] Finish Docker configuration
- [ ] Set up Caddyfile
- [ ] Test full stack locally
- [ ] Research server providers
- [ ] Sign up for Resend (email)

By end of Week 1:
- [ ] Site live on your domain with HTTPS
- [ ] Contact form working
- [ ] Basic monitoring set up

---

**Remember:** Perfect is the enemy of done. Launch with the basics, then improve incrementally.

Good luck! 🚀

