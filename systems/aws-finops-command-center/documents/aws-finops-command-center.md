<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# AWS FinOps Command Center

**Project Link:** [View Project](https://learn.nextwork.org/projects/91b508b8-c548-4372-a701-e10eb119e1cf)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/91b508b8-c548-4372-a701-e10eb119e1cf_iqjpp8z0)

## Building an Enterprise FinOps Command Center

### The mission: automated cost governance for a Fortune 10 company

In this project, I built an enterprise FinOps command center designed to improve cloud cost visibility, automate governance actions, and provide leadership-ready financial reporting for a Fortune 10 scale environment.

The architecture combined CUR 2.0 exports, Glue, Athena, Grafana, Cost Anomaly Detection, Lambda remediation workflows, Budget Actions, Organizations Tag Policies, and executive dashboards into a unified FinOps operating model. The objective was to move from reactive billing analysis toward automated cost intelligence and operational governance.

## Setting Up the FinOps Toolchain

### Environment goals and tooling overview

I verified AWS CLI, Terraform, and Docker Desktop installations before launching Grafana OSS with the Athena datasource plugin. The Terraform project structure was then scaffolded and the AWS provider initialized to support infrastructure deployment.

The environment established the operational foundation for cost ingestion, dashboarding, governance automation, and remediation workflows while keeping infrastructure reproducible through infrastructure-as-code practices.

### Verifying AWS Organizations management account access

AWS Organizations access was validated successfully against the management account and confirmed the environment was operating with the ALL feature set and Service Control Policies enabled.

This validation mattered because management account access is required for organization-wide governance capabilities such as CUR configuration, Tag Policies, Cost Allocation Tags, and cross-account FinOps visibility.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/91b508b8-c548-4372-a701-e10eb119e1cf_8ht39a7c)

## Architecting with Pre-Artifacts: ADRs, Diagrams, and Governance Plans

### Principal-level documentation before writing a single line of infrastructure code

Before deployment, I created architecture decision records covering Grafana vs QuickSight, CUR 2.0 vs Cost Explorer API, and Tag Policies vs manual tagging approaches.

I also developed Mermaid architecture diagrams, requirements documentation, implementation plans, risk registers, and FinOps operating cadence templates. This established governance and operational expectations before infrastructure provisioning began.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/91b508b8-c548-4372-a701-e10eb119e1cf_i5avjjhc)

### Tag Policies vs. Cost Allocation Tags: why both are required

Tag Policies enforced required keys such as CostCenter, Environment, and Owner during resource creation through AWS Organizations governance.

Cost Allocation Tags performed the separate role of exposing those values inside CUR and Cost Explorer for chargeback reporting. One enforces compliance while the other enables financial visibility. Neither replaces the other, and enterprise chargeback models require both simultaneously.

## Deploying the CUR 2.0 → Glue → Athena Data Pipeline

### Wiring the cost intelligence data pipeline end-to-end

I deployed the CUR 2.0 export pipeline including S3 delivery, billing report configuration, Glue catalog resources, crawlers, Athena workgroups, and chargeback analysis views.

Cost Allocation Tags were then activated so cost ownership data flowed consistently from AWS resources into CUR datasets and executive reporting workflows.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/91b508b8-c548-4372-a701-e10eb119e1cf_atk52h58)

### CUR 2.0 region constraints and delivery timing

CUR 2.0 operates as a global billing service and only accepts report configuration from us-east-1 regardless of workload region placement.

Initial report delivery required roughly 24 hours before Parquet files became available in S3 for Glue crawling and Athena querying. This operational delay influenced dashboard development because reporting artifacts depended on data availability before validation could occur.

## Building the Grafana Executive Dashboard

### Translating raw billing data into CFO-ready visualizations

I configured the Grafana Athena datasource and created dashboards covering chargeback reporting, executive KPIs, and 12-month cost projections.

The dashboards translated raw CUR datasets into leadership-ready visualizations focused on cost ownership, trend analysis, spend forecasting, and organizational accountability.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/91b508b8-c548-4372-a701-e10eb119e1cf_cr69dp12)

### Template variables enabling dynamic cost exploration without query editing

The chargeback dashboard introduced template variables for cost center, linked account, and reporting windows.

Executives could filter spend dynamically through dashboard controls while Athena queries automatically re-scoped results using ${costcenter:sqlstring} and ${account:sqlstring}. This removed the need to edit SQL directly and allowed financial analysis to remain interactive.

## Automating Anomaly Detection and Remediation

### Deploying the full detection-to-action control loop

