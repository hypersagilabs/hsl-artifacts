# Website Change Log - Agency Launch Hardening

Date: 2026-03-03
Scope: `/home/spirimi/hsl-website/apps/frontend`
Purpose: single reference for what was changed during launch prep, why it changed, and what can be safely reversed later.

## How to use this file
- Use this as the operational map instead of digging through commit history.
- Each item includes:
  - what changed
  - why
  - current state
  - whether to reverse later
  - exactly where to change it

---

## 1) Content Visibility Controls (toggles)

### Homepage social proof / placeholder-proof content hidden
- Why: avoid showing generated/placeholder social proof and metrics before real client proof is ready.
- Current state: hidden.
- Reverse later: yes, when real testimonials/metrics are available.

Where:
- `apps/frontend/src/routes/+page.svelte:126`
  - `showTrustBanner = false`
- `apps/frontend/src/routes/+page.svelte:129`
  - `showCaseStudyLinks = false`
- `apps/frontend/src/routes/+page.svelte:131`
  - `showTestimonials = false`
- `apps/frontend/src/routes/+page.svelte:132`
  - `showMetrics = false`

### Services page social proof/case-study links hidden
- Why: same reason as homepage.
- Current state: hidden.
- Reverse later: yes.

Where:
- `apps/frontend/src/routes/services/+page.svelte:200`
  - `showCaseStudyLinks = false`
- `apps/frontend/src/routes/services/+page.svelte:202`
  - `showTestimonials = false`

### Industry-page case-study proof blocks hidden
- Why: industry proof blocks referenced generated case studies.
- Current state: hidden.
- Reverse later: yes, when real case studies are live.

Where:
- `apps/frontend/src/routes/industries/e-commerce/+page.svelte:72`
  - `showCaseStudyProofPoint = false`
- `apps/frontend/src/routes/industries/healthcare/+page.svelte:72`
  - `showCaseStudyProofPoint = false`
- `apps/frontend/src/routes/industries/legal/+page.svelte:72`
  - `showCaseStudyProofPoint = false`
- `apps/frontend/src/routes/industries/manufacturing/+page.svelte:72`
  - `showCaseStudyProofPoint = false`

### Header resources nav hidden
- Why: resources section not launch-ready.
- Current state: hidden.
- Reverse later: yes, when resources are fully ready.

Where:
- `apps/frontend/src/routes/+layout.svelte:30`
  - `showResourcesInNav = false`

### Footer links hidden (blog/case-studies/trust badges)
- Why: prevent unfinished content and premature trust badges.
- Current state: blog and case studies hidden in footer; trust badges hidden.
- Reverse later: yes.

Where:
- `apps/frontend/src/lib/components/Footer.svelte:3`
  - `showCaseStudiesInFooter = false`
- `apps/frontend/src/lib/components/Footer.svelte:5`
  - `showBlogInFooter = false`
- `apps/frontend/src/lib/components/Footer.svelte:7`
  - `showFooterTrustBadges = false`

### 404 page: resource cards suppressed/replaced
- Why: avoid dead-end links on error path.
- Current state: case studies hidden; blog card replaced with contact CTA.
- Reverse later: maybe.

Where:
- `apps/frontend/src/routes/+error.svelte:6`
  - `showCaseStudyLinks = false`
- `apps/frontend/src/routes/+error.svelte:8`
  - `showBlogLink = false`

---

## 2) Resource Routing and Indexing Behavior

### Global resources gating with FAQ exception
- Why: keep unfinished resources hidden while keeping FAQ publicly available.
- Current state:
  - `/resources/faq` is live
  - all other `/resources/*` redirect to `/` with `307`
- Reverse later: yes (once resources are ready, remove/relax gate).

Where:
- `apps/frontend/src/hooks.server.ts:14`
  - `if (path.startsWith('/resources') && !path.startsWith('/resources/faq'))`
- `apps/frontend/src/hooks.server.ts:20`
  - sets `x-robots-tag: noindex, nofollow` on redirected resource requests.

### Layout robots handling updated for FAQ exception
- Why: prevent hidden resources from indexing while allowing FAQ indexing.
- Current state: noindex for resources except FAQ.
- Reverse later: yes.

Where:
- `apps/frontend/src/routes/+layout.svelte:58`
  - `page.url.pathname.startsWith('/resources') && !page.url.pathname.startsWith('/resources/faq')`

---

## 3) Copy and Messaging Changes

