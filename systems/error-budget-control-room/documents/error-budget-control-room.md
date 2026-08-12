<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Error Budget Control Room

**Project Link:** [View Project](https://nextwork.ai/projects/b172a54a-456f-42f9-8c9a-b1a8d6637104)

**Author:** Roy Piring Jr: Sr. Cloud Engineer | Architect  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/b172a54a-456f-42f9-8c9a-b1a8d6637104_zkltzj2g)

## Building an Error Budget Control Room from First Principles

### The problem this build solves

I built the control room to derive and validate error-budget burn-rate thresholds instead of accepting generic alert values without understanding them.

The thresholds matter because bad alert design creates two opposite risks. Alerts that fire too often create noise and fatigue, while alerts that react too slowly can let meaningful SLO consumption continue without intervention.

Deriving the values manually gave me a way to connect the alert configuration back to the underlying SRE math. The result was a system where the on-call behavior could be traced to explicit assumptions about budget consumption, SLO period, and alert windows.

### Setting up the repository and stack

In this step, I set up the GitHub repository and containerized observability environment before implementing the burn-rate rules.

The stack gave me a repeatable place to run Prometheus, Grafana, and the supporting services while GitHub tracked the delivery work.

This mattered because the later alert proof depended on a stable environment. I needed to know that failures came from alert logic or injected traffic, not from an inconsistent local setup.

## Proving the Stack is Green

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/b172a54a-456f-42f9-8c9a-b1a8d6637104_z5az8of2)

### Project board and issue tracking

I created six board issues covering setup, design, build, prove, deliver, and after-action work.

The setup issue, Error Budget Control Room, moved to Done after the environment was verified. The remaining five issues stayed in Todo until their corresponding work began.

That board state established the delivery sequence before deeper SRE design started. It also made the later proof and documentation work visible as separate parts of the build instead of folding everything into one task.

## Designing with AI: Arguing the Denominator

### Design-before-build approach

In this step, I drafted the Service Level Indicators and Architecture Decision Records before creating the production alert rules.

The goal was to establish a mathematically grounded error-budget model first. The SLI definitions had to describe signals that could actually be measured from the available metrics, while the ADRs recorded the reasoning behind the choices.

This mattered because burn-rate math is only as valid as its denominator. If the SLI cannot be measured correctly, precise alert thresholds still produce the wrong operational answer.

### Rejecting an unmeasurable SLI candidate

I rejected Candidate C, which combined “not 5xx and fast enough” into one SLI.

HTTP status and latency existed in separate metrics with no shared request identifier. That meant the available series could not prove a true per-request logical AND between successful status and acceptable latency.

The candidate sounded useful but could not be measured honestly from the available telemetry. Rejecting it kept the SLI contract aligned with what the system could actually observe.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/b172a54a-456f-42f9-8c9a-b1a8d6637104_wm0bmsgv)

## Generating and Verifying Burn-Rate Rules

### From OpenSLO source of truth to Prometheus rules

In this step, I wrote the OpenSLO specification that served as the source for the service-level objectives and burn-rate configuration.

Sloth then generated the Prometheus recording and alerting rules from that specification. This kept the production rule files tied to a declared SLO definition instead of maintaining hand-edited Prometheus expressions separately.

The generated rules still required verification. Automation produced the configuration, but the threshold math had to agree with the values I derived independently.

### Hand-verifying the 14.4x and 6x multipliers against the Workbook

The two production burn-rate multipliers were 14.4x for fast burn and 6x for slow burn.

I derived them from the Workbook relationship r_th = (p × T) / w, where the threshold equals the fraction of budget consumed multiplied by the SLO period, divided by the alert window.

For the fast burn, consuming 2% of a 30-day error budget in 1 hour produced 14.4. For the slow burn, consuming 5% in 6 hours produced 6. Those calculations gave me an independent check against the generated alert configuration.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/b172a54a-456f-42f9-8c9a-b1a8d6637104_hcnhb9z3)

## Proving Firing Order: The Fast Burn Pages First

### Injecting a 10% error rate and observing alert sequence

In this step, I injected a 10% error rate into the service and watched the Prometheus alert state change over time.

The purpose was to prove that the fast-burn alert fired before the slow-burn alert. Both rules could see the same unhealthy traffic, but their time windows were intentionally different.

The observed firing order connected the configuration back to the hand-derived design. The fast window elapsed first, so the corresponding alert reached its firing condition before the slower window.

## Deriving and Delivering the Full Artifact Set

### Stakeholder readout, teach-back, and derivation docs

In this step, I documented the burn-rate derivation and prepared the stakeholder readout and teach-back material.

The derivation showed where the alert multipliers came from instead of treating 14.4x and 6x as unexplained constants. The readout translated that math into the operational reason for having separate fast and slow burn alerts.

The teach-back forced me to explain both the formulas and the on-call behavior. That made the documentation part of the proof rather than a summary written after the implementation.

### What the lab preserves and what it does not

The compressed lab preserved the production rate thresholds of 14.4x and 6x, along with the same 12:1 ratio between the long and short windows.

Only the time windows changed. The production 1h/5m pair became 5m/25s, while 6h/30m became 30m/2m30s, allowing the firing sequence to be observed during a short lab session.

The 30-day error-budget period was not compressed. Because of that, the lab proved threshold behavior and firing order, but it did not preserve the real-world meaning of consuming a specific percentage of a 30-day budget inside the shortened lab time.

## Scoring the AI: After-Action Review

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/b172a54a-456f-42f9-8c9a-b1a8d6637104_yeuy49tn)

### Honest meta-analysis of AI accuracy and review rigour

Score 1 recorded one rejection. The AI proposed an SLI set that was mostly usable but included Candidate C, which could not be measured from the available metrics.

Score 2 recorded 2 of 6 answers as wrong. The window calculations were correct, but the alert multipliers were missing until I derived them manually.

Together, those scores showed that the AI was useful for accelerating the design but could not be treated as the authority. The non-zero review scores mattered because the review step changed the final implementation instead of simply approving the generated output.

## Reflections and Key Takeaways

### Tools and concepts mastered

The key tools I used included Docker for local orchestration, Prometheus for metrics and alert evaluation, Grafana for visualization, k6 for controlled load and error injection, and GitHub for version control and issue tracking.

The main concepts I learned included deriving burn-rate thresholds from the Google SRE Workbook formulas, separating measurable SLIs from attractive but unsupported ones, using controlled error injection to test alert behavior, and proving alert firing order instead of assuming the configuration worked.

I also learned the value of recording the design through ADRs and then scoring AI-assisted output against the final engineering result. The combination made the system easier to defend because both the math and the review path were visible.

### Time and challenge reflection

This build took me approximately 75 minutes.

The hardest part was making sure the hand-derived burn-rate thresholds aligned with the Google SRE Workbook formulas while also testing their firing order inside compressed lab windows.

The challenge was keeping those two ideas separate. The multipliers stayed tied to the production SLO math, while the shortened windows existed only to make the behavior observable during the session. Mixing those concepts would have produced a convincing lab with the wrong interpretation.

### Skills gained and next steps

I completed this build to learn how to derive error-budget burn-rate alerts from first principles instead of copying threshold values from documentation.

The work connected SLI design, mathematical derivation, generated Prometheus rules, controlled failure injection, and live alert behavior into one evidence path.

Next, I want to apply the same observability patterns to larger distributed microservice architectures, where multiple services, dependencies, and SLOs need coordinated alerting without creating unnecessary on-call noise.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/b172a54a-456f-42f9-8c9a-b1a8d6637104)*
