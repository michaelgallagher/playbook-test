# Test coverage quality

> **Status:** Supporting guidance — supplements the [technical playbook](../technical-playbook.md), section 2.1
> **Date:** March 2026

---

## The problem

Section 2.1 of the Playbook requires that "all automated test suites pass" as a condition of a safe change. That condition is necessary but not sufficient. A suite of tests that passes consistently but makes no meaningful assertions provides no safety signal at all — it just provides false confidence.

Coverage metrics are the most common proxy for test quality. They are useful but routinely misunderstood. **Coverage measures what was executed, not what was verified.** A test that calls a function and asserts nothing will contribute to line coverage. A feature with 90% coverage can still have critical paths entirely unverified.

This document describes how the platform addresses the gap between "tests pass" and "tests are trustworthy."

---

## Three levels of test quality

It helps to think about test quality at three levels:

| Level | Question | How to measure |
|-------|----------|---------------|
| **Existence** | Do tests exist and pass? | CI pipeline: test run results |
| **Coverage** | Do the tests execute the code? | Coverage tooling: line, branch, statement % |
| **Effectiveness** | Do the tests detect faults? | Mutation testing: mutation score % |

The Playbook's §2.1 currently gates on level 1. The platform requires all three levels.

---

## Coverage thresholds

Coverage thresholds are enforced as hard pipeline gates — not reported as advisory metrics. A build that falls below threshold cannot be merged.

### What is measured

- **Line coverage** — the percentage of executable lines reached by at least one test
- **Branch coverage** — the percentage of decision branches (if/else, switch cases, ternaries) covered by tests. More meaningful than line coverage for conditional logic
- **Statement coverage** — similar to line coverage; redundant if line coverage is already measured, but included for completeness in some tooling

### What is not measured by coverage alone

Coverage cannot tell you whether the assertions in a test are correct, meaningful, or even present. A test that calls `renderComponent()` and then immediately passes will show green coverage against every line in that component.

This is why mutation testing is required alongside coverage metrics.

---

## Mutation testing

Mutation testing is the strongest automated signal available for test suite effectiveness. Tools like [Stryker](https://stryker-mutator.io) (JavaScript/TypeScript — directly relevant to the Vue micro-frontend architecture) deliberately introduce small faults into the production code and then run the test suite against the mutated version.

Example mutations:
- `if (age > 18)` becomes `if (age >= 18)`
- `return true` becomes `return false`
- `count++` becomes `count--`
- A conditional block is removed entirely

If the test suite fails when a mutation is present, the mutation is **killed** — the tests are detecting that fault. If the test suite passes despite the mutation, the mutation **survives** — the tests are not catching that class of fault.

The **mutation score** is the percentage of mutations killed:

$$\text{mutation score} = \frac{\text{mutations killed}}{\text{total mutations}} \times 100$$

A high mutation score is strong evidence that the test suite would catch the kinds of faults that mutations represent. It is not a guarantee — no automated metric is — but it is a substantially stronger signal than coverage alone.

### Stryker for the Vue micro-frontend architecture

Stryker has first-class support for Vue 3 and Vitest/Jest. It is the recommended mutation testing tool for web channel tenant teams. Configuration is provided in the platform CI/CD pipeline templates.

For native (iOS and Android), equivalent mutation testing approaches exist per platform:
- **iOS (Swift):** [Muter](https://github.com/muter-mutation-testing/muter)
- **Android (Kotlin/Java):** [PiTest](https://pitest.org) (via Kotlin interop) or [Kotlin mutation testing](https://github.com/JetBrains-Research/kotlinRMiner) tooling

---

## Contract testing for integration boundaries

Unit and integration tests verify behaviour within a service boundary. They cannot verify that the BFF's actual API behaviour matches what the front end expects at runtime.

**Consumer-driven contract testing** — using [Pact](https://pact.io) — fills this gap:

1. The consumer (front-end micro-frontend or native module) defines a **contract**: the requests it will make and the responses it expects
2. The provider (BFF service) verifies that it satisfies those contracts in CI, without needing the consumer to be running
3. Contracts are published to a **Pact Broker** (provided as a platform service) and verified on every provider build

This means that if the BFF changes in a way that breaks a consumer's expectations, the provider's pipeline fails — before that change reaches staging or production.

Contract testing is **required** for all BFF-to-frontend integration boundaries. It is not a substitute for integration testing in staging but operates earlier, faster, and with a clearer failure signal.

---

## Test strategy as a design-checkpoint artefact

Coverage thresholds and mutation scores are enforcement mechanisms — they catch gaps after the fact. The more effective intervention is agreeing test strategy at design time, before a line of test code is written.

As part of the design review checkpoint (§2.2 of the Playbook), tenant teams must produce a **test strategy note** covering:

- Which risk controls (from the §2.3 risk register) require which type of test
- Where the highest-risk conditional logic lives and how it will be covered at branch level
- Which integration boundaries require contract tests
- Any areas where mutation score targets may need to be higher than the platform default (e.g. clinical decision logic)

This is not a lengthy document. It is a short artefact that makes test intent explicit before implementation begins — so that coverage gaps are caught at design, not at PR review or in production.

---

## Platform minimum thresholds

The following are the platform minimum thresholds, enforced as hard pipeline gates. Tenant teams may set higher thresholds for their own features; they may not set lower ones without a formally approved, time-bound exemption.

| Metric | Minimum threshold | Notes |
|--------|------------------|-------|
| Branch coverage | 80% | Applied per-module, not as an aggregate |
| Mutation score | 70% | Applied to business logic; UI-only rendering code may be excluded with justification |
| Contract tests | 100% of BFF integration boundaries | No exceptions; all boundaries must have a Pact contract |

These thresholds will be reviewed after the Phase 0 internal pilot (§6.1 of the Playbook) and may be revised upward for subsequent tenant cohorts based on what the pilot reveals.

---

## Ownership

| Responsibility | Owner |
|---------------|-------|
| Platform pipeline tooling (Stryker config, Pact Broker, coverage reporters) | NHS App team |
| Setting and maintaining the minimum threshold values | NHS App team |
| Test strategy for a feature's risk-register controls | Tenant team |
| Achieving and maintaining thresholds for their feature's code | Tenant team |
| Test strategy review at design checkpoint | NHS App team (reviewer) + Tenant team (author) |

The NHS App team owns the bar; tenant teams own clearing it. The NHS App team does not write tests for tenant features, and passing tests written by the NHS App team does not constitute the tenant team having satisfied the coverage requirement.

---

## Relationship to other Playbook sections

| Playbook section | How this guidance relates |
|-----------------|--------------------------|
| §2.1 What makes a change safe | This document defines how "all automated test suites pass" is made trustworthy |
| §2.2 Shift left | Test strategy note is a design-checkpoint artefact |
| §2.3 Risk management process | Controls in the risk register must be traceable to tests; mutation score verifies those tests are effective |
| §6.4 Onboarding checklist | Contract test setup and Stryker configuration are part of the pipeline bring-up |
