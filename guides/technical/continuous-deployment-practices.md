# Continuous deployment: Practices and coping strategies

> **Status:** Supporting guidance — supplements the [technical playbook](../technical-playbook.md), section 4.4
> **Date:** March 2026

---

## The goal, plainly stated

The requirement to "deploy on every commit to main" is not a requirement that every user sees every change the moment it is pushed. The Playbook already separates deployment from release (§5.1). The goal is narrower and more achievable: **main should always be in a known-deployed, tested state, and integration problems should be discovered in hours rather than at the end of a sprint.**

NHS England is not currently a continuous deployment organisation, and may never be one in the purist sense. That is fine. The practices in this document describe how to get most of the benefit without requiring a cultural transformation that isn't going to happen overnight.

---

## What is non-negotiable

**Every merge to main must automatically trigger a deployment to the shared staging environment.**

This is the hard floor. It is not continuous deployment to production — it is automated deployment to staging, which is a significantly lower bar. It means:

- The shared staging environment always reflects the current state of main
- Integration failures between a tenant's code and the platform are caught within hours of merging, not weeks later
- There is a known-good staging deployment as the baseline for any release decision

If a team's pipeline cannot reliably deploy to staging on every merge, that is a pipeline problem to fix — not a reason to defer the requirement.

---

## What is aspirational

Automated promotion from staging to production is the long-term target. Teams that achieve it benefit from smaller, lower-risk releases and faster feedback loops. But it is not a precondition for onboarding, and teams not yet there are not in violation of the Playbook.

What is not acceptable is indefinite deferral — "we'll deploy when we're ready." If a team cannot articulate a named cadence and a path toward more frequent deployment, that is a risk to raise, not a default to accept silently.

---

## Feature flags as the primary safety mechanism

The single most effective coping strategy for organisations not yet at continuous deployment is **feature flags used consistently from the start**.

If every piece of in-progress work is deployed behind a flag that defaults to off, then:

- Code can be merged and deployed to production continuously without users seeing incomplete or untested features
- The decision of when to release is calm and deliberate, not forced by deployment state
- A bad deployment can be mitigated by toggling a flag rather than requiring a rollback

This is already a platform requirement (§1.5, §5.1). The point here is that feature flags are not just a release mechanism — they are what makes continuous deployment to production safe enough to attempt. Teams that treat flags as optional are making their own deployment pipeline riskier than it needs to be.

---

## Short-lived branches over rules about commit frequency

A practical proxy for "deploy every commit" that is more tractable in most teams: **no branch should remain unmerged for more than two working days.** (The Playbook's §2.4 rule — PRs open no more than 7 working days — is the outer boundary; the real target is much shorter.)

Long-lived branches accumulate integration risk. A branch that has been open for a week against an active codebase will have a non-trivial merge, hidden conflicts, and assumptions that have shifted. Short branches, merged frequently, avoid all of that.

The cultural target is: small, focused changes, merged fast, deployed immediately to staging. The size of the PR is a better leading indicator than commit count.

---

## Release cadence for teams not yet at continuous deployment

If fully automated production promotion isn't achievable, teams should operate on a named, short cadence with automated gates. Acceptable cadences, in order of preference:

| Cadence | Notes |
|---------|-------|
| Continuous (automated) | Ideal; every staging deployment promotes to production after gates pass |
| Daily | Achievable for most teams with a mature pipeline |
| Per-sprint (weekly or fortnightly) | Acceptable starting point; longer than a fortnight accumulates significant risk |
| Ad-hoc | Not acceptable — "we deploy when we decide to" without a cadence produces unpredictable integration surface |

Whatever cadence a team operates, the automated gates must pass before any production promotion. A cadence is not a substitute for quality gates — it is a schedule for when quality gates are evaluated.

---

## Branch by abstraction for structural changes

Some changes genuinely cannot be hidden behind a feature flag: schema migrations, structural refactors, API contract changes. For these, **branch by abstraction** allows incremental merging while keeping main constantly deployable:

1. Introduce an abstraction layer over the thing you want to change
2. Move all callers to use the abstraction (small, mergeable PRs)
3. Implement the new behaviour behind the abstraction
4. Switch the abstraction to use the new implementation
5. Remove the old implementation and the abstraction layer once the switch is stable

Each step is a small, independently deployable change. The "big refactor" never exists as a single large branch sitting unmerged while the rest of the codebase moves on around it.

---

## Progressive delivery in production

Even when code reaches production, staged rollout decouples "deployed" from "visible to all 13 million users." Progressive delivery — starting with 1% of users, watching the metrics, expanding — is:

- Already the default for native (§5.4)
- Equally available for web via the feature flag cohort model (§5.3)
- The correct default for any significant change regardless of channel

Progressive delivery means that continuous deployment to production and low release risk are not in conflict. The risk is managed by cohort size and monitoring, not by delaying deployment.

---

## Signals that a team's deployment practice has lapsed

The following are observable signals that a team has drifted from the expected deployment practice. They should appear as scorecard failures in the IDP (see [Feature Lifecycle Tracking](../../open-discussions/feature-lifecycle-tracking.md)):

- No staging deployment in the last 7 days
- Open PRs older than 7 working days
- A production release that is significantly behind the staging deployment (indicating code is piling up unshipped)
- Design system dependency pinned to a version older than the current release

These are not immediate violations — they are signals to investigate. The question is always: is this a temporary state with a plan, or has the practice structurally lapsed?

---

## Relationship to other Playbook sections

| Playbook section | How this guidance relates |
|-----------------|--------------------------|
| §2.4 Code and quality standards | PR age limit (7 working days) is the outer boundary; the real target is much shorter |
| §4.4 Web channel — staying current | This document provides the detail behind §4.4's deployment expectations |
| §5.1 Deployment is not release | The separation of deployment from release is what makes frequent deployment safe |
| §5.2 Authority to activate | Feature flag governance; production releases are a separate act from staging deployments |
| Feature lifecycle tracking | Deployment frequency is a key scorecard signal for unmaintained features |
