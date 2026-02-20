# Frontend Messaging Recommendations

**Date:** 2026-02-20
**Based on:** Competitor analysis (Agent Integrator) vs current HSL website audit
**Constraint:** No invented numbers — only reshaping existing copy and framing.

---

## Executive Assessment

The current site is well-built technically and has strong structural bones (industry verticals, compliance positioning, services with pricing). But the **messaging is generic and passive**. It reads like a template — competent but forgettable. Compared to the competitor's punchy, outcome-driven copy ("Every day you wait, you get more replaceable"), HSL's site lacks:

1. **Stakes** — there's no cost-of-inaction framing anywhere
2. **Specificity** — headlines like "Operate like a large enterprise" and "Solutions that scale" could belong to any agency
3. **Credibility signals** — fake leadership bios, placeholder phone numbers, and empty case study/blog pages actively damage trust
4. **Clear buyer path** — "Book Discovery Call" is generic; the visitor doesn't know what happens next

The competitor has gaps we can exploit (shallow proof, founder-dependent, no compliance story), but we need sharper copy to do it.

---

## Homepage (`+page.svelte`)

### H1. Replace the headline

**Current:** "Operate like a large enterprise"
**Problem:** Aspirational but vague. "Large enterprise" isn't what SMEs aspire to — they want results without the bloat.

**Recommended:** "Your operations, automated. Your team, multiplied."
**Why:** Outcome-specific, creates a mental picture, implies scale without bureaucracy.

### H2. Replace the badge

**Current:** "v0 for Enterprise"
**Problem:** "v0" means nothing to a business owner. It's developer jargon.

**Recommended:** "AI Agents for SMEs & Mid-Market"
**Why:** Immediately qualifies the ICP. Tells the visitor "this is for me."

### H3. Rewrite the subhead

**Current:** "With AI agents, rapid MVPs, and distribution that earns. Remote-first roboagency delivering compliance-by-design solutions."
**Problem:** Feature list, not a value proposition. "Roboagency" needs explanation.

**Recommended:** "We build AI agents that handle your repetitive work, ship MVPs in weeks, and keep you compliant from day one. No bloated teams. No six-month timelines."
**Why:** Addresses pain (repetitive work, slow timelines, compliance risk), states delivery model, and differentiates on speed.

### H4. Sharpen the CTAs

**Current:** "Book Discovery Call" / "Explore Solutions"
**Problem:** Generic. The visitor doesn't know what they're committing to.

**Recommended:**
- Primary: "Get a Free Operations Audit" → `/contact`
- Secondary: "See How It Works" → `/services`

**Why:** The primary CTA offers something concrete and low-commitment. "Free audit" is the entry point that the competitor uses effectively with their "Fix My Operations" framing.

### H5. Rewrite the proof bar

**Current:** "Remote-first" / "Compliance by Design" / "Enterprise-grade Security"
**Problem:** These are features, not proof. They don't differentiate.

**Recommended:** "GDPR compliant by design" / "2–3 week deployment" / "Human-in-the-loop safety" / "From £5k pilot"
**Why:** More specific, includes a timeline and price anchor, and the compliance angle is a genuine differentiator the competitor lacks.

### H6. Rewrite the solutions section header

**Current:** "Solutions that scale" / "From AI agents to quantum readiness, we cover the full spectrum."
**Problem:** Generic. "Full spectrum" is a red flag for early-stage agencies — it signals lack of focus.

**Recommended:** "What we build" / "Focused solutions that ship in weeks, not months."
**Why:** Direct, sets an expectation, and the speed framing counters the "big agency = slow" objection.

### H7. Remove or deprioritize Quantum & Frontier R&D

**Problem:** This service isn't credible for a lean agency. It dilutes the core message and makes the offering feel unfocused — exactly the over-extension weakness identified in the competitor.

**Recommended:** Remove from homepage. Keep on services page as a "Coming soon" or "Research partnerships" footer item if desired.

### H8. Sharpen the industry cards

**Current pain/outcome framing is good** but the labels "Pain:" and "Outcome:" are clinical.

**Recommended:** Replace "Pain:" → "The problem" and "Outcome:" → "What we deliver"
**Why:** Slightly warmer, more conversational, clearer.

### H9. Add context to metrics

**Current:** Numbers float without narrative. "-40% Operations Cost" / "Legal Firm" — no link, no story.

**Recommended:** Add a one-line qualifier beneath each metric card, e.g. "Legal firm automated intake triage, cutting ops costs by 40% in 8 weeks." If you can't link to a case study yet, at least give the number a sentence of context.

### H10. Rewrite the compliance section

**Current header:** "Compliance & safety built in"
**Problem:** Fine but passive.

**Recommended header:** "Compliance isn't optional. We build it in from day one."
**Why:** Takes a stance. Makes it feel like a principle, not a feature checkbox.

### H11. Remove the ROI Calculator link

**Current:** Links to `/roi-calculator` which doesn't exist.
**Problem:** Broken links destroy credibility.

**Recommended:** Remove until the calculator is built. Replace with a CTA to "Get a Free Operations Audit" or a simple "Talk to us about your ROI" link to `/contact`.

### H12. Rewrite the bottom CTA section

