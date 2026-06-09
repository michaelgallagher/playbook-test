# Technical Playbook

> **Version:** 0.1 — Draft for review
> **Date:** March 2026
> **Author:** NHS App Engineering, NHS England
> **Status:** DRAFT — Internal review only. Not approved for external distribution.

---

## The Epilogue Criteria

Before anything else, the discussions that shaped this Playbook produced four litmus tests. Apply them whenever a rule in this document feels wrong, incomplete, or outdated. If a rule fails one of these tests, raise it — the Playbook is a living document.

| # | Test | The question to ask |
|---|------|---------------------|
| 1 | **Reversibility** | Is this decision reversible? If not, are we comfortable making it now? |
| 2 | **Cognitive load** | Does this rule make a tenant team's job easier or harder? |
| 3 | **User experience** | Does this protect or endanger the experience of 13 million monthly active users? |
| 4 | **Automation** | Can this be automated? If not, why not — and when might it be? |

These are referred to throughout as the **Epilogue criteria**.

---

## Foreword

The NHS App is used by over 13 million people every month. It is one of the most critical digital touchpoints in the NHS. Every change to it carries real clinical and reputational risk.

For most of its life, the NHS App has been built and maintained by a single, tightly-controlled engineering team. That model has served the service well: it produced a consistent, safe, and reliable product.

But two forces are now pulling against each other:

1. **The NHS App team has a capacity ceiling.** A single team responsible for every change to the web application, the APIs, and the iOS and Android native applications cannot scale indefinitely without sacrificing the quality and reliability that 13 million people depend on.
2. **Teams across NHS England have services they want to deliver through the NHS App.** These teams have the domain expertise, the funding, and the motivation to build features for their users — but today there is no safe, structured way for them to contribute to a shared codebase and a shared production environment.

The question this Playbook answers is: **how do we allow those teams to safely deliver features within a shared ecosystem?**

The answer is not "let everyone commit to the same repositories and hope for the best." Nor is it "funnel every change through the NHS App team and accept the bottleneck." It is a third path: a defined platform model with clear ownership, automated guardrails, and a social contract that protects the 13 million people who use this service while enabling the teams who want to contribute to it.

### The Landlord-Tenant model

We use the **Landlord-Tenant** metaphor to reason about the new model. It is a starting point for clear thinking, not a mandate cast in stone.

- **The Landlord (NHS App team)** owns the common areas: navigation, authentication, the application shell, the design system, the plumbing, and the infrastructure that keeps the building standing. The Landlord is responsible for the safety, integrity, and quality of the overall service.
- **The Tenants (NHSE service teams)** own their shops inside the building. They design their own space, serve their own customers, and are responsible for maintaining what they build.

The Landlord does not build the shops for tenants. The Landlord defines the building regulations that all shops must comply with, provides the shared utilities, and enforces standards that protect every customer who walks through the door.

This evolution means the NHS App team shifts from **feature factory to enabling platform**. That shift is not primarily a structural decision — it is an engineering and cultural one. This Playbook is the bridge.

### How to read this document

This document is written for two audiences simultaneously:

- **Tenant engineering teams** — teams across NHS England who want to deliver functionality through the NHS App.
- **NHS App leadership and engineers** — the Landlord, who owns and maintains the Playbook.

Rules are stated directly and prescriptively. Where a rule might feel arbitrary or restrictive, a **Why this matters** explanation follows. Understanding the "why" is as important as following the rule — because context enables good judgement in the situations this Playbook does not yet cover.

The NHS App spans three delivery channels — **Web** (the NHS App web application), **iOS**, and **Android**. Where rules differ by channel, they are called out explicitly under a channel heading. A rule that appears without a channel qualifier applies to all three.

---

## Part 1 — The Social Contract

The Social Contract defines who owns what, who is accountable for what, and what happens when things go wrong or when project funding runs out.

### 1.1 Ownership delineation

The following table defines the split between Landlord and Tenant responsibilities. Where this table is silent, the Landlord's position is that the Landlord owns it until explicitly agreed otherwise.

| Domain | Landlord (NHS App team) | Tenant (service team) |
|--------|------------------------|-----------------------|
| Navigation, authentication, shell, design system | ✅ Owns | Must not modify |
| Shared infrastructure and pipelines | ✅ Owns | Uses; does not configure independently |
| Feature flagging infrastructure | ✅ Owns | Controls their own flags via agreed interfaces |
| Observability and analytics platforms | ✅ Owns | Instruments their code against agreed standards |
| Clinical assurance, IG (data protection), and InfoSec coordination | ✅ Owns process | Provides evidence, attends, and signs off |
| Tenant feature code | Provides guardrails | ✅ Owns |
| Tenant backend services | Provides platform | ✅ Owns |
| Tenant incident resolution | Coordinates | ✅ First responder |
| Tenant test coverage | Sets minimum bar | ✅ Owns |
| Tenant runbooks and knowledge base entries | Gate at feature activation | ✅ Produces |

**Why this matters:** Unclear ownership is how features get abandoned, incidents escalate past their owner, and costs accumulate without accountability. This table is the starting point for any disagreement about who should act.

### 1.2 You own what you ship. Permanently.

A team that ships a feature to the NHS App accepts **ongoing ownership** of that feature for its entire lifecycle. There is no mechanism to hand a feature back to the NHS App team once it is live.

This is a non-negotiable condition of entry to the platform. It applies even if:

- The original team is reorganised or disbanded
- The project funding cycle ends
- The feature sees low or declining usage

**Why this matters:** The NHS App cannot become a dumping ground for unmaintained services. Authoritative ownership of every technical artefact — from code to runbooks to data stores — must trace back to an identifiable team at all times. The NHS App team is not resourced to absorb orphaned features, and doing so would degrade the service for everyone.

### 1.3 Incident ownership

When a tenant feature causes or contributes to an incident:

1. **The tenant owns the incident.** They are the first responder, they diagnose the fault, and they fix it.
2. **The NHS App team owns coordination.** The NHS App team manages communications to users, coordinates across teams if the incident has cross-cutting impact, and controls the communications channels (`#alerts-high`, `#highsev-incident-updates`, in-app messaging via N&M).
3. **The NHS App team owns the kill switch.** The NHS App team can deactivate any tenant feature at any time, without prior agreement from the tenant team, if it is causing harm to the overall service or to patients.

