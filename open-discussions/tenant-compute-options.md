# Tenant compute: Options and tensions

> **Status:** Supporting reference — supports the [technical playbook](../guides/technical-playbook.md), section 3.1
> **Date:** March 2026

---

## The question

Section 3.1 of the Playbook lists standardised Kubernetes (k8s) compute within the NHS Azure tenant as a platform-provided capability, and §3.2 mandates that all services run within that tenant. This document explores whether that is the right position, or whether there is a coherent case for tenant teams running their own compute — for backend services, for micro-frontend bundle hosting, or for both.

This is not a resolved question. The current Playbook defaults to centralised NHS Azure compute because that is the simplest model from a security and governance standpoint. But "simplest" is not always "best", and the tension between platform control and genuine tenant autonomy is real.

---

## Three compute models

### Model A — All compute within NHS Azure (current Playbook position)

All tenant backend services and micro-frontend hosting run on NHS App-provisioned k8s clusters within the NHS Azure tenant. The NHS App team provides, configures, and maintains the compute infrastructure.

**What this gives you:**
- Single security perimeter — IG, InfoSec, and pen testing scope is clear and consistent across all tenants
- Centralised observability — all services emit logs and metrics into shared Splunk without cross-boundary forwarding
- NHS App team absorbs infrastructure toil — tenant teams focus on their service, not their infrastructure
- BFF pattern is straightforward — the BFF and the services behind it are all within the same network boundary

**What this costs you:**
- Genuine delivery independence is limited — a tenant's infrastructure capacity, scaling, and configuration depends on the NHS App team
- NHS App team carries infrastructure risk on behalf of all tenants — a poorly-behaved tenant service can affect shared cluster resources
- Onboarding cannot start until the NHS App team provisions capacity
- Tenant teams with existing, mature infrastructure capability are constrained to a model they may not need

---

### Model B — Tenant-owned compute, all within NHS Azure

Tenant teams provision and operate their own k8s clusters (or other compute) within the NHS Azure tenant. The NHS App team provides guardrails — approved Azure subscription configuration, network policies, and tagging standards — but does not operate the compute itself.

**What this gives you:**
- True infrastructure ownership — the team that wrote the service also owns what it runs on
- Independent scaling — a tenant can scale their infrastructure without a dependency on the NHS App team
- Cleaner cost attribution — Azure spend is directly attributable to the tenant's subscription from day one
- Consistent with "you own what you ship, permanently" (§1.2) — ownership now extends to the runtime

**What this costs you:**
- Tenant teams need infrastructure capability — not all tenant teams will have Azure expertise or a platform engineer
- Observability requires explicit configuration — tenant compute must be configured to forward logs and metrics to shared Splunk; if that pipeline breaks, incident investigation is impaired
- NHS App team needs a governance model for approving tenant Azure configurations — this is not currently defined
- Security posture is now distributed — the NHS App team's ability to assert a known-good security baseline across all tenant infrastructure is weaker

---

### Model C — Tenant-owned compute, outside NHS Azure

Tenant teams run backend services (and potentially micro-frontend hosting) on their own existing infrastructure, outside the NHS Azure tenant entirely. The BFF remains inside NHS Azure and acts as the boundary between the NHS App and the tenant's external services.

**What this gives you:**
- Maximum tenant autonomy — teams with mature, approved infrastructure can use it
- Some tenant teams may already have IG-cleared environments that are better suited to their backend data
- No dependency on NHS Azure capacity for tenant services

**What this costs you:**
- Every data flow from NHS App to tenant backend now crosses an infrastructure boundary that requires IG review — what NHS patient data is transmitted, to what environment, under what processing agreements
- The BFF now makes outbound calls to external services — network security, latency, and reliability are harder to control and audit
- Observability is split — NHS App front-end behaviour is in NHS Azure Splunk; tenant backend behaviour is in whatever the tenant operates. Cross-boundary incident investigation becomes significantly harder
- Platform pen testing does not cover tenant infrastructure — tenants must run independent pen tests
- Content Security Policy implications for micro-frontend hosting — if tenant JS bundles are served from non-NHS origins, the shell's CSP must whitelist those origins, creating audit surface and supply-chain risk
- NHS England contractual and data governance obligations may constrain this regardless of preference

---

## The micro-frontend hosting case

Micro-frontend bundles (once the Playbook's target web architecture is reached — see [Micro-frontends: Direction of Travel](micro-frontends.md)) are static JS assets, not data-processing services. They do not handle patient data directly. The IG argument for keeping them within NHS Azure is weaker than for backend services.

The most natural compute for static JS bundles is a CDN, not k8s. This points toward either:
- An NHS Azure-hosted CDN (Azure CDN / Front Door) provisioned by the NHS App team — consistent with Model A, but more appropriate than k8s for this use case
- A tenant-operated CDN — consistent with Model B or C, but introduces the CSP and supply-chain considerations noted above

This is a sufficiently distinct question from backend compute that it may warrant a separate decision.

---

## Key dependencies of the compute decision

The choice of compute model has direct knock-on effects on the following:

| Area | Impact |
|------|--------|
| **IG and data governance** | Cross-boundary data flows require per-flow IG review under Model C; Model B within NHS Azure is lower friction |
| **BFF architecture** | Model C changes the BFF's threat model — outbound calls to external services are fundamentally different from internal NHS Azure service-to-service calls |
| **Observability** | Models B and C require explicit log/metric forwarding into shared Splunk; a broken forwarding pipeline creates an observability gap during incidents |
| **Platform pen testing** | Models B and C partially or fully remove tenant infrastructure from platform pen testing scope |
| **Tenant team capability requirements** | Models B and C require infrastructure and platform engineering skills that not all tenant teams will have |
| **Cost attribution** | Model B makes cost attribution natural and clean; Model A requires internal chargeback or allocation |
| **Onboarding speed** | Model A requires NHS App team provisioning; Models B and C allow tenants to start sooner if they have their own infrastructure ready |
| **CSP and supply-chain risk** | Model C micro-frontend hosting from external origins widens the shell's CSP and introduces a supply-chain vector |

---

## Open question

This decision needs to be made deliberately before the first external tenant cohort onboards. See Appendix A, item A10 in the Playbook for the formal open question.
