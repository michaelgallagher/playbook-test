# Micro-frontends: Direction of travel for the NHS App web channel

> **Status:** Supporting reference — supports the [technical Playbook](../guides/technical-playbook.md), section 2.5
> **Date:** March 2026

---

## Current reality

The NHS App web application is currently a **monolithic Vue application** maintained in a single repository by the NHS App team. There is no existing micro-frontend infrastructure. The rules in §2.5 of the Playbook describe the intended target architecture — not the state tenants will find when they first arrive.

This matters because the monolith creates a delivery constraint that the Playbook's model is designed to eventually eliminate: today, a tenant contributing a web feature must do so within the single shared codebase, which means their code, their pipeline, and their release cadence are tightly coupled to everyone else's.

This document explains what micro-frontends are, why they are the right direction for the NHS App platform model, what the migration path looks like, and what tenant teams should expect during the transition.

---

## What is a micro-frontend?

A micro-frontend applies the microservices principle to the browser. Instead of a single application bundle that owns all routes and all UI, the shell application loads independently-built and independently-deployed UI modules at runtime.

In practice for the NHS App:

- The **shell** (owned by the NHS App team) handles authentication, navigation, and routing. It is responsible for loading the correct micro-frontend for a given route.
- Each **micro-frontend** (owned by a tenant team) is a self-contained UI component or section — built, tested, versioned, and deployed independently by the team that owns it.
- The shell consumes each micro-frontend as a remote module, fetched at runtime from a CDN or artefact store.

The tenant team's repository, CI/CD pipeline, and deployment lifecycle are entirely separate from the shell and from other tenants. A tenant can deploy a new version of their feature without any coordination with the NHS App team or with other tenants — as long as their module satisfies the integration contract with the shell.

---

## Why micro-frontends are the right architecture for this model

The Landlord-Tenant model described in the Playbook requires genuine delivery independence for tenant teams. A monolithic front end makes that impossible in several ways:

| Problem with a monolith | How micro-frontends address it |
|------------------------|-------------------------------|
| All teams share a single pipeline — one team's breakage blocks everyone | Each team has their own pipeline; failures are isolated |
| Release coordination required across all teams for every deployment | Each team deploys independently; no coordination required |
| A security or accessibility issue in one area can block all deployments | Issues are contained to the affected micro-frontend |
| Growing codebase becomes harder to reason about and test | Each module is small, independently testable, and owned by one team |
| Monorepo tooling complexity grows with team count | Team repositories are independent by default |

The flip side is that micro-frontends introduce their own complexity — runtime composition, versioning contracts between shell and modules, consistent design system application across independently-built bundles. These are the problems the Playbook's rules (tracking `-latest`, enforced design system compliance, contract testing) are designed to manage.

---

## The technical approach: Module Federation

The recommended implementation is **Webpack Module Federation** (or the Vite equivalent via [vite-plugin-federation](https://github.com/originjs/vite-plugin-federation)), which allows:

- The shell to declare **remote** modules it will consume at runtime
- Each tenant micro-frontend to **expose** components or routes as federated modules
- Shared dependencies (Vue, the design system) to be declared as **singletons** — loaded once by the shell and shared across all micro-frontends, avoiding version conflicts and bundle bloat

A tenant's micro-frontend exposes a route component. The shell loads it on demand when the user navigates to that route. From the user's perspective the application is seamless; from the engineering teams' perspective each piece is independently owned and deployed.

### Key constraints of Module Federation

- **Shared singleton dependencies must be version-compatible.** If the shell uses Vue 3.4 and a micro-frontend declares Vue 3.2 as a shared dependency, Module Federation will attempt to negotiate a compatible version. This is why tenants must track `-latest` on curated internal dependencies — pinning creates the risk of an incompatible singleton negotiation at runtime.
- **The integration contract between shell and micro-frontend must be stable.** The shell declares what routes it will attempt to load and from where. If a micro-frontend changes its exposed module name or entry point, the shell breaks. This is a breaking change and must follow the versioning and deprecation rules in §4.2 of the Playbook.
- **Runtime errors in a micro-frontend must not crash the shell.** The shell must implement error boundaries so that a failing micro-frontend degrades gracefully — showing a fallback state rather than taking down the whole application.

---

## Migration path: from monolith to micro-frontends

The migration is not a big-bang rewrite. The recommended approach is **strangler fig** — extracting features incrementally into micro-frontends while the monolith continues to serve everything else.

### Phase 1 — Shell extraction (NHS App team)
The NHS App team extracts the shell (authentication, navigation, routing) from the monolith and establishes the Module Federation configuration. The monolith becomes a single large "legacy" micro-frontend loaded for all routes not yet extracted.

### Phase 2 — Greenfield tenants
New tenant features are built as micro-frontends from day one, using the onboarding template. These do not touch the monolith at all.

### Phase 3 — Existing feature extraction
Existing features in the monolith are extracted into micro-frontends, owned by their respective teams, on a feature-by-feature basis. The legacy micro-frontend shrinks as features are extracted.

### Phase 4 — Monolith retirement
Once all features have been extracted, the legacy micro-frontend container is retired. The monolith no longer exists as a single deployable unit.

---

## What this means for tenant teams onboarding during the transition

Tenant teams onboarding **before the shell extraction is complete** will be working within the monolith. This means:

- Their code lives in the shared NHS App repository, not their own repository
- Their pipeline is the shared monolith pipeline
- Their releases are coupled to the overall application release cadence
- The "independently deployable" model described in §2.5 of the Playbook does not yet apply to them

The NHS App team will communicate clearly when the shell extraction is complete and independent micro-frontend onboarding becomes available. Until that point, the onboarding process for web will follow an interim path agreed during the initial engagement step (§6.4).

The rules in §2.5 (design system compliance, `-latest` tracking, no modification of shell, authentication, or navigation) apply in both the transitional and the target architecture.

---

## Open questions

The following questions are relevant to the micro-frontend programme and will need resolution as the transition progresses:

- What is the target timeline for Phase 1 (shell extraction)? This gates when the first external tenant can use the independent micro-frontend model.
- How are runtime errors in micro-frontends surfaced to the tenant team's monitoring (Splunk)? The shell's error boundary needs to emit structured events that tenant dashboards can consume.
- How is the Module Federation manifest versioned and published? Tenants need a stable way to discover what version of the shell they are integrating against.
- What is the governance process for changes to the shell's integration contract — the exposed routes and shared dependency declarations? These are breaking changes for all tenants and need the full stone-ripple analysis described in §4.1.