The NHS App team is not responsible for diagnosing or fixing tenant features. If a tenant feature is causing a P1 incident and the tenant team is unresponsive, the NHS App team will deactivate the feature. The tenant will fix the underlying fault before reactivation is considered.

**Why this matters:** The current model of 24/7 on-call coverage embedded in every squad is not scalable. Centralised coordination with distributed ownership is. But that only works if lines of responsibility are unambiguous before an incident — not during one.

### 1.4 The feature lifecycle and the "delete button"

Every feature has a lifecycle: **Proposed → In development → In review → Active → Maintained → Retired**.

Teams are expected to actively manage their features through this lifecycle. The NHS App team will, on a regular cadence, audit active features and flag those that appear unmaintained — indicated by: no recent deployments, open security advisories without a remediation plan, unresolved accessibility issues, or no active owning team identifiable.

Flagged features follow this escalation process:

1. **Notice:** a notice period is issued to the owning team (duration TBD — see Appendix A, item A1).
2. **Escalation:** if there is no substantive response, the matter is escalated to portfolio level (P&P) for a business decision on whether ownership can be transferred or the feature must be retired.
3. **Retirement:** the feature is disabled via feature flag and, after a further stated notice period, removed from the codebase.

Retirement is not a failure. It is responsible portfolio management. Teams should plan for it from day one.

**Why this matters:** Every NHS England team has inherited dead systems that persist indefinitely because no clear process exists to remove them. We are not repeating that pattern in the NHS App.

**How we track this at scale:** At the scale this platform is targeting, a manual audit cadence is not sufficient. See [Feature Lifecycle Tracking](../open-discussions/feature-lifecycle-tracking.md) for the software catalog and scorecard model that automates this process — including tooling options and how each signal maps to a machine-readable rule.

### 1.5 Feature Activation Criteria

This checklist operates at the **feature** level, not the story or PR level. Individual stories and pull requests are gated by the automated pipeline (see Part 2), which enforces the conventional story-level checks: tests pass, linting clean, peer review approved (i.e. definition of done). Those checks happen continuously throughout development and are not listed here.

This checklist is the gate before the feature flag is flipped and the feature becomes visible to users. A feature may be fully deployed to production — across many PRs and many sprints — while this checklist remains incomplete. That is expected and correct. Deployment and activation are separate acts (see §5.1).

A feature is ready to activate when **all** of the following are true:

- [ ] All automated tests pass at the required thresholds (see Part 2)
- [ ] Accessibility audit passed (WCAG 2.2 AA minimum)
- [ ] Information Governance (IG) sign-off obtained
- [ ] Information Security (InfoSec) sign-off obtained
- [ ] Clinical assurance obtained (where the feature touches clinical workflows)
- [ ] Runbook written and published to the shared knowledge base
- [ ] Monitoring and alerting configured against the agreed SLOs (see Part 3)
- [ ] Feature flag registered and kill-switch tested end-to-end
- [ ] On-call escalation path documented and registered with the NHS App team
- [ ] SBOM entry created and registered

**Why this matters:** A feature without a runbook costs the NHS App team incident response time diagnosing a system their engineers did not build. A feature without monitoring is invisible until it breaks publicly. This checklist is the service wrapper — it is what separates a safely activated feature from a future liability.

### 1.6 Engineering Redlines

