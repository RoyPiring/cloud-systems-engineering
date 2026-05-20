<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Get Promoted: Build AWS DR Worth $1M

**Project Link:** [View Project](https://learn.nextwork.org/projects/dde42cdf-e829-4a4a-a8af-ec2b7bc0b960)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/dde42cdf-e829-4a4a-a8af-ec2b7bc0b960_m89d8h2n)

## The $4.7M Problem That Demanded a Solution

### Project mission and business context

In this project, I designed a multi-region AWS disaster recovery architecture to protect MegaCart from revenue-impacting outages and reduce the business risk exposed during the previous Black Friday incident, which resulted in approximately $4.7M in losses.

The objective extended beyond technical recovery. The architecture needed to align recovery strategy with revenue impact, customer experience, and executive expectations while maintaining operational and financial efficiency. The final design combined tiered recovery models, workload prioritization, gameday validation, and TOGAF-style executive artifacts into a promotion-level architecture package.

## Building the War Room: Tools and Environment

### Step goals and setup strategy

I verified Terraform 1.15.x, initialized LocalStack through Docker for local AWS emulation, and configured Cursor IDE with five parallel Composer agents to accelerate delivery.

The environment intentionally avoided live AWS resources by using LocalStack to emulate AWS APIs locally. This enabled rapid iteration, repeated validation, and full infrastructure teardown without incurring cloud cost while still preserving infrastructure-as-code workflows.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/dde42cdf-e829-4a4a-a8af-ec2b7bc0b960_i0xppyas)

### Tools installed and their roles

Terraform through tflocal served as the infrastructure engine and deployed the complete DR environment against LocalStack rather than AWS.

LocalStack provided local AWS emulation through the megacart-lab environment, allowing the architecture to be built, destroyed, and tested entirely on localhost.

Locust generated production-style load and chaos traffic during gameday execution. This allowed failover testing to occur under active workload conditions rather than synthetic idle states.

## Architecting the Primary Region: Principal-Level Design Decisions

### Primary stack architecture goals

Agent-A was directed to build the primary three-tier AWS stack using Terraform and deploy it into LocalStack. The implementation included application, data, and networking layers together with ADR documentation mapped to SAP-C02 architecture domains.

The objective was not only deploying infrastructure but documenting architectural intent, tradeoffs, and recovery reasoning so the design remained explainable to engineering leadership and review committees.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/dde42cdf-e829-4a4a-a8af-ec2b7bc0b960_pb0an26y)

### ADR design decision with SAP-C02 mapping

Decision four evaluated Aurora MySQL against standard RDS MySQL Multi-AZ.

Aurora was selected because failover completes in approximately thirty seconds compared to sixty to one hundred twenty seconds for standard RDS Multi-AZ configurations. At peak transaction periods, those additional recovery minutes directly translate into revenue exposure.

The decision aligned to SAP-C02 domain D2.2 covering resilient database architectures, recovery patterns, and high-availability design.

## Designing DR Tiers and Building the Business Case

### Workload-to-tier assignment strategy

Agent-B generated four DR workspace modules and workloads were mapped into recovery tiers according to revenue impact, customer experience sensitivity, and acceptable downtime.

Checkout, catalog, user profiles, and analytics each received different recovery treatments based on operational value rather than applying one universal architecture.

This tiered model optimized cost by matching protection levels to business criticality instead of over-engineering every workload.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/dde42cdf-e829-4a4a-a8af-ec2b7bc0b960_8f1w2uti)

### Justifying active-active for checkout: the $100K/min argument

Checkout was assigned active-active protection because peak transaction periods generate approximately $100K per minute.

A five-minute warm-standby recovery would create roughly $500K of direct exposure, making the lower-cost model financially unacceptable. Historical evidence reinforced this decision because the prior forty-seven-minute outage produced approximately $4.7M in losses.

The active-active design introduced approximately $51K annual cost but protected the highest-value transaction path. The investment effectively pays for itself within seconds of avoided outage time.

Catalog workloads remained warm-standby because temporary browsing degradation creates inconvenience but does not immediately stop revenue capture.

## Gameday: Proving Every DR Tier Works With Measured Data

### Chaos engineering and failover testing approach