I deployed Cost Anomaly Detection monitors with SNS integration, built a Lambda remediator with cross-account reach, and integrated EventBridge workflows and Budget Actions as layered governance controls.

The architecture created a full detection-to-remediation pipeline where unusual spending automatically triggered response workflows instead of relying on manual review cycles.

### The dual remediation pattern: auto-stop for non-prod, manual approval for production

Environment tags acted as the operational safety boundary. Development, Test, and Staging workloads automatically qualified for remediation actions while production workloads required human approval.

This balanced cost control with operational safety by allowing fast automation where risk remained low while preserving review gates where downtime could affect customers.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/91b508b8-c548-4372-a701-e10eb119e1cf_2agu7fn1)

### Layered guardrails: Budget Actions as static backstop, Lambda as dynamic responder

Budget Actions operated as static governance by forecasting spend thresholds and automatically attaching IAM restrictions once limits were exceeded.

The Lambda remediator provided dynamic response by reacting to anomaly signals and stopping already-running non-production workloads. Together they covered both forecast overruns and real-time cost spikes.

## Validating the End-to-End Pipeline with a Spike Test

### Proving the system works under simulated real-world conditions

I launched a non-production t3.micro spike instance to simulate runaway spend and validated remediation workflows using synthetic anomaly events.

Grafana dashboards, remediation logs, and chargeback views were then reviewed to confirm the full pipeline functioned correctly from anomaly detection through visualization.

### Synthetic anomaly event triggers Lambda, stops spike instance, logs to DynamoDB

EventBridge rejected synthetic aws.ce events because only AWS Cost Anomaly Detection may publish that source.

Validation instead occurred by directly invoking the remediation Lambda using the same payload structure. The function identified the Development workload, executed STOP_NON_PROD_EC2, stopped the instance, wrote logs to DynamoDB, and surfaced the remediation as dashboard annotations inside Grafana.

## Delivering the CFO Executive Pitch Deck

### Assembling a Fortune 10-ready presentation from live dashboards and pre-artifacts

I assembled the executive pitch deck using live Grafana dashboards together with architecture artifacts, risk matrices, technical debt scorecards, and optimization roadmaps.

The final package translated engineering implementation into executive language focused on savings opportunities, governance maturity, operational risk reduction, and long-term optimization strategy.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/91b508b8-c548-4372-a701-e10eb119e1cf_cm8oj3sd)

### Well-Architected Cost Pillar alignment as a credibility multiplier

The roadmap aligned directly with AWS Well-Architected Cost Optimization principles rather than presenting optimization work as isolated engineering efforts.

Mapping quarterly initiatives against COST01 through COST11 created a clear maturity progression and gave leadership recognizable governance language tied to measurable business outcomes.

## Secret Mission: Carbon Footprint Overlay and Automated Weekly FinOps Cadence

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/91b508b8-c548-4372-a701-e10eb119e1cf_s5h5f5oz)

### Replacing manual reporting with an automated weekly KPI digest via SNS and EventBridge Scheduler

I automated weekly FinOps reporting using EventBridge Scheduler, Lambda, Athena queries, and SNS delivery.

The weekly digest included spend trends, tag coverage, Savings Plans utilization, rightsizing backlog, anomaly counts, and week-over-week movement while pulling data directly from the same Athena views powering executive dashboards.

Operational value of automated reporting cadence

Manual spreadsheet reporting creates drift because reporting quality depends on availability and timing.

The automated cadence established a consistent operating rhythm where FinOps and finance teams began every week from the same validated KPI baseline. This improved decision speed and reduced reporting inconsistency across stakeholders.

## Reflections and Key Takeaways

### Tools mastered and FinOps concepts internalized

This project combined Terraform, AWS CLI, Docker, Grafana, CUR 2.0, Athena, Glue, Cost Anomaly Detection, Lambda remediation workflows, Budget Actions, and Organizations governance into a unified FinOps platform.

The concepts reinforced included chargeback reporting, anomaly detection, cost governance, automated remediation, dashboard engineering, financial accountability, and operational cost optimization.

### Time investment and hardest challenges

This project required approximately three hours to complete.

The most difficult part involved troubleshooting Lambda execution timeouts while validating Athena query locations, Glue catalog configuration, and end-to-end integration behavior between reporting components.

### What's next on the learning journey

I completed this project to strengthen enterprise FinOps implementation skills and understand how automated governance can reduce operational cost risk.

The next area I want to deepen is advanced Terraform architecture patterns and large-scale infrastructure lifecycle management.

---

*Built with [NextWork](https://learn.nextwork.org) - [View this project](https://learn.nextwork.org/projects/91b508b8-c548-4372-a701-e10eb119e1cf)*