The NHS England Engineering Redlines (available at https://nhs.sharepoint.com/sites/X26_EngineeringCOE/SitePages/Red-lines.aspx) are hard, non-negotiable rules. Every rule in this Playbook sits above that baseline. Where this Playbook is silent, the Redlines apply. Where a Redline appears to conflict with your use case, you must request a formal exemption — there are no informal workarounds.

---

## Part 2 — Technical Guardrails

The goal of the technical guardrails is to allow tenant teams to ship changes to 13 million monthly active users **without requiring human review from the NHS App team for every change**. That is only possible if tenant code is trustworthy by default — and trustworthiness is built through automation, not through review gates.

### 2.1 What makes a change "safe"

A change is safe to ship without NHS App team intervention when all of the following are true:

1. All automated test suites pass: unit, integration, contract, accessibility, and performance
2. No new security advisories are introduced by the change (automated dependency scanning)
3. No new accessibility violations are introduced
4. The risk register for the feature has been updated and all controls have associated passing automated tests
5. Code standards enforcement passes (automated — not a manual review step)
6. The change has been peer-reviewed and approved by the tenant team's own engineers

**Why this matters:** Human review by the NHS App team at this scale is a bottleneck that will break the model. The answer to "how do we know this is safe?" must be automation, not gatekeeping. This does not mean quality is lower — it means quality is enforced differently, and earlier.

**What makes a test suite trustworthy:** "All tests pass" is a necessary condition, not a sufficient one. A suite with no meaningful assertions will pass every time. The platform enforces branch coverage thresholds, mutation scores, and contract tests for integration boundaries as hard pipeline gates. See [Test Coverage Quality](technical/test-coverage-quality.md) for the full model — including tooling, minimum thresholds, and ownership.

### 2.2 Shift left

Problems caught at design time cost one conversation. Problems caught at PR review cost a day. Problems caught in production cost an incident, a potential clinical risk, and possible patient harm.

**The expectation is that design and technical alignment discussions happen before a line of code is written, not at PR review.**

The NHS App team provides design review checkpoints as part of onboarding (see Part 6). Use them. A PR that introduces a fundamental design disagreement will be rejected at review. Raising that same disagreement at a design checkpoint will resolve it in hours.

The required shift-left practices are:

- Architecture Decision Records (ADRs) for any non-trivial technical decision, approved before implementation begins
- Design system compliance review before visual implementation starts
- IG and InfoSec engagement at requirements stage, not at sign-off stage
- Clinical assurance engagement at design stage for any feature touching clinical workflows

### 2.3 Risk management process

For every feature, the tenant team must maintain a living risk register that forms a traceable chain:

**Requirements → Identified risks → Controls linked to each risk → Automated tests that verify each control → Reporting on gaps**

This is not a document exercise. It is a traceable, automated chain. The quality gates in the CI/CD pipeline must be traceable back to controls in the risk register. A gap in automated test coverage for a control is a known, unmitigated risk being shipped to 13 million people. Report it; do not obscure it.

**Why this matters:** Risk management without traceability to automation is theatre. The Playbook requires that compliance is continuously verifiable, not periodically asserted.

### 2.4 Code and quality standards

All tenant code must comply with the [NHS Digital Software Engineering Quality Framework](https://github.com/NHSDigital/software-engineering-quality-framework). Compliance is enforced by automated tooling in the CI/CD pipeline. Human review does not substitute for a failing automated check.

In addition:

- Code standards are enforced via automated linting in the pipeline — a pipeline that does not pass cannot be merged
- `CODEOWNERS` files must be populated and kept current at all times; stale `CODEOWNERS` entries are a pipeline failure
- Pull requests must be small and focused — a PR that changes more than one logical concern will be rejected
- PRs must not remain open and unactioned for more than 7 working days

If your team has a review bottleneck, the answer is smaller PRs merged more frequently — not larger PRs reviewed less often.

### 2.5 Web channel — technical specifics

> **Note:** The rules below describe the **target architecture** for web channel tenant delivery. The NHS App web application is currently a monolithic Vue application, not a micro-frontend architecture. The migration from monolith to micro-frontends is in progress. Tenant teams onboarding before that migration is complete will follow an interim delivery path agreed during initial engagement. See [Micro-frontends: Direction of Travel](../open-discussions/micro-frontends.md) for a full explanation of what micro-frontends are, why they are the right architecture for this model, the migration plan, and what to expect during the transition.

The target architecture is a Vue-based micro-frontend architecture. Tenant teams will deliver web features as micro-frontends.

- Tenant source code lives in the tenant team's own repositories
- Micro-frontend code is built and published as a versioned artefact consumed by the NHS App shell
- The NHS App shell handles routing, authentication, and navigation — tenants must not modify these
- Tenants must use NHS App Design System components; custom visual components require a design review and approval before implementation begins
- Tenants must track **`-latest`** on all curated internal dependencies (design system, shared utilities). Pinning to a fixed version is not permitted without a time-bound exception granted by the NHS App team
- Tenants must deploy to the shared environment on every commit to their main branch — "we'll deploy when we're ready" is not an acceptable practice

**Why this matters:** The web application must present a coherent, consistent user experience regardless of which team contributed each part. A tenant pinning to an old design system version and deferring updates is user experience debt that eventually costs the whole platform — including tenants other than the one deferring.

### 2.6 Native channel — iOS and Android technical specifics

The native applications have fundamentally different delivery mechanics from the web. They must be treated as a separate channel with separate constraints.

- **Deployment to production is release.** Once a build passes App Store review and is published, it is live for users who update. There is no post-deployment toggle in the way the web channel has. Plan accordingly.
- **Continuous Deployment to staging is mandatory.** Every change must flow automatically to the staging environment without manual intervention. If a change cannot complete the automated pipeline to staging, it is not ready to ship.
- **Multiple concurrent production versions are an operational reality.** Not every user updates immediately. Features must be designed to work correctly across the entire supported app version range. Behaviour that is correct in the current version but broken for users on an older supported version is a defect.
- **Combinatorial feature flag testing is required.** Native features interact with feature flags at runtime. All tested on/off combinations must be recorded with an audit trail before production release.
- **Kill-switches are the primary lever for native.** Because a build cannot be retracted once published, every native feature must have a kill-switch — a feature flag that disables the feature remotely without requiring a new app submission — tested in staging before the build is submitted for App Store review.

**Why this matters:** A defect in a web feature can be fixed in a deployment that takes minutes. A defect in a native build that has passed App Store review requires a new submission, a new review cycle, and a period during which the broken version is live with no ability to force an update. The higher bar for native automation and kill-switch readiness reflects that asymmetry directly.

---

## Part 3 — The Platform Subsidy

"Why would a team want to work with us rather than building their own channel?" The answer is the Platform Subsidy: the capabilities, infrastructure, and services that the NHS App provides to every tenant, which would cost a standalone team significant time and money to replicate.

### 3.1 What the platform provides

> **Open question:** The capabilities below reflect the current centralised compute model — all services run within the NHS Azure tenant on NHS App-provisioned infrastructure. There is a potential case for tenant teams owning their own compute, either within NHS Azure or on their own infrastructure, but that decision carries significant knock-on impacts for IG, observability, the BFF architecture, and security posture. This has not yet been resolved. See [Tenant Compute: Options and Tensions](../open-discussions/tenant-compute-options.md) for a full analysis, and Appendix A, item A10 for the open question.

| Capability | What you get |
|-----------|--------------|
| **Compute** | Standardised Kubernetes (k8s) clusters within the NHS Azure tenant |
| **CI/CD** | Standardised pipeline templates; build, test, and deploy to staging automated from day one |
| **Observability** | Splunk for log aggregation, search, and alerting; dashboards provisioned per feature area |
| **Analytics** | Databricks for data analysis; Adobe Analytics for user behaviour; broad access to population-level data, not just the tenant's own feature slice |
| **Feature flagging** | Centralised feature flag infrastructure with kill-switch capability, cohort-based rollout, and a full audit trail |
| **Design system** | NHS App Design System component library for web; native design system components for iOS and Android |
| **Client-side logging** | Standardised logging framework (web: Sentry-equivalent; native: equivalent mobile tooling) |
| **Security testing** | Platform-level penetration testing on a regular cadence; results shared across all tenants |
| **Staging environments** | Shared staging environment for integration testing before production release |
| **Onboarding support** | Documented onboarding path, reference implementations, cookbooks, and access to the NHS App engineering community |

### 3.2 What you do not get to bring yourself

The following are **not negotiable**. Tenants may not substitute their own alternatives without a formally approved, time-bound exemption.

| Category | Rule | Reason |
|----------|------|--------|
| **Analytics tooling** | You must use the platform analytics stack (Databricks, Adobe Analytics) | Adding new data processors requires IG/GDPR review and creates new InfoSec attack surface. The review burden consistently outweighs any benefit |
| **Compute** | All services must run within the NHS Azure tenant | Data sovereignty, InfoSec posture, and contractual obligations require this without exception |
| **Client-side logging** | You must use the platform logging framework | Fragmented logging makes incident investigation across tenant boundaries impossible at scale |
| **Observability** | You must use Splunk | Provides cross-tenant service health visibility for R&M; also consistent with the logging requirement above |

If you have a genuine exceptional need to deviate, raise it formally with the NHS App platform team. The answer will usually be no. If the answer is yes, it will be a documented, reviewed, time-bound exemption — not an ad-hoc workaround that later becomes architectural debt.

### 3.3 Architecture pattern: backend connectivity

Tenant backend services must communicate with NHS App front ends through an **Adapter/BFF (Backend for Frontend) pattern**. The BFF is a secure adapter layer that lives within the NHS Azure tenant and mediates all communication between the NHS App front end and tenant backend systems.

- The front end (web or native) never calls a tenant backend service directly
- The BFF validates the user's NHS Login session and enforces request-level authentication at the platform boundary
- The BFF aggregates data from one or more tenant backend services and shapes the response for the specific needs of the client (web or native) — reducing round trips and avoiding the over-fetching that comes from exposing raw backend APIs directly to the front end
- Tenant services live behind the BFF, within the NHS Azure tenant

**Identity propagation:** The BFF does not terminate identity — tenant backend services will typically need to know *who* is making a request in order to serve the correct data. The BFF is responsible for propagating the authenticated user's identity to downstream services in a verified, tamper-proof form. The mechanism for doing this (forwarding the original NHS Login JWT, exchanging it for a service-scoped token, or issuing a signed internal identity assertion) is not yet defined and is a significant design decision. See Appendix A, item A11.

**Why this matters:** Direct front-end to backend connections across tenant boundaries create unpredictable latency, brittle coupling, and an InfoSec exposure that is difficult to audit. The BFF pattern enforces a clear, auditable integration boundary with a single point of security control.

See Appendix A, item A6 for an open question about BFF ownership.

### 3.4 Performance SLOs

Every tenant feature is assigned a **performance budget** at onboarding. This covers:

- Page load / screen render time (P90 target)
- API response time (P90 and P99 targets)
- Error rate (maximum acceptable percentage)
- Availability (minimum percentage)

A tenant feature whose production metrics consistently breach its SLO will be flagged for investigation. If not remediated within the agreed timeline, the feature will be disabled.

**Why this matters:** A slow or unreliable tenant feature degrades the perceived quality of the entire NHS App for all users — not just the users of that specific feature. Your SLO is effectively a contract with all 13 million people who use this service, not just your own users.

---

## Part 4 — Communication and Discoverability

At scale, one of the largest ongoing engineering costs is not building new things — it is finding out what already exists and staying synchronised with how it changes. This section defines how the NHS App team communicates platform changes, and how tenant teams can discover what the platform already offers.

### 4.1 How we communicate platform changes

The NHS App team follows this model for all platform changes that affect tenant teams:

1. **Identification:** changes are classified as breaking, non-breaking, or deprecating at the point they are planned — not at the point they ship
2. **"Stone ripple" analysis:** for any change to a core component or API, the NHS App team performs an impact analysis to identify all affected tenants before the change ships
3. **Advance notice:** deprecations are communicated with a clearly stated end-of-life date and a migration guide, published before the change is made
4. **Supported version range:** at any point in time, the NHS App platform supports a defined version range. Tenants operating on versions outside that range are in a formally unsupported state and are accountable for the risks that creates
5. **Non-negotiable updates:** some changes — security patches, clinical safety updates, accessibility requirements — cannot be deferred. When a non-negotiable update is issued, tenants have a stated adoption window; that window will not be extended

**Why this matters:** "We didn't know about the change" is not an acceptable explanation for a tenant feature breaking in production. The NHS App team owns the communication; tenant teams own the adoption.

### 4.2 Versioning strategy

**APIs** use major-version-only versioning. The version is surfaced in the URL path (e.g. `/v1/`, `/v2/`) or as a request header. Only breaking changes produce a new major version — a breaking change is any change that could cause a correctly-built client to fail. Non-breaking additions (new fields, new optional parameters) do not require a version increment. When a new major version is introduced, the prior major version enters a deprecation period with a stated end-of-life date before support is withdrawn. Full SemVer (`MAJOR.MINOR.PATCH`) is not applied to APIs — minor and patch increments are internal implementation details invisible to consumers.

**Libraries and binaries** (the design system component library, shared web utilities, native SDK components) follow **Semantic Versioning (SemVer)**. All three version parts carry meaning for consumers: breaking changes increment the major version, backwards-compatible additions increment the minor version, and fixes increment the patch version. Breaking changes are preceded by a deprecation period before the prior major version is withdrawn.

**Native SDK components** follow SemVer with the additional constraint that breaking changes must account for App Store review cycle timelines when calculating the minimum deprecation window — a build submission is required before a tenant can adopt a breaking SDK change, and that submission takes time.

### 4.3 Discoverability: finding what already exists

Before building anything, tenant teams must check whether the capability already exists on the platform. Building something that already exists is wasted effort and creates fragmentation the platform team then has to manage.

| Resource | What it provides | Audience |
|----------|-----------------|----------|
| **Design system playground** | Interactive catalogue of all web and native UI components with usage examples | All tenant teams |
| **API discovery catalogue** | Searchable registry of all platform APIs, with versioning, ownership, and status | All tenant teams |
| **Reference implementations / cookbooks** | End-to-end examples of common integration patterns for web and native | All tenant teams |
| **Platform capability map** | Complete inventory of available platform capabilities and tooling | NHS App engineers and tenants |

These resources are owned and maintained by the NHS App team. If they are out of date or missing content that tenant teams need, that is a platform failure — raise it.

**Implementation note:** These four resources are most naturally delivered as features of an Internal Developer Portal (IDP) rather than as independently-built artefacts. The API discovery catalogue, ownership maps, and design system playground are first-class capabilities of platforms like Backstage — the same investment that automates feature lifecycle tracking also delivers discoverability. See [Feature Lifecycle Tracking](../open-discussions/feature-lifecycle-tracking.md) for the full tooling discussion.

### 4.4 Web channel — staying current

- **Every merge to main must automatically deploy to the shared staging environment.** This is the non-negotiable floor — main must always be in a known-deployed, tested state so that integration problems are discovered in hours, not weeks.
- **Staging is a production-like environment.** It must mirror production in configuration, infrastructure, and behaviour — it is not a permissive sandbox. In particular, **feature toggle configuration in staging must match production at all times**, except during the controlled window when a toggle is being tested before it is promoted. A staging environment that behaves differently from production does not give you the signal you need before a release.
- Automated promotion from staging to production is the long-term target. Teams not yet there must operate on a named, short release cadence. "We'll deploy when we're ready" is not a cadence.
- Feature flags are the primary mechanism that makes frequent deployment safe — code deployed behind a flag that defaults to off carries no user-visible risk. Teams must use them consistently from the start.
- Tenants must track **`-latest`** on all curated internal dependencies (see Part 2.5); pinning is not permitted
- When the NHS App team releases a new major version of the design system, tenants are assigned a migration window during which their code must be updated. This window will not be extended regardless of team capacity

**Why this matters:** A design system that has diverged across ten tenant implementations is not a design system — it is ten separate implementations that happen to start from the same point. Continuous tracking prevents incremental drift from becoming an unrecoverable gap.

**Continuous deployment in practice:** NHS England is not yet a continuous deployment organisation, and the journey there requires more than a pipeline rule. See [Continuous Deployment: Practices and Coping Strategies](technical/continuous-deployment-practices.md) for pragmatic approaches — including feature flags as a safety valve, short-lived branches, release cadences for teams not yet at full CD, and branch-by-abstraction for structural changes.

### 4.5 Native channel — staying current (iOS and Android)

The native channels have additional constraints that the web does not:

- Deprecation timelines **must account for App Store review cycles** (typically 24–72 hours for Apple; typically shorter for Google Play, but not guaranteed). These timelines are factored into the deprecation window the NHS App team sets — they are not grounds for extending that window.
- When a non-negotiable update is issued (e.g. a platform SDK security patch), tenants have a stated window to submit an updated build. Builds using a deprecated SDK version that miss that window may be blocked from the App Store by the platform team.
- Tenant teams are expected to maintain a practice of **continuous deployment to staging** for every commit. If absorbing a platform update requires an extraordinary effort, that is evidence that the practice of continuous deployment has lapsed.

### 4.6 Ownership maps

Every discrete section of the user interface — in both web and native channels — has a named owning team. Ownership is maintained through two complementary mechanisms that serve different purposes:

- **`CODEOWNERS`** (in each tenant repository) — maps file paths to teams for automatic PR review assignment. It answers: *"who reviews changes to this code?"*
- **`catalog-info.yaml`** (in each tenant repository, aggregated by the IDP) — declares the owning team, on-call contact, lifecycle state, and the UI routes the feature owns (via a `nhs-app/ui-routes` annotation). Aggregated across all tenant repos, this becomes the live, queryable ownership map for the entire platform. It answers: *"who do I contact when /prescriptions is broken in production?"*

The practical consequence is that the ownership map is not a document the NHS App team maintains centrally — it is a live view derived from metadata that tenant teams own in their own repositories. **A missing or stale `catalog-info.yaml` is a pipeline failure**, not a compliance reminder. There is no separate update SLA; ownership is updated as part of the PR that changes it.

See the [tooling section of Feature Lifecycle Tracking](../open-discussions/feature-lifecycle-tracking.md#tooling-options) for the IDP options that serve as the aggregation layer.

---

## Part 5 — Release Choreography

The NHS App serves a clinical population. The stakes of a bad release are not just user dissatisfaction — they include potential patient harm. Release choreography is how we ship confidently at scale while managing that risk explicitly.

### 5.1 Deployment is not release

**Deployment** is the act of placing code into an environment.
**Release** is the act of making that code visible and active for users.

These two operations must be **explicitly separated** in every tenant integration. Code can be deployed to production but remain hidden behind a feature flag. The decision to release — to flip the flag — is a separate act, governed separately, and requires its own set of gates to be satisfied.

This separation is critical in the context of clinical workflows, where clinical assurance sign-off must be completed before release, not before deployment.

**Why this matters:** The ability to deploy without releasing means that release decisions are calm, deliberate, and reversible — not rushed because code is in a half-deployed state that must either go forward or be rolled back under pressure. The separation of deployment from release is one of the highest-leverage practices available on this platform.

### 5.2 Authority to activate and deactivate

| Action | Authority |
|--------|-----------|
| Activate a tenant feature for rollout | The tenant team, after all release gates are passed |
| Activate a platform-wide change | The NHS App team |
| Deactivate a tenant feature | The tenant team **or** the NHS App team |
| Emergency deactivation of any feature on any channel | The NHS App team — always, without prior notice or consultation |

**The NHS App team always retains the unconditional right to deactivate any feature, on any channel, at any time.** This is not a statement of distrust toward tenant teams — it is a requirement of operating a platform used by 13 million people, some of whom may be in clinical settings when a failure occurs.

The NHS App team are the people who are called when something is broken. They are the most motivated party to ensure that activations are safe, and they are the ones who bear the operational consequences of a bad release. Their unconditional deactivation authority reflects that accountability.

### 5.3 Web channel — rollout mechanics

- Tenant teams define the rollout cohort and request activation; the NHS App team executes the flag change in production. Tenants do not have direct access to production feature flag controls.
- Kill-switches must be registered not just at the feature level, but for every external system integration that the feature depends on

**Current state — Service Journey Rules (SJRs):** Feature rollout today is controlled through a bespoke, hand-coded system called Service Journey Rules. SJRs currently support targeting by ODS code and are being extended to support additional user attributes. They are the mechanism through which content entitlement (which users see which content) and feature activation (whether a feature is on or off) are currently managed.

SJRs are a known constraint:
- They are not a standard, interoperable interface — targeting logic is tightly coupled to their implementation
- Extending them to support new attributes requires platform-level code changes
- There is no self-service capability for tenant teams; all changes require NHS App team intervention
- Visibility of SJR configuration across tenant teams is limited, creating a risk that one tenant's entitlement configuration silently affects another's feature behaviour

All SJRs must be visible to all tenants through a platform-provided dashboard. This is a current gap and a known risk — no tenant should be surprised by another tenant's entitlement configuration affecting their feature's behaviour.

**Target state — OpenFeature:** The long-term target is a feature flagging system built on the [OpenFeature](https://openfeature.dev/) standard. OpenFeature is a CNCF project that defines a vendor-neutral API for feature flagging, allowing the underlying flag evaluation engine to be swapped without changing application code. The benefits for this platform are significant:

- Tenant code references a standard SDK, not a proprietary NHS App integration
- Targeting rules (ODS code, user attributes, percentage rollout, A/B variants) are expressed as structured, portable evaluation context — not hand-coded conditions
- The flag evaluation backend can evolve (or be replaced) without touching tenant code
- Self-service flag management becomes achievable without platform-level code changes

**Tension to resolve:** The migration from SJRs to an OpenFeature-compatible system is not yet planned. Tenant teams onboarding now will build against the SJR model. The platform will need a migration path that does not require tenants to re-implement their rollout logic when the underlying system changes. See Appendix A, item A12.

### 5.4 Native channel — rollout mechanics (iOS and Android)

The native channels have fundamentally different release mechanics and require a different set of planning assumptions:

- **Deployment to production is release.** Once a build passes App Store review and is published, it is live for users who update. There is no "deploy then decide whether to release" for native in the way web provides it.
- **Kill-switches are the primary post-release risk lever.** Because a published app version cannot be retracted, the kill-switch — a feature flag that disables the feature remotely without requiring a new submission — is the most important risk mitigation tool available. Every native feature must have its kill-switch tested and confirmed working in staging before the build is submitted for App Store review. A build without a tested kill-switch will not be approved for submission.
- **Staged rollout is the default for significant releases.** Both the App Store and Google Play support staged rollout by percentage of users. For any feature release of material scope, staged rollout starting at a small cohort percentage is the expected approach — not an option.
- **Clinical assurance, IG, and InfoSec gates happen before App Store submission.** Once a build is in users' hands, the available response options are kill-switch and new submission. Those are both inferior to not shipping a problem in the first place. All gates must be passed before submission, not after.
- Feature flags scoped to device or session are the mechanism for progressive post-publication rollout where staged App Store rollout is insufficient.

### 5.5 Release gates

The Feature Activation Criteria in §1.5 define what must be true before any feature is activated. This table is the channel-specific expression of those criteria — it shows which gates apply to which delivery channel and, for native, at which point in the submission process they must be satisfied.

No feature on any channel releases without satisfying all applicable gates. These are not advisory checkboxes — they are hard gates. A missing gate is a blocker.

| Gate | Web | iOS | Android |
|------|:---:|:---:|:-------:|
| All automated tests passing | ✅ | ✅ | ✅ |
| Accessibility audit (WCAG 2.2 AA) | ✅ | ✅ | ✅ |
| Information Security sign-off | ✅ | ✅ | ✅ |
| Information Governance sign-off | ✅ | ✅ | ✅ |
| Clinical assurance (where applicable) | ✅ | ✅ pre-submission | ✅ pre-submission |
| Performance SLO baseline established | ✅ | ✅ | ✅ |
| Kill-switch tested end-to-end | ✅ | ✅ | ✅ |
| Runbook published to knowledge base | ✅ | ✅ | ✅ |

---

## Part 6 — Team Responsibilities and Onboarding

### 6.1 Phase 0 — Dogfooding and first external pilot

Phase 0 runs two streams in parallel:

1. **Internal dogfooding:** an NHS App internal team acts as a tenant — going through every step from initial engagement through to a feature live in production — using only the tooling, documentation, and support structures this Playbook describes.
2. **First external pilot:** a single, carefully selected external tenant team goes through the same process at the same time, under close support from the NHS App team.

Running these streams in parallel is deliberate. Internal dogfooding surfaces the gaps that the authors cannot see because they built the process; the external pilot surfaces the gaps that only someone new to it will hit. Neither stream alone is sufficient.

The external pilot team is not a guinea pig — they are a close collaborator with skin in the game. They have a real feature to deliver and real users to serve; the platform has to actually work for them, not just in theory. They will receive a level of direct support that will not be available to subsequent tenant cohorts. In return, they are expected to be candid about every point of friction, confusion, or missing tooling they encounter.

Required outputs of Phase 0:

- An updated version of this Playbook reflecting every gap, ambiguity, or friction point found during both streams
- Validated onboarding tooling and documentation (both web and native tracks)
- Completed and reviewed onboarding checklists for both tracks
- A retrospective with the external pilot team, documenting what worked and what did not
- A clear, explicit go/no-go decision for the first broader external tenant cohort

**Why this matters:** Asking external teams to follow a process the NHS App team has never followed themselves is disrespectful of their time and will surface problems at the worst possible moment. Dogfooding catches the obvious issues; a real external team catches the ones that feel obvious to insiders but aren't. The first broader cohort should arrive at an onboarding experience that has already been improved by at least two complete pilot runs.

### 6.2 What the NHS App team centrally owns

The following capabilities and responsibilities are owned by the NHS App team permanently. They will not be delegated to tenant teams.

- Design system and component library (web and native)
- Infrastructure, platform hosting, and Kubernetes configuration
- CI/CD pipeline definitions and templates
- Code standards tooling and automated standards enforcement
- BAU security patching for platform components
- Platform-level penetration testing and ongoing security posture management
- Service management wrapper (incident coordination, communications, escalation)
- Clinical assurance process coordination
- Performance testing infrastructure
- Feature flagging infrastructure
- Observability and analytics infrastructure (Splunk, Databricks, Adobe Analytics)
- Staging and preview environments
- Software Bill of Materials (SBOM) — in close collaboration with tenant teams
- Integration with internal suppliers (NHS Login, NHS Notify, and others)
- This Playbook — ownership, ongoing improvement, and version control

### 6.3 What tenant teams own

- All functional feature code
- All tenant backend services and their runtime configuration
- Incident first response for their own features
- Full automated test coverage for their code (unit, integration, contract, end-to-end)
- Their feature's risk register and control mapping
- Their runbooks and knowledge base entries
- Their SBOM entries
- Their on-call rota and escalation contacts, registered with R&M

### 6.4 Onboarding path

All features are expected to deliver across all three channels — **Web**, **iOS**, and **Android**. The onboarding process reflects this: every team goes through both the Web track and the Native track.

Delivering on a single channel only — for example, piloting on web before building native — requires an explicit, time-bound exception agreed with the NHS App platform team at initial engagement. An exception is not a permanent position; it must include a committed plan and timeline for reaching full channel coverage.

#### Both tracks — initial steps

1. **Initial engagement:** contact the NHS App platform team via [TBD channel] and describe your service, its scope, and your target delivery timeline
2. **Design review checkpoint:** attend a design review session with the NHS App architecture team before writing any code. This is where architectural decisions are made, not at PR review.
3. **Technical readiness assessment:** complete the readiness checklist to confirm your team has the prerequisites: relevant language and framework skills, tool access, IG clearances, and a nominated on-call contact registered with the NHS App team
4. **Repository and access provisioning:** NHS App team provisions your repository from the correct template, CI/CD pipeline access, and staging environment access
5. **ADR for integration approach:** write and obtain approval for the Architecture Decision Record describing how your feature integrates with the platform, before implementation begins

#### Web track — onboarding checklist

- [ ] Micro-frontend repository created from NHS App web template
- [ ] CI/CD pipeline running end-to-end: lint → unit test → integration test → build → deploy to staging, triggered on every commit
- [ ] Design system dependency configured to track `-latest`
- [ ] `CODEOWNERS` file populated with current team members
- [ ] Feature flag registered in the platform feature flag service
- [ ] Kill-switch tested in staging (feature disabled and re-enabled remotely without a deployment)
- [ ] BFF/adapter service deployed to staging and integration-tested
- [ ] Performance baseline measurement run and SLO agreed with NHS App platform team
- [ ] Monitoring and alerting configured in Splunk, with thresholds aligned to agreed SLO
- [ ] Runbook drafted and reviewed by the NHS App team before first production release
- [ ] SJR and entitlement configuration reviewed with the NHS App team
- [ ] SBOM entry created and registered

#### Native track — onboarding checklist (iOS and Android)

- [ ] Native module and bundle integration approach agreed with NHS App native platform engineer before implementation begins
- [ ] CI/CD pipeline running end-to-end: lint → unit test → integration test → build → deploy to staging, automated with no manual steps
- [ ] Feature flag registered in the platform feature flag service, with kill-switch verified against a staging build (remote disable and re-enable confirmed)
- [ ] Combinatorial flag testing matrix defined; all tested combinations logged with an audit trail
- [ ] BFF/adapter service deployed to staging and integration-tested
- [ ] App Store (Apple) and Google Play developer access confirmed and app signing configuration in place
- [ ] Staged rollout percentage agreed (starting cohort — do not default to 100%)
- [ ] Clinical assurance and IG/InfoSec gates passed before first App Store submission
- [ ] Performance baseline established and SLO agreed
- [ ] Monitoring and alerting configured
- [ ] Runbook drafted and reviewed by the NHS App team before first App Store submission
- [ ] SBOM entry created and registered
- [ ] Supported app version range agreed with NHS App native platform engineer

#### Going live (both tracks)

> **Note:** "Going live" means **feature activation** — flipping the flag so users can see the feature. It does not mean the first deployment. Code must be deployed to staging behind a feature flag that defaults to off from the very first commit. By the time a team reaches this checklist, the feature has been deployed and running hidden in production for some time. These steps govern the decision to make it visible.

1. **Feature Activation Criteria met:** confirm all items in §1.5 are satisfied. This is the gate. Nothing below this point is valid if any §1.5 item is outstanding.
2. **Staging validation:** full integration test on the shared staging environment with the feature flag on, with a go/no-go sign-off from both the tenant team and the NHS App platform team.
3. **NHS App team briefing:** brief the NHS App team on your feature — its kill-switch location and operation, escalation contacts, known risks, and any platform dependencies — before activation. Do not assume the NHS App team will find this information in a runbook under incident conditions.
4. **Phased activation:** start with a restricted cohort (see §5.3 and §5.4). Do not activate to 100% of users on day one under any circumstances.
5. **Post-activation monitoring period:** a mandatory monitoring window during which the tenant team is in active monitoring posture. The duration of this window is agreed at onboarding based on feature scope and risk. The feature is not considered stable until the monitoring window closes with no significant issues.

### 6.5 When you get stuck

| Situation | Contact |
|-----------|---------|
| A Playbook rule that seems wrong or inapplicable to your context | NHS App platform team — raise it formally; rules that fail the Epilogue criteria should be challenged |
| CI/CD pipeline or access problem | NHS App infrastructure team (infrastructure support channel) |
| Design system question | Design system team (design channel) |
| IG or InfoSec query | NHSE InfoSec / SIRO office via the standard route |
| Clinical assurance query | NHS App clinical safety officer |
| Performance or observability question | NHS App platform engineering (observability channel) |
| Escalation — blocked with no response after a reasonable period | Principal Systems Engineer, NHS App |

---

## Part 7 — Commercials and Funding

The commercial model underpinning this platform matters because the wrong model produces the wrong behaviours. This section states our positions clearly.

### 7.1 The platform is centrally funded

The NHS App platform team is funded centrally as an **enabling platform**. It does not charge tenant teams for platform usage. The platform team's mandate is to make tenant teams successful — not to operate as a cost centre recovering spend from those it serves.

**Why this matters:** A platform team that charges per deployment, per API call, or per feature hosted creates adversarial incentives. Tenant teams cut corners on monitoring, delay updates, and find reasons to resist standards when every engineering decision has a price tag attached. The platform and the tenant teams are on the same side.

### 7.2 Tenant teams own their own infrastructure costs

Each tenant team is responsible for the running costs of their own backend services — compute, storage, and any data transfer attributable to their services.

**Note:** The commercial position here depends on an architectural question that is not yet fully resolved: whether tenant backend services run on NHS App-provisioned infrastructure within the NHS Azure tenant, on tenant-owned infrastructure within NHS Azure, or on tenant-owned infrastructure outside NHS Azure. Each model has a different cost attribution structure. See [Tenant Compute: Options and Tensions](../open-discussions/tenant-compute-options.md) and Appendix A, item A10 for the full analysis. The principle below holds regardless of which model is adopted — tenant teams pay for their own services — but the mechanism for attributing and settling those costs will differ by model.

This keeps the commercial model clean: the platform team's costs are for the platform; the tenant team's costs are for their service. When a service is shut down or a project ends, the shutdown is clean and the costs stop with it.

**Why this matters:** Central ownership of all infrastructure costs creates a situation where abandoned features and abandoned services continue to generate cost with no owner motivated to challenge it. Distributed cost ownership is a natural forcing function for feature lifecycle management — a team whose budget is paying for a running service has a direct incentive to either maintain it or shut it down.

### 7.3 Named anti-patterns

The following funding and commercial arrangements are **not compatible** with this model. Teams or contracts that exhibit these patterns will be asked to change them before onboarding proceeds.

| Anti-pattern | Why it is harmful |
|--------------|------------------|
| **Fixed-scope, fixed-cost contracts for tenant features** | Forces delivery of the agreed scope regardless of whether that is the right thing to build. Quality and adaptability take second place to contract compliance. |
| **Loaning centralised NHS App specialist capacity to tenant teams** | Creates dependency rather than building tenant team capability. Does not set areas up for long-term self-sufficiency. |
| **Per-deployment or per-feature platform charges** | Creates adversarial incentives between the platform team and the teams it is supposed to enable. |
| **Tenant teams configuring or modifying shared platform infrastructure** | Giving tenants access to shared infrastructure (Kubernetes cluster config, pipeline templates, observability platform) leads to inconsistency, security fragmentation, and incident blast radius that crosses tenant boundaries. This is distinct from a tenant team owning and operating *their own* backend infrastructure — which may be the right model depending on the compute decision (see Appendix A, item A10 and [Tenant Compute: Options and Tensions](../open-discussions/tenant-compute-options.md)). |

### 7.4 What happens when funding ends

When a tenant team's project funding ends or is redirected:

1. The tenant team must notify the NHS App platform team immediately upon confirmation that the funding change is happening.
2. A handover plan must be agreed before the team disbands: either ownership transfers to a named team, or the feature enters the formal retirement process (see Part 1.4).
3. Running costs for the tenant's backend services must be attributed and settled before offboarding is complete.
4. **There is no "the NHS App team takes it over" option.** The NHS App cannot absorb the operational cost and cognitive load of features whose owning teams have dematerialised.

**Why this matters:** The NHS App team should never be in the position of operating a feature they did not build and have no runbook for. Good portfolio management at programme level means making the decision about a feature's future before the team responsible for it has already left.

---

## Appendix A — Open Questions

The following items arose during the discussions that shaped this Playbook. They require explicit decisions before the Playbook can be considered final for external onboarding.

| # | Question | Area | Suggested owners |
|---|----------|------|-----------------|
| A1 | What is the formal process and authority for retiring an unmaintained feature, including the notice period and who holds the "delete button" at portfolio (P&P) level? | Social Contract | NHS App leadership, P&P |
| A2 | What service tier (currently Silver+) applies to tenant-owned features in production? Does any tenant feature require Platinum tier, and if so, what triggers that assessment? | Service management | NHS App, SIRO |
| A3 | Do tenant teams have any formal out-of-hours on-call obligation? If a tenant feature causes an out-of-hours P1 and the tenant team is unreachable, what is R&M's escalation protocol? | Social Contract, Incident | R&M, NHS App |
| A4 | Analytics: is there a viable "new aggregation layer" exception for tenants with specific analytics requirements, and if so, what is the formal process for requesting and reviewing it? | Platform Subsidy | NHS App, IG |
| A5 | When multiple tenant teams are queued for deployment or release activation, what is the prioritisation process and who arbitrates conflicts? | Release Choreography | NHS App (R&M) |
| A6 | BFF ownership: does each tenant team own and operate their own BFF service, or do all requests go through a single NHS App-owned BFF? What are the security, performance, and operational ownership implications of each approach? | Architecture | NHS App architecture |
| A7 | Release gate authority: who holds the authority to grant or refuse a release gate waiver — is this always the NHS App platform team, or can that authority be delegated to senior members of a tenant team? How federated can this become without losing coherence? | Release Choreography | NHS App |
| A8 | Cost attribution: even with central platform funding, what cost attribution reporting should exist so that programme-level decisions can be made about the value of a tenant feature relative to its running cost? | Commercials | NHS App, Finance |
| A9 | Pipeline templates — constraint or enabler? Standardised CI/CD pipeline templates guarantee a minimum bar of risk management and help teams move quickly from a standing start. But they also constrain capable teams who may have legitimate reasons to do things differently, and risk becoming a ceiling rather than a floor. Should the template model be strictly enforced (teams must use the template, period), or should there be a formal mechanism for a team to propose and adopt alternate pipeline implementations that demonstrably meet or exceed the template's quality and risk management guarantees — without requiring a full exemption process? | Technical Guardrails, Team Evolution | NHS App architecture, senior tenant engineering leads |
| A10 | Tenant compute model: should the platform mandate that all tenant services run on NHS App-provisioned compute within NHS Azure (current position), allow tenant teams to operate their own compute within NHS Azure, or allow tenant compute outside NHS Azure entirely? The decision has direct knock-on effects on IG and data governance, the BFF architecture and threat model, observability continuity, platform pen testing scope, and tenant onboarding speed. Micro-frontend bundle hosting (static assets, no patient data) may warrant a separate decision from backend service compute. See [Tenant Compute: Options and Tensions](../open-discussions/tenant-compute-options.md) for full analysis. | Platform Subsidy, Architecture, IG | NHS App architecture, NHS App IG lead, NHS England SIRO office |
| A11 | Identity propagation through the BFF: the BFF validates NHS Login sessions at the platform boundary, but tenant backend services will typically need verified user identity to serve the correct data. What is the standard mechanism for propagating identity from the BFF to tenant backends — forward the original NHS Login JWT, perform a token exchange for a service-scoped token, or issue a signed internal identity assertion? Each option has different security, coupling, and operational implications. This decision needs to be made before the first tenant backend integration is built and must be consistent across all tenants. | Architecture, InfoSec | NHS App architecture, NHS Login team |
| A12 | Feature flagging system migration: the platform currently uses Service Journey Rules (SJRs) — a bespoke, hand-coded system for content entitlement and feature activation. The target is a system built on the OpenFeature standard, which provides a vendor-neutral SDK API and decouples tenant code from the underlying flag evaluation engine. What is the migration path? How do tenants onboarding during the SJR era avoid being required to re-implement their rollout logic when the platform migrates? Should OpenFeature be adopted incrementally (new features use it from day one) or via a full platform cut-over? | Platform evolution, Tenant onboarding | NHS App platform team, senior tenant engineering leads |
