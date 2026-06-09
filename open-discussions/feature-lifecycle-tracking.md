# Feature lifecycle tracking

> **Status:** Supporting reference — supports the [technical playbook](../guides/technical-playbook.md), section 1.4
> **Date:** March 2026

---

## The problem

Section 1.4 of the Playbook commits the NHS App team to auditing active features on a regular cadence and flagging those that appear unmaintained. The signals named there — no recent deployments, open security advisories, unresolved accessibility issues, no identifiable owning team — are the right ones.

But at the scale this platform is targeting (potentially 90+ tenant teams), a manual audit is not a viable mechanism. The people running it will miss things, the cadence will slip, and the data will be stale before it's acted on.

This document describes how that audit process should be automated, what models exist for doing it, and what tooling choices are available.

---

## The standard model: Software Catalog + Scorecards

The established approach in platform engineering is an **Internal Developer Portal (IDP)** with a **software catalog** as its foundation.

### Software catalog

Every feature, service, and backend component is a **registered entity** in the catalog. Each entity carries machine-readable metadata:

- **Owner** — the team responsible (maps to an entry in the organisation's identity system)
- **Lifecycle state** — one of the states defined in the Playbook: `proposed`, `in-development`, `in-review`, `active`, `maintained`, `retired`
- **Type** — e.g. `web-microfrontend`, `native-module`, `bff-service`, `api`
- **Links** — to the source repository, runbook, monitoring dashboard, and SBOM entry
- **Dependencies** — which platform APIs, design system versions, and external systems the feature uses

This catalog becomes the authoritative answer to "what is running in this platform, and who owns it?" — a question that is currently difficult to answer quickly and reliably.

### Scorecards

Layered on top of the catalog, **scorecards** (also called Tech Insights, Service Maturity, or Service Health checks depending on the tool) define automated rules that are evaluated continuously against every registered entity.

A scorecard rule is a question that can be answered from machine-readable signals:

| Rule | Source signal |
|------|--------------|
| Has there been a deployment in the last 30 days? | Deployment frequency from CI/CD (GitHub Actions, Azure DevOps) |
| Are there open security advisories without a linked remediation? | Dependabot / Snyk alerts without a linked PR or issue |
| Is the `CODEOWNERS` file non-empty and resolvable? | GitHub API — do the referenced teams and users still exist? |
| Are all SLO thresholds currently passing? | Splunk / monitoring platform |
| Are there unresolved accessibility failures in the pipeline? | CI/CD pipeline test results |
| Is the design system dependency tracking `-latest`? | Package manifest — is the pinned version the current release? |
| Is the runbook present and recently reviewed? | Confluence / knowledge base last-modified date |
| Is the SBOM entry current? | SBOM registry |

Features that fail scorecard rules surface automatically in a dashboard. The NHS App team does not need to run a periodic manual audit — the audit runs continuously.

---

## Mapping to the §1.4 lifecycle states

The scorecard model maps directly onto the lifecycle described in the Playbook:

```
Proposed → In development → In review → Active → Maintained → Retired
                                                      ↑
                              Scorecard failures here trigger the §1.4 escalation process
```

A feature in `active` or `maintained` state that begins failing scorecard rules is automatically flagged. The §1.4 escalation process (Notice → Escalation → Retirement) is then triggered by that flag, not by a human remembering to look.

Retirement is recorded in the catalog as a state change to `retired`. The entry is preserved for audit purposes but is excluded from active monitoring.

---

## Tooling options

### Backstage (open source — Spotify)

[Backstage](https://backstage.io) is the market-leading open source IDP and the most directly relevant option:

- Software catalog with lifecycle state, owner, and dependency tracking built in
- **Tech Insights** plugin provides the scorecard/rule engine
- Native integrations: GitHub (CODEOWNERS, deployment frequency), Dependabot/Snyk, PagerDuty, and most CI/CD systems
- Already in use within parts of NHS England — there may be an existing instance to build on rather than starting from scratch

Backstage also addresses other Playbook requirements naturally:
- The **API discovery catalogue** (§4.3) is a first-class feature of Backstage
- The **ownership maps** (§4.6) are derived automatically from catalog metadata
- The **design system playground** (§4.3) can be embedded as a Backstage plugin

The trade-off: Backstage requires meaningful investment to set up and maintain. It is a platform product in its own right, and someone needs to own it.

### Cortex (commercial)

[Cortex](https://www.cortex.io) is a commercial IDP with a stronger out-of-the-box scorecard and service maturity model. Less setup than Backstage for the catalog + scorecard combination, with a hosted SaaS option that reduces operational overhead.

### OpsLevel (commercial)

[OpsLevel](https://www.opslevel.com) takes a similar scorecard-first approach to Cortex. Particularly strong on the "service maturity" framing — teams see their own scorecard and are nudged to close gaps, rather than only the platform team seeing failures.

### Port (commercial, newer entrant)

[Port](https://www.getport.io) has a more flexible data model than the other options, which suits a platform that doesn't yet have a fixed service taxonomy. Easier to get running quickly. Useful if the shape of the catalog entities is still being defined.

---

## The role of the SBOM

The Playbook already requires an SBOM entry for every feature (§1.5 Definition of Done). A well-formed SBOM is a natural anchor for lifecycle tracking:

- It declares component ownership and provenance
- It can be ingested by **Dependency-Track** (open source) or **Snyk** to provide continuous CVE monitoring against registered components — one of the key unmaintained-feature signals

The SBOM entry in the catalog and the SBOM entry in the security tooling should be the same artefact, generated once and consumed by both.

---

## Dependency-Track and SBOM security monitoring

[Dependency-Track](https://dependencytrack.org) (OWASP project, open source) ingests SBOMs and monitors them continuously against the NVD, GitHub Advisories, and other vulnerability databases. When a new CVE is published for a component in a registered SBOM, Dependency-Track raises an alert — regardless of whether the owning team has deployed recently.

This is the automated mechanism for the "open security advisories without a remediation plan" signal in §1.4, and it operates across the entire portfolio continuously without requiring a deployment or pipeline run to trigger a scan.

---

## Key design decision

The main choice is between:

1. **Build on Backstage** — higher setup cost, but addresses the catalog, scorecard, API discovery, ownership maps, and design system playground requirements from a single platform. Investment is justified if the NHS App platform team has the capacity to own and operate it.
2. **Derive signals from existing tooling** — use GitHub, Azure DevOps, Splunk, and Snyk data directly, without a dedicated portal. Lower setup cost but produces a fragmented picture, requires custom aggregation work, and doesn't serve tenant teams directly with self-service visibility.

At a scale of 90+ tenant teams, the investment in option 1 is likely to pay for itself quickly — the audit burden it removes is significant, and it makes the "who owns this?" question answerable in seconds rather than requiring an investigation.

The recommendation is to treat the IDP as a platform component in its own right, owned by the NHS App team, and to address it during or immediately after the Phase 0 internal pilot (§6.1) — so that the first external tenant cohort arrives at a platform where their feature is registered from day one.

---

## Relationship to other Playbook sections

| Playbook section | How this tooling serves it |
|-----------------|---------------------------|
| §1.4 Feature lifecycle | Automated scoring replaces manual audit; lifecycle state tracked in catalog |
| §1.5 Feature Activation Criteria | Feature activation checklist items can be scorecard rules — completion is verified, not asserted |
| §4.3 Discoverability | API catalog and capability map are catalog features, not separate documents |
| §4.6 Ownership maps | Ownership maps are derived from catalog metadata — always current |
| §6.2 What the NHS App team owns | The IDP/catalog is a platform capability the NHS App team owns and operates |