### Homepage hero copy generalized beyond regulated-only
- Why: messaging needed to appeal to both regulated and creative/dynamic industries.
- Current state: broadened positioning.

Key text:
- `apps/frontend/src/routes/+page.svelte:161`
  - `For Dynamic & Regulated Industries`
- `apps/frontend/src/routes/+page.svelte:165`
  - `The intelligent AI systems your forward-thinking business can't buy off the shelf`

### Homepage trust/compliance paragraph softened
- Why: avoid over-claiming compliance as universal requirement.
- Current state: broader language with conditional compliance guardrails.

### Footer brand line generalized
- Why: avoid regulated-only framing.
- Current state:
  - `Production-grade intelligent systems for dynamic and regulated industries. From pilot to production in weeks.`

Where:
- `apps/frontend/src/lib/components/Footer.svelte:20`

---

## 4) CTA Rework to Avoid Hidden Resources

Replaced resource/download/blog CTAs with contact/book-call variants (intentionally varied wording).

Where:
- `apps/frontend/src/routes/industries/legal/+page.svelte:244`
  - `Schedule a legal workflow call`
- `apps/frontend/src/routes/industries/property/+page.svelte:213`
  - `Book a property systems call`
- `apps/frontend/src/routes/industries/accountancy/+page.svelte:213`
  - `Talk through your month-end bottlenecks`
- `apps/frontend/src/routes/industries/education/+page.svelte:213`
  - `Book a call about admissions automation`
- `apps/frontend/src/routes/industries/creative/+page.svelte:213`
  - `Book a creative ops strategy call`
- `apps/frontend/src/routes/+error.svelte:57`
  - `Book a Discovery Call`

Reverse later: optional. These are launch-safe and can remain even after resources return.

---

## 5) Layout/Visual Adjustments

### Homepage industry cards rebalanced
- Why: last row looked left-heavy.
- Current state: `lg:grid-cols-4` for balanced two-row grid.

Where:
- `apps/frontend/src/routes/+page.svelte` industries section grid class.

Reverse later: usually no.

### Services benefits final card centered
- Why: “Outcomes, not hours” card looked off-balance in wrap state.
- Current state: last item gets centered positioning classes.

Where:
- `apps/frontend/src/routes/services/+page.svelte:225`
  - `class:lg:col-start-2={i === benefits.length - 1}`

Reverse later: no.

---

## 6) Domain and Metadata Normalization

### Canonical/OG/JSON-LD moved to agency domain
- Why: active client-facing product is `agency.hypersagi.com`.
- Current state: metadata now uses `https://agency.hypersagi.com`.

Where:
- `apps/frontend/src/lib/components/SEO.svelte`
- `apps/frontend/src/routes/+layout.svelte`
- `apps/frontend/src/routes/resources/blog/[slug]/+page.svelte:462`

### Removed `hypersagilabs.com` usage
- Why: domain not owned.
- Current state: replaced with `agency.hypersagi.com` and `hello@hypersagi.com` defaults.

Where:
- `apps/frontend/src/lib/server/email.ts`
- `apps/frontend/src/routes/contact/+page.server.ts:26`

---

## 7) Email Flow Changes

### Contact form behavior (unchanged in logic, updated in links/defaults)
- Sends internal notification to `CONTACT_EMAIL` (fallback `hello@hypersagi.com`).
- Sends auto-reply to submitter.
- Auto-reply links now target `https://agency.hypersagi.com`.

Where:
- `apps/frontend/src/routes/contact/+page.server.ts`
- `apps/frontend/src/lib/server/email.ts`

Reverse later: probably no.

---

## 8) Quick Reactivation Playbook (when content matures)

Recommended order:
1. Real testimonials ready:
   - set `showTestimonials = true` in:
     - `routes/+page.svelte`
     - `routes/services/+page.svelte`
2. Real case studies ready:
   - set `showCaseStudyProofPoint = true` in 4 industry pages
   - set `showCaseStudyLinks = true` in homepage/services/404
   - set `showCaseStudiesInFooter = true`
3. Blog ready:
   - set `showBlogInFooter = true`
   - if blog/resources should be public, remove resources redirect gate in `hooks.server.ts`
4. Full resources launch:
   - set `showResourcesInNav = true`
   - remove resource noindex exceptions as needed in `+layout.svelte`
5. Trust badges/legal proofs verified:
   - set `showFooterTrustBadges = true`

---

## 9) Keep vs Reverse Decision Matrix