The validation phase implemented Route 53 failover records, ARC routing-control simulations, Locust load generation, and chaos injection scripts.

The gameday process intentionally forced failures while active traffic remained present so recovery behavior could be measured under realistic conditions.

Measured RTO outputs were exported as structured JSON artifacts to support later compliance and executive reporting workflows.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/dde42cdf-e829-4a4a-a8af-ec2b7bc0b960_h4d3lgjx)

### What the 23-second RTO reveals about the architecture

Checkout recovery completed in twenty-three seconds, significantly below the declared sixty-second RTO target.

The result confirmed that active-active behaved correctly because recovery depended primarily on Route 53 weighting adjustments and healthy secondary services rather than cold starts, database promotion, or infrastructure initialization.

The measured gap between actual recovery and target recovery created operational headroom for DNS propagation delays, health convergence, and transient failures while remaining inside business objectives.

For a workload generating $100K per minute, measured sub-minute recovery validated the architectural investment.

## The Executive Deliverables Package: Promotion Evidence

### TOGAF deliverables production goals

The final package included ADR documentation, KPI dashboards, Mermaid visualizations, gameday compliance reporting, cost analysis artifacts, and before/after architecture diagrams.

The objective was producing decision-ready evidence rather than technical implementation notes. Every artifact translated architecture into language consumable by leadership, governance boards, and promotion committees.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/dde42cdf-e829-4a4a-a8af-ec2b7bc0b960_a5fgcqjd)

### What each TOGAF deliverable proves to the promotion committee

The deliverable package contained five TOGAF-aligned artifacts under megacart-dr/docs.

ADR.md documented recovery strategy and SAP-C02 decisions.

BeforeAfterDiagram.md illustrated the transition from single-region failure risk toward multi-region tiered resilience.

CostAvoidanceReport.md and KPIDashboard.md demonstrated financial outcomes including $74K spend, $206K annual savings, and sixty-three times ROI.

GamedayComplianceReport.md provided operational proof through measured failover results.

Together these artifacts demonstrated architecture maturity, financial alignment, and validated operational execution.

## Secret Mission: Cost-Per-Nine Analysis and Ransomware Protection

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/dde42cdf-e829-4a4a-a8af-ec2b7bc0b960_2xnbnxyb)

### Ransomware drill results and the 63x ROI justification

The ransomware scenario simulated administrative compromise and attempted deletion of recovery points.

Vault Lock together with deny-delete policies blocked destructive actions even under elevated permissions. Recovery assets remained available despite the simulated compromise.

This created defense-in-depth protection aligned with SAP-C02 D3.4 recovery and resilience principles.

The cost model followed workload value rather than blanket availability targets. Checkout received 99.99% protection while lower-value workloads operated at lower availability tiers.

This reduced projected cost from approximately $280K to $74K while preserving recovery objectives and producing sixty-three times return on investment.

Four out of four gameday scenarios passed with measured failover in seconds.

## Reflections: Skills, Concepts, and the Architecture Mindset

### Key tools and concepts mastered

This project combined Terraform, LocalStack, Cursor IDE, Route 53 failover controls, Locust testing, and TOGAF-style documentation into a unified DR engineering workflow.

The concepts reinforced throughout the build included multi-region resilience, DR tiering, chaos testing, RTO/RPO planning, FinOps optimization, cost-per-nine analysis, and executive architecture communication.

### Time investment and challenges

This project required approximately two hours to complete.

The most difficult part was translating infrastructure decisions into executive narratives. Technical implementation alone was insufficient because the architecture also needed to justify spend, demonstrate risk reduction, and communicate business value clearly.

Balancing technical depth with leadership messaging became the largest challenge.

### Learning outcomes and next steps

I completed this project to strengthen multi-region disaster recovery design skills and understand how architecture decisions connect directly to business continuity outcomes.

The next capability I want to deepen is producing larger TOGAF-aligned executive packages and building promotion-level architecture portfolios grounded in measurable business impact.

---

*Built with [NextWork](https://learn.nextwork.org) - [View this project](https://learn.nextwork.org/projects/dde42cdf-e829-4a4a-a8af-ec2b7bc0b960)*
