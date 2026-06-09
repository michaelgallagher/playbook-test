# Playbook Discussions

| Status     | Last reviewed |
| ---------- | ------------- |
| Deprecated | 2026-06-08    |

---

> **Note on extraction:** Text extracted verbatim from the board image. The column headings and Foreword/Epilogue text appear identical across all three participants. Sticky note content within each section represents each person's individual responses and annotations. Some individual sticky notes within the denser clusters were below legible resolution and are marked `[illegible]`.

---

## Column headings (shared across all participants)

1. Foreword
2. Social Contract *(Ownership & Liability)*
3. Technical Guardrails *(Automation over Governance)*
4. Value Proposition *(The "Subsidy")*
5. Team Size Reduction *(Team Evolution)*
6. The "Noise" Floor *(Communication & Discoverability)*
7. Release Choreography *(The Sync Problem)*
8. Commercials *(Funding & Chargeback)*
9. Epilogue

---

## Danny Williams & Dave Lush

### Foreword

We are assuming a Landlord and Tenant model for the NHS App with a defined **Social Contract**.

- **The Landlord (NHS App teams):** You own the "common areas"—the navigation, the authentication, the shell, the design system, and the "plumbing."
- **The Tenants (NHSE Service Owners):** They own the "shops" inside the shopping centre.

Therefore it is natural that the NHS App evolves into an enabling platform rather than a feature factory.

This isn't a cast in stone decision, it's just a starting point to make it easier to reason about the challenge(s).

---

### Social Contract *(Ownership & Liability)*

1. When a feature causes a critical failure, what is the automated response?
2. A team builds a feature, ships it, and then their project funding ends. The feature starts to rot. Who owns the "Delete" button?

**Sticky notes — yellow:**
- If a team ships something and it breaks they fix it
- They have to work within the guardrails and constraints of the app still
- Tech, infosec, accessibility, everything....
- How does support manifest?
- Impractical to have 24/7 support in every squad
- Central team responsible for keeping the lights on
- Not on them to diagnose and fix the problems (think ITIL problem management)
- DoD = knowledgebase and the full service wrapper for shipping a feature

**Sticky notes — green:**
- They may not want to own it... They may have budget and a need? Needs to also cover maintenance

**Sticky notes — orange:**
- Not sure who owns something when it starts to rot...
- Especially if funding dries up for an area
- If something isn't owned, how do we gracefully retire it?

---

### Technical Guardrails *(Automation over Governance)*

1. What defines a "Safe" change that requires zero human intervention from the Core team?
2. How do teams across NHSE ship updates to 13 million MAUs?

**Sticky notes — yellow:**
- Are people allowed to deliver in channels outside of the App / NHS.UK?
- Automated testing
- Risks are managed before code is committed
- Clearly written and comprehensive requirements
- <- MECE
- Clear coding standards well adhered to

**Label:** Risk management of features

**Sticky notes — yellow (Risk management cluster):**
- Identify risks from requirements & solution
- link controls to risks
- the controls are requirements
- automated tests linked back to controls
- Good automated reporting of this to understand gaps & risks

---

### Value Proposition *(The "Subsidy")*

1. What is the "Minimum Viable Platform" we provide that makes a tenant want to work with us?
2. Can a tenant team use their own analytics tool instead of the Platform standard?

