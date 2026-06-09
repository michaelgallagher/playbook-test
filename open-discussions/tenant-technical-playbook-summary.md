# Technical playbook — Executive summary

> **Version:** 0.1 | **Date:** March 2026 | **Status:** DRAFT — Internal review only

---

## The problem this solves

The NHS App has 13 million monthly active users and a single engineering team that cannot scale to meet demand from across NHS England. Teams with funded services want to contribute features through the App. Today there is no safe, structured way for them to do that. The Playbook defines the third path: not a free-for-all, not a bottleneck — a governed platform model.

## The model in one sentence

NHS App is the **Landlord**. NHSE service teams are **Tenants**. The Landlord owns the building; Tenants own their shops. The Landlord sets building regulations; Tenants must follow them. A team that ships a feature accepts permanent ownership of it — there is no handing it back.

---

## What the Playbook requires

**From tenant teams:**

- Own your feature for its full lifecycle. If funding ends, ownership transfers or the feature is retired — it does not default to the NHS App team.
- Deploy behind a feature flag from day one. Activation (making it visible to users) is a separate, gated act from deployment.
- Meet the Feature Activation Criteria (§1.5) before any flag flip: all tests passing, accessibility audited, IG/InfoSec signed off, clinical assurance where applicable, runbook published, monitoring live, kill-switch tested.
- Maintain automated test coverage to platform-defined thresholds — branch coverage and mutation scores are hard pipeline gates, not targets.
- Ship to all three channels (Web, iOS, Android). Single-channel piloting requires an explicit time-bound exception.
- Be the first responder to your own incidents. The NHS App team coordinates; you diagnose and fix.

**From the NHS App team:**

- Classify all platform changes as breaking, non-breaking, or deprecating before they ship.
- Give advance notice of deprecations with a migration guide and a firm end-of-life date.
- Own the kill switch — the right to deactivate any feature on any channel at any time, without prior notice.
- Maintain the shared platform: design system, CI/CD templates, feature flag infrastructure, observability stack, BFF architecture, staging environments.

---

## The Platform Subsidy

In exchange for compliance with the Playbook, tenant teams get: Kubernetes compute (NHS Azure), standardised CI/CD, Splunk observability, Databricks + Adobe Analytics, centralised feature flagging with kill-switch, the NHS App Design System (web and native), platform-level pen testing, and a documented onboarding path. Building this independently would cost a individual team months.

---

## Onboarding: Phase 0

Before the first external tenant cohort, the NHS App team will run two streams in parallel: an internal dogfooding team (acting as tenants themselves) and a single external pilot team (a close collaborator with a real feature to deliver). Neither stream alone is sufficient. The first broader cohort does not open until both are complete and the Playbook has been updated based on what they found.

---

## Deep dives

The following supporting documents expand on areas where the Playbook is deliberately brief:

| Document | Covers |
|----------|--------|
| [Feature Lifecycle Tracking](feature-lifecycle-tracking.md) | Software catalog + scorecard model for automated portfolio audit; IDP tooling options (Backstage, Cortex, OpsLevel, Port); SBOM and Dependency-Track integration |
| [Test Coverage Quality](../guides/technical/test-coverage-quality.md) | What makes a test suite trustworthy; branch coverage thresholds; mutation testing (Stryker); contract testing (Pact); platform minimum thresholds table |
| [Micro-frontends: Direction of Travel](micro-frontends.md) | Why micro-frontends; Webpack Module Federation approach; four-phase migration from current Vue monolith; what this means for early-onboarding tenants |
| [Tenant Compute: Options and Tensions](tenant-compute-options.md) | Three compute models with honest pros/cons; IG, BFF, and observability knock-on impacts; micro-frontend bundle hosting as a separate sub-question |
| [Continuous Deployment: Practices and Coping Strategies](../guides/technical/continuous-deployment-practices.md) | Staging floor vs. production CD; feature flags as safety valve; short-lived branches; release cadences; branch-by-abstraction; progressive delivery |

---

## Open questions requiring decisions before external onboarding

| # | Question | Urgency |
|---|----------|---------|
| A1 | Formal notice period and P&P authority for feature retirement | Before first external tenant |
| A6 | BFF ownership: one shared NHS App BFF vs. tenant-owned BFF services | Before first backend integration |
| A10 | Tenant compute model: NHS App-provisioned vs. tenant-owned (within or outside NHS Azure) | Blocks cost model and IG sign-off |
| A11 | Identity propagation through the BFF: JWT forwarding, token exchange, or signed internal assertion | Before first backend integration |
| A12 | Feature flagging migration: from bespoke SJRs to OpenFeature standard | Blocks self-service capability |

Full open questions list at Appendix A of the Playbook.