Keep as-is now:
- balanced card layouts
- generalized hero/brand positioning
- contact-first CTA replacements
- agency-domain metadata alignment

Reverse later (content maturity dependent):
- testimonials hidden state
- case studies hidden state
- blog hidden state
- resources nav hidden state
- resources redirect gate (except FAQ currently)
- footer trust badges hidden state
- homepage trust banner/metrics hidden state

---

## 10) Verification Commands (copy/paste)

Route status sweep:
```bash
for p in / /about /services /industries /contact /resources/faq /resources/blog /resources/downloads /resources/case-studies /this-page-does-not-exist; do
  code=$(curl -sk -o /dev/null -w '%{http_code}' --resolve agency.hypersagi.com:443:127.0.0.1 "https://agency.hypersagi.com$p")
  printf '%-28s %s\n' "$p" "$code"
done
```

Check hidden-resource links on public pages:
```bash
for p in / /about /services /industries /industries/legal /industries/healthcare /industries/e-commerce /industries/manufacturing /industries/accountancy /industries/property /industries/education /industries/creative /contact /this-page-does-not-exist; do
  html=$(curl -sk --resolve agency.hypersagi.com:443:127.0.0.1 "https://agency.hypersagi.com$p")
  if echo "$html" | rg -q 'href="/resources/(blog|downloads|case-studies)'; then
    echo "$p: BAD"
  else
    echo "$p: OK"
  fi
done
```

Check metadata domain consistency:
```bash
curl -sk --resolve agency.hypersagi.com:443:127.0.0.1 https://agency.hypersagi.com/ \
  | rg -n 'canonical|og:url|agency\.hypersagi\.com|hypersagi\.com|hypersagilabs\.com'
```

---

## 11) Operator Checklist (embedded)

Use this before/after major copy or visibility changes.

Per-change checklist:
- update code
- rebuild container: `docker compose -f /home/spirimi/hsl-website/infra/docker/compose.yml -p hsl-core up -d --build`
- route sweep (Section 10)
- hidden-link sweep (Section 10)
- metadata domain check (Section 10)
- manual visual spot-check on `/`, `/services`, one industry page, `/contact`, 404 page

Daily launch checklist:
- homepage loads (`200`) and primary CTA goes to `/contact`
- `/resources/faq` is `200`
- `/resources/blog`, `/resources/downloads`, `/resources/case-studies` remain `307` (until intentionally opened)
- contact form validation still works
- footer shows no unfinished links (blog/case studies) unless intentionally re-enabled

Weekly checklist:
- claims audit: ensure visible stats/case claims are still evidence-backed
- link audit: no accidental links to hidden resources
- domain audit: canonicals/OG/JSON-LD still on `agency.hypersagi.com`
- email audit: submission + auto-reply still deliver to inbox (not spam)

---

## 12) Reactivation Criteria (do-not-enable-until)

Testimonials:
- At least 2 approved client quotes with name/title/company or agreed anonymization
- Replace current placeholder text before toggling on

Case studies:
- Real writeups complete (problem, scope, measured outcome, permission status)
- Slugs and links verified

Blog:
- At least 3 publish-ready posts
- Internal links audited (no broken links, no hidden-resource dead ends)

Trust badges/compliance claims:
- Only enable claims that are currently true and verifiable

---

## 13) Rollback and Recovery

If a release introduces regressions:
- revert only the affected toggles first (fastest path)
- rebuild container
- rerun Section 10 checks

If resources accidentally open:
- restore resources gate in `hooks.server.ts`
- ensure nav/footer resource toggles are still off
- rebuild and verify 307 behavior

If domain metadata drifts:
- reset base URLs in:
  - `lib/components/SEO.svelte`
  - `routes/+layout.svelte`
  - `routes/resources/blog/[slug]/+page.svelte`

---

## 14) Ownership and Decision Log

Current product surface:
- public product site: `agency.hypersagi.com`
- temporary apex redirect: `hypersagi.com` -> `agency.hypersagi.com`
- `hypersagilabs.com` is not owned and should not appear in code or email templates

Messaging intent:
- agency is the first HyperSagi Labs product
- positioning should support both dynamic and regulated industries
- avoid overstating compliance and proof until artifacts are real

---

## 15) Notes

- This document reflects the current working state as of 2026-03-03.
- If you re-enable content, keep claims evidence-backed to avoid credibility risk.
- If/when `hypersagi.com` becomes a separate product site again, revisit canonical and OG base URLs intentionally.