**Sticky notes — yellow:**
- Standardised compute (i.e. k8s)
- Standardised service levels
- Within our Azure tenant
- Standard observability
- Standard analytics
- Standard CICD
- Regular platform pen tests etc
- Collaborative environment and support from peers (e.g. #prof-engineering)
- Peer design reviews & community
- Design system & component library

**Sticky notes — orange:**
- Much of this is opaque / not particularly discoverable
- How do I onboard myself into this environment?
- Developer onboarding experience (and ongoing DX)
- Onboarding experience and ongoing of every other role
- Dog food it

---

### Team Size Reduction *(Team Evolution)*

Ultimately, we think an outcome from this model would be that the size of the NHS App team could be reduced (this could be years away though), as we shift from "feature builders" to "enablers":

1. When a tenant team gets stuck on our playbook, who helps them

**Sticky notes — yellow:**
- The current behaviours in the org would not support this evolution
- Highly motivated to see results... Struggle with the capacity & time to deliver it all
- Some areas have reduced capacity significantly... And would leave a gap
- Do we end up with a higher overall cost because of organisational cruft from decentralised ownership?

**Sticky notes — blue (outlined):**
- Lack of knowledge, skills, capacity, time

**Sticky notes — green:**
- We think this would be cheaper
- Each part of the org managing its own channel
- The current model of collaboration

**Sticky notes — orange:**
- Are these all P&P teams?
- These need be portfolio level decisions for the area
- Ongoing investment and support

---

### The "Noise" Floor *(Communication & Discoverability)*

1. When the Platform team updates a core component or API, how do 90 teams migrate?
2. How do teams know what capabilities already exist?

**Sticky notes — yellow:**
- Everyone is part of the same extended team
- Gets the same comms in a timely manner
- Potential platform teams for web and native in the future
- When a non-negotiable update comes along, the teams need to pick up the change... They cannot kick the can down the road
- What timelines are reasonable?
- Need something that helps with discoverability of (1) backed APIs and (2) web / native capabilities
- Clear team based ownership of modules within web and native

**Sticky notes — green:**
- Need clear team based owners for different sub-sections of the web and native applications

---

### Release Choreography *(The Sync Problem)*

1. Who has the authority to toggle a feature off in production?
2. Can a tenant team roll out their feature to only 1% of the 13M MAUs?

**Sticky notes — blue (outlined):**
- Run & maintain for feature activation

**Sticky notes — yellow:**
- These are the people who will get called out if it is broken
- They are the ones who are most motivated to know something is safe to activate
- To tease out with Nick... If it's not R&M, how do we know something is genuinely safe to activate?
- Think R&M are _always_ allowed to deactivate even if they are not the only activators
- For (2). They define the cohorts and R&M presses the button.
- Currently ODS code and percentage of population
- Themes allow this to be sliced on any other data element that can be pulled (decision made in the front end)

---

### Commercials *(Funding & Chargeback)*

1. How is the Platform team funded so they also building features for others?

*[Individual sticky note responses — illegible at available resolution]*

---

### Epilogue

1. Is each decision reversible?
2. Does each decision **increase** or **decrease** the cognitive load of a tenant team?
3. Does each decision **protect** or **endanger** the 13M MAU user experience?
4. Can this decision be **automated**? (If not, why?)

---

---

## Duncan Brown

### Foreword

We are assuming a Landlord and Tenant model for the NHS App with a defined **Social Contract**.

- **The Landlord (NHS App teams):** You own the "common areas"—the navigation, the authentication, the shell, the design system, and the "plumbing."
- **The Tenants (NHSE Service Owners):** They own the "shops" inside the shopping centre.

Therefore it is natural that the NHS App evolves into an enabling platform rather than a feature factory.

This isn't a cast in stone decision, it's just a starting point to make it easier to reason about the challenge(s).

---

### Social Contract *(Ownership & Liability)*

1. When a feature causes a critical failure, what is the automated response?
2. A team builds a feature, ships it, and then their project funding ends. The feature starts to rot. Who owns the "Delete" button?

**Sticky notes — yellow:**
- A screen owned by a tenant causes a critical failure
- Takes out the screen
- Takes out the service
- Blocks a build / deploy
- Comms is a key dependency
- Want comms up in the App asap
- Need a way to pro-actively communicate with the users (N&M?)
- Landlord puts up the sign
- If we have an upstream issue
- Need to be able to bring the app team into the incident process asap
- The tenant would own the incident?
- Service bridge would still own the co-ordindation
- #alerts-high
- #highsev-incident-updates
- Currently just out of hours infra support at tier 3+
- May need to link landlord specialist platform team with tenant stream team
- Does the service need to move to platinum? i.e. beyond silver+
- If something starts to rot... The process for removal needs to link up at P&P level? i.e. for a decision
- If we turn it off, how do we do it?
- If we not the funding to continue must go somewhere
- Would be good to make the boundaries up front for why something might get retired in the face of reduced funding
- Ability to do targeted testing via the app would be very valuable... Again clear lifecycle

**Sticky notes — green:**
- Whoever builds it fixes it!

---

### Technical Guardrails *(Automation over Governance)*

1. What defines a "Safe" change that requires zero human intervention from the Core / Landlord team?
2. How do teams across NHSE ship updates to 13 million MAUs?

**Sticky notes — yellow:**
- We want things to fail as quickly as possible
- Off the back of good automation
- We _do_ want a design bottleneck of some sort
- Discussions need to shift left (not design by committee on a PR)
- How can we minimise the amount of deviation we find by the time something gets into a PR?
- This is automated minimisation
- In order to achieve this we need to codify as much as possible in the automation
- Part of this is probably a cultural shift to trust the automation more
- Which in turn requires feedback on "is this working?" e.g. escaped defect counts
- This _might_ be the navigation team we've discussed with Mike Gallagher
- Adding some test mutation could help
- markdown based standards with a decent LLM could help here on PRs
- Who owns the overall user experience of the App and how it comes together? A feature may appear as a sub-section of a page
- We've talked about platform teams... Does a platform team end up owning the homescreen?

**Sticky notes — blue (outlined):**
- Is there value in the tenant being trusted to _properly own_ their own subsections?

**Label:** Existing Challenges

**Sticky notes — orange:**
- Pull requests get bottlenecked
- Too many levels of review (also outside squad)
- Branching strategy is overly complex
- Grabbing attention for review is always impossible when someone is outside your squad
- Automate codeownership / PR notifications / pester lists
- The existing pain also results in PRs that are too large

---

### Value Proposition *(The "Subsidy")*

1. What is the "Minimum Viable Platform" we provide that makes a tenant want to work with us?
2. Can a tenant team use their own analytics tool instead of the Platform standard?

**Sticky notes — yellow:**
- Everything currently ends up in Splunk and Databricks
- Actual analytics ends up in Databricks
- Observability is Splunk
- If you do use your own tooling stack, you will run afoul of
- Answer here must be "no"?
- IG / GDPR (think new subprocessors)
- Info Sec (new attack surface)
- We also have Adobe Analytics & Target
- Assume that access to the number is table stakes for all tenants
- And this is broad access... Not just their part of the service?
- Who is responsible for the performance of the tenant's elements?
- Give a performance budget / SLO for elements in the service
- Where will tenant backend services live? We assume in existing infra
- This means a secure link from an adapter / BFF to the NHSE services

**Sticky notes — green:**
- New aggregation and visibility might be viable?

**Sticky notes — orange:**
- Worth experimenting with a "you own your entire backend service" vs "all requests go through an app service BFF"

---

### Team Size Reduction *(Team Evolution)*

Ultimately, we think an outcome from this model would be that the size of the NHS App team could be reduced (this could be years away though), as we shift from "feature builders" to "enablers":

1. When a tenant team gets stuck on our playbook, who helps them

**Sticky notes — yellow:**
- Someone needs to own
- Design system & components
- Potentially infra
- The pipelines and definitions of build / testing
- Code standards and enforcement
- Prioritisation process... Who gets to deploy / release first?
- This playbook / process... Reflect on it and improve it
- BAU security patches
- Pen testing / security posture
- Service management wrapper
- Clinical assurance
- Performance testing
- How would it manifest if this dissemination was going wrong?
- Staging / preview environments
- External supplier integrations?
- Core capabilities (feature flagging, analytics, observability etc...)
- Definition of _some_ NFRs
- Integrations with "internal suppliers" e.g. Login

**Sticky note — yellow (outside right):**
- Needs to be quite close collaboration on the SBoM

---

### The "Noise" Floor *(Communication & Discoverability)*

1. When the Platform team updates a core component or API, how do 90 teams migrate?
2. How do teams know what capabilities already exist?

**Sticky notes — yellow (migration diagram):**
- Identification of type of change... and impacts
- Clear timelines needed to adhere to
- Assume supported version range in
- Really clear upfront communication of deprecation
- Clear opinionated versioning strategy
- How do we then manage opinionated change?
- SemVer?
- APIs
- Identify "stone ripple" changes early
- E.g. IA BDD

**Sticky notes — blue (outlined):**
- iOS / Android / Vue / others

**Label:** Discoverability

**Sticky notes — Discoverability cluster:**
- Design system playground *(platformy)*
- API discovery and experimentation *(everyone)*
- Reference screen / implementation / cookbooks *(platformy + everyone)*
- What is in my toolbox? *(platformy)*
- A cohesive approach to all of the above *(platformy)*

> **Key:** green icon = platformy / orange icon = everyone

**Sticky note — yellow (outside left):**
- Needs to be quite close collaboration on the SBoM

---

### Release Choreography *(The Sync Problem)*

1. Who has the authority to toggle a feature off in production?
2. Can a tenant team roll out their feature to only 1% of the 13M MAUs?

**Sticky notes — yellow:**
- We are currently constrained to rollout by ODS code
- Using Service Journey Rules (SJRs)
- This is a content entitlement function rather than strict feature flag
- SJRs need to be visible to all who might be interested in seeing it (all the tenants)
- I.e. what is on and what's off (and for who)
- If we needed to turn something off quickly
- Wielding of the feature flags could be beneficial (kill switch)
- Would also be beneficial to toggle per external system (not just a feature)
- This is all about maintaining quality of service
- Within a capability, what aspects are available at any one point in time?
- Separation of deployment from release
- Especially in relation to clinical culture & risk management
- Current feature flagging approach is pretty immature in terms of rollout
- What is the design / gatekeeping / coordination role look like?
- How federated can that be?
- What should be centralised vs decentralised?
- Are we basically talking about a co-operative?
- Maybe this pulls us back into the world of layered architecture (if we're not careful)
- Maybe the more we can give teams _full_ ownership of their part of the journey, the easier this becomes
- Could be helpful to think about the NHS App service as a platform primarily
- What do we actually need to automate and improve?
- What are the highest frequency touchpoints?
- Clinical assurance is a key example of this

**Sticky note — yellow (outside right):**
- Plus IG and infosec

---

### Commercials *(Funding & Chargeback)*

1. How is the Platform team funded once they stop building features for others?

**Sticky notes — yellow:**
- A well funded platform team that acts as an enabler would be best
- Having teams own their own infrastructure would help keep the funding simple
- Shutdown of an area then results in shutdown of products or shift in ownership
- Loaning out of centralised and specialised capacity feels like an anti-pattern
- Doesn't set areas up for long term success *(arrow pointing to "Loaning out of centralised...")*
- Invest in NHS App Service being a platform team
- How do you migrate their current feature ownership?
- Think of the App as PaaS
- If you could push a container into the PaaS... You might own the runtime but not the actual infra
- How do you maintain security without constraining?
- istio ftw :D

**Sticky notes — orange:**
- The worst way we could tackle this is if the commercials force us to deliver in the wrong way
- E.g fixed scope fixed cost

---

### Epilogue

1. Is each decision reversible?
2. Does each decision **increase** or **decrease** the cognitive load of a tenant team?
3. Does each decision **protect** or **endanger** the 13M MAU user experience?
4. Can this decision be **automated**? (If not, why?)

---

### Additional notes (Duncan Brown — outside main grid)

- Would be good to move towards some form of spike...
- External team collab
- Internal dog fooding

---

---

## Andrew Blundell

### Foreword

We are assuming a Landlord and Tenant model for the NHS App with a defined **Social Contract**.

- **The Landlord (NHS App teams):** You own the "common areas"—the navigation, the authentication, the shell, the design system, and the "plumbing."
- **The Tenants (NHSE Service Owners):** They own the "shops" inside the shopping centre.

Therefore it is natural that the NHS App evolves into an enabling platform rather than a feature factory.

This isn't a cast in stone decision, it's just a starting point to make it easier to reason about the challenge(s).

---

### Social Contract *(Ownership & Liability)*

1. When a feature causes a critical failure, what is the automated response?
2. A team builds a feature, ships it, and then their project funding ends. The feature starts to rot. Who owns the "Delete" button?

**Sticky notes — yellow:**
- Any product without a team maintaining it
- Put a team in place or kill the product
- Have already made efforts to kill of the plague of dead systems
- The NHS App team cannot become a dumping ground for unmaintained services
- Becomes about mature portfolio management as an organisation
- Authoritative data on who owns what and where it lives is already challenging
- Need to be able to attribute all technical artefacts back to the owners... This also needs to ladder all the way up to portfolio

---

### Technical Guardrails *(Automation over Governance)*

1. What defines a "Safe" change that requires zero human intervention from the Core team?
2. How do teams across NHSE ship updates to 13 million MAUs?

**Sticky notes — yellow:**
- Automated tests up the wazoo (Dave's words)
- Fast tests that prove that my change in particular is safe, in the context of the whole
- Continuous Deployment of native as far as staging
- Deploy into production is a synonym for release with native
- Is many many production app versions that bad?
- Flag on and off combinatorial testing
- Tracking of tested combinations (with audit trail)

---

### Value Proposition *(The "Subsidy")*

1. What is the "Minimum Viable Platform" we provide that makes a tenant want to work with us?
2. Can a tenant team use their own analytics tool instead of the Platform standard?

**Sticky notes — yellow:**
- Can we separate to the level that?
- Tenants source code can live in their own repositories
- They have micro-frontend code that lives separately
- Maybe native code bundles that get wrapped in??
- Reminds me of OSGI 😀
- Assume we are not moving any services or processing inside the App service
- Assume that app front end will connect with my services and my infrastructure
- Processes by which my work will be accepted or not
- They can work in isolation and not need to test _everything_ for every change
- A convention for feature toggling
- App team needs to protect the overall service
- Logging and alerting frameworks
- Observability as a whole
- Assume the app will have some scaffolding for
- Service management above this
- Standardised client side logging (e.g. Sentry or mobile equivalent)
- Design system & component library
- Some automated regression testing for integration
- End user monitoring
- Small ecosystem of pipelines

**Sticky notes — pink/salmon:**
- Not
- Compute
- Storage
- Repositories
- Performance
- Functional

---

### Team Size Reduction *(Team Evolution)*

Ultimately, we think an outcome from this model would be that the size of the NHS App team could be reduced (this could be years away though), as we shift from "feature builders" to "enablers":

1. When a tenant team gets stuck on our playbook, who helps them

*[Individual sticky note responses — illegible at available resolution]*

---

### The "Noise" Floor *(Communication & Discoverability)*

1. When the Platform team updates a core component or API, how do 90 teams migrate?
2. How do teams know what capabilities already exist?

**Sticky notes — yellow:**
- It is viable to have different looking interfaces across the same overall service?
- Imagine a design system change rolling across multiple micro-frontends
- Part of the contract could be
- You need to deploy every day (or commit)
- Whenever you deploy it needs to use -latest on curated internal dependencies. e.g. design system
- Proper feature flagging is essential (not just SJRs)

**Green outlined boxes (with diagram arrow):**
- We need clear lines of production and consumption between services
- Provider and customer *(arrow pointing up to the above)*

---

### Release Choreography *(The Sync Problem)*

1. Who has the authority to toggle a feature off in production?
2. Can a tenant team roll out their feature to only 1% of the 13M MAUs?

**Sticky notes — yellow (with diagram arrows):**
- Who can release? *(arrow up from "Tenant")*
- Who can switch off? *(arrow up from "Landlord or tenant")*
- Need a clear contract of understanding between landlord and tenant
- If you do not make good, your service will be removed after X period
- Tenant
- Landlord or tenant
- Native plus web skills makes this interesting
- Especially if we just want to do a small scale pilot
- What if the existing service is "good enough" and not part of the app?
- Can we still surface it

---

### Commercials *(Funding & Chargeback)*

1. How is the Platform team funded so they also building features for others?

*[Individual sticky note responses — illegible at available resolution]*

---

### Epilogue

1. Is each decision reversible?
2. Does each decision **increase** or **decrease** the cognitive load of a tenant team?
3. Does each decision **protect** or **endanger** the 13M MAU user experience?
4. Can this decision be **automated**? (If not, why?)

---

### Additional notes (Andrew Blundell — outside main grid)

- How do make it work quickly when it going well?
- What do you do when it goes wrong?