**Current:** "Ready to transform your operations?" / "Start with an agent pilot or book a workshop to upskill your team."
**Problem:** Passive, no urgency.

**Recommended:** "Every week of manual work is money you're leaving on the table." / "Start with a pilot — see results in 2–3 weeks, not months."
**CTA buttons:** "Start an Agent Pilot" / "Book a Workshop"

---

## Services Page (`services/+page.svelte`)

### S1. Rewrite the hero

**Current:** "Services that scale with you" / "From AI agents to quantum readiness, we deliver enterprise-grade solutions with startup speed."

**Recommended:** "What we do" / "Focused AI solutions. Shipped fast. Built to last."

### S2. Replace the benefits bar labels

**Current:** "Rapid Delivery" / "Remote-first" / "Cost Effective" / "Results Focused"
**Recommended:** "Ship in weeks" / "Global team, your timezone" / "From £5k pilot" / "Outcomes, not hours"

### S3. Remove or demote Quantum & Content Studio

These dilute the core offer. Keep AI Agents, MVP Sprints, Advisory, and Compute Ops as the four pillars. Content Studio can become a sub-offering under Advisory. Quantum goes to a future/research section.

---

## About Page (`about/+page.svelte`)

### A1. Remove fake leadership team — CRITICAL

**Current:** Dr. Sarah Chen (MIT PhD), Marcus Rodriguez (two exits), Aisha Patel (100+ projects).
**Problem:** These are clearly placeholder bios. If any visitor Googles these names and finds nothing, trust is destroyed instantly. This is the single highest-risk element on the entire site.

**Recommended:** Replace with a founder-focused section (your real bio) or remove the leadership section entirely and replace with "Our Approach" — a process/methodology section that builds trust without requiring named individuals.

### A2. Fix the stats

**Current:** "7+ Years Experience" / "100% Remote-First" / "48hrs Response Time" / "24/7 Global Coverage"
**Problem:** "7+ years experience" of what? The company or the founder? If the company is new, this is misleading.

**Recommended:** Only keep stats you can genuinely back. If the founder has 7+ years, say "7+ years in AI & automation" with founder context. "48hrs Response Time" is fine. "100% Remote-First" and "24/7 Global Coverage" are fine.

### A3. Rewrite the mission statement

**Current:** "To democratize enterprise-grade AI capabilities, making them accessible to organizations of all sizes through remote-first delivery and compliance-by-design principles."

**Recommended:** "We exist to give SMEs the AI capabilities that only large enterprises could afford — without the cost, complexity, or compliance risk."
**Why:** Same idea, half the words, and names the tension (SMEs vs enterprises, cost vs capability).

### A4. Rewrite the hero

**Current:** "Building the future of work"
**Problem:** Cliché. Every AI company says this.

**Recommended:** "The agency that builds your AI workforce"
**Why:** Specific, ties to the "agents as team members" mental model.

---

## Contact Page (`contact/+page.svelte`)

### C1. Remove fake phone number

**Current:** "+1 (555) 123-4567"
**Problem:** Obviously fake. Damages trust.

**Recommended:** Remove phone entirely if you don't have one. Keep email and "Remote-first (Global)" location. Add a "Typical response time: under 24 hours" note instead.

### C2. Fix email address

**Current:** "hello@hypersagilabs.com"
**Problem:** Domain doesn't match `hypersagi.com`.

**Recommended:** Use `hello@hypersagi.com` or `contact@hypersagi.com` — must match the actual domain.

---

## Industries Page (`industries/+page.svelte`)

### I1. Largely good as-is

The pain/outcome framing and per-industry stats are effective. The main improvement is contextualizing the stats (see H9 above) and potentially adding a "Talk to a specialist" CTA per industry card rather than just "Learn more."

---

## Global / Layout (`+layout.svelte`)

### L1. Rewrite footer tagline

**Current:** "Remote-first roboagency delivering AI agents, MVPs, and enterprise solutions."

**Recommended:** "AI agents and MVPs for SMEs. Compliance built in."
**Why:** Shorter, names the ICP, leads with the differentiator.

### L2. Consider renaming nav CTA

**Current:** "Book Discovery Call"
**Recommended:** "Get a Free Audit" — lower friction, more concrete.

---

## Priority Order for Implementation

| # | Change | Impact | Effort |
|---|--------|--------|--------|
| 1 | **A1** Remove fake leadership team | Critical (trust) | Low |
| 2 | **C1** Remove fake phone number | Critical (trust) | Low |
| 3 | **C2** Fix email domain | Critical (trust) | Low |
| 4 | **H11** Remove broken ROI calculator link | High (credibility) | Low |
| 5 | **H1–H4** Rewrite homepage hero (headline, badge, subhead, CTAs) | High (conversion) | Medium |
| 6 | **H7** Remove Quantum from homepage | High (focus) | Low |
| 7 | **H12** Rewrite bottom CTA with stakes | Medium (conversion) | Low |
| 8 | **A3–A4** Rewrite about hero + mission | Medium | Low |
| 9 | **S1–S3** Sharpen services page | Medium | Medium |
| 10 | **H5–H6, H8–H10** Remaining homepage refinements | Medium | Medium |
| 11 | **L1–L2** Layout/nav refinements | Low | Low |
