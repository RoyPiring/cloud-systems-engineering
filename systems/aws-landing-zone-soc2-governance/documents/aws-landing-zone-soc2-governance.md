<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Architect an AWS Landing Zone from Scratch

**Project Link:** [View Project](https://learn.nextwork.org/projects/2577e51c-ef17-4b61-8d4f-ae269f397cd2)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2577e51c-ef17-4b61-8d4f-ae269f397cd2_hqq9wjmz)

## Architecting NovaBurst's Governance Foundation

### The mission: SOC 2 compliance for a post-IPO gaming company

In this project, I built a production-style AWS landing zone designed to support a rapidly growing gaming company operating under SOC 2 compliance requirements. The objective was to establish a scalable multi-account governance model that could standardize security controls, automate account onboarding, and reduce operational drift as new business units and acquisitions were integrated into the environment.

The architecture combined AWS Organizations, Terraform, Service Control Policies, StackSets, IAM Identity Center, and Service Catalog into a centralized governance framework. Instead of treating cloud accounts as isolated deployments, the environment was designed as a governed operating platform where security, monitoring, and compliance controls inherit automatically through organizational structure.

### Setting up the environment

I configured Service Control Policies to establish preventive governance controls across the AWS Organization before provisioning additional workloads.

SCPs operate above IAM and define the maximum permission boundary allowed within member accounts. This prevents local administrators from bypassing organizational controls and ensures governance policies remain enforceable across every environment.

The overall design separated centralized governance from local operational ownership. Engineering teams retained the ability to operate workloads independently while core compliance and security requirements remained enforced at the organizational level.

## Environment Setup and Terraform Initialization

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2577e51c-ef17-4b61-8d4f-ae269f397cd2_f9nrycdq)

### Required IAM permissions for the management account

The management account required elevated permissions across Organizations, IAM, and CloudFormation services to bootstrap the landing zone successfully.

OrganizationsFullAccess enabled OU creation, account management, SCP attachment workflows, and organization-level governance orchestration. IAMFullAccess was required to configure permission sets, federated identity mappings, and cross-account access roles. AWSCloudFormationFullAccess enabled StackSets deployment across governed accounts.

These permissions established the control-plane foundation Terraform relied on to provision and manage organization-wide infrastructure safely and consistently.

## Designing the Multi-Account OU Topology

### Reasoning through the 4-OU hierarchy

I designed a four-OU organizational hierarchy to separate workloads by governance level, operational purpose, and risk exposure.

The structure isolated production, sandbox, security, and shared-services environments into distinct governance boundaries with dedicated SCP inheritance paths. Architecture Decision Records and Mermaid diagrams documented the reasoning behind the hierarchy and clarified how governance controls propagate throughout the organization.

This design allows the organization to scale without manually reapplying security controls for every new account.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2577e51c-ef17-4b61-8d4f-ae269f397cd2_ejn3rcd5)

### SCP inheritance and automatic governance for new accounts

SCP inheritance was intentionally centered around OU placement rather than manual account-level policy attachment.

As soon as a new account is created and placed into an OU, it automatically inherits every SCP attached along the organizational path, including both root-level and OU-level restrictions. This ensures governance controls are applied immediately during onboarding rather than retrofitted later.

The result is a landing zone where organizational structure directly determines baseline security posture and operational constraints.

## Bootstrapping the AWS Organization with Terraform

### Scaffolding the Org module with Cursor Composer

I used Cursor Composer to scaffold the Terraform organization module responsible for provisioning OUs, member accounts, SCP attachments, and foundational governance resources.

The Terraform code was reviewed against the Architecture Decision Records before deployment to ensure the implementation aligned with the intended governance model. Once validated, the configuration was applied to provision the AWS Organization directly from infrastructure-as-code.

This created a reproducible and auditable governance baseline where organizational infrastructure can be rebuilt or extended through version-controlled workflows instead of manual console operations.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2577e51c-ef17-4b61-8d4f-ae269f397cd2_cip5qbwt)

### Understanding feature_set, policy types, and FullAWSAccess

The organization was configured using feature_set = "ALL" to enable advanced AWS Organizations capabilities beyond consolidated billing.

Enabling SERVICE_CONTROL_POLICY as a policy type allowed SCP evaluation, inheritance, and enforcement across roots, OUs, and member accounts. FullAWSAccess remained attached at the root as the baseline allow-list SCP until explicit deny-based governance controls were layered on top.

This reflects an allow-by-default governance model where compatibility is preserved while targeted restrictions are introduced through explicit deny policies.

## Building the SCP Library to Close Every Auditor Finding

### Six preventive controls mapped to six SOC 2 findings

I generated six SCP policy files aligned directly to six simulated SOC 2 audit findings and deployed them across the appropriate organizational units.

The SCP library enforced preventive controls around restricted AWS regions, governance escape prevention, unauthorized service usage, and operational boundary enforcement. Each policy was red-teamed through Cursor agents simulating auditor review before deployment.

This transformed the governance model from broad organizational policy into measurable preventive enforcement tied directly to audit evidence.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2577e51c-ef17-4b61-8d4f-ae269f397cd2_1s3bw0wh)

### Why the deny-list strategy depends on FullAWSAccess at Root

SCPs do not grant permissions by themselves. They only limit actions that IAM policies would otherwise allow inside member accounts.

FullAWSAccess remains attached at the root so standard IAM workflows continue functioning while targeted deny-based SCPs selectively restrict high-risk operations. Removing the baseline allow-list SCP without replacing it carefully can unintentionally create an implicit-deny posture across the organization and introduce widespread operational outages.

The deny-list strategy preserves compatibility while incrementally tightening governance boundaries through explicit restrictions.

## Deploying the StackSets Security Baseline

### Auto-deploying CloudTrail, Config, and GuardDuty to every account

I activated trusted access for CloudFormation StackSets and generated baseline templates for CloudTrail, AWS Config, and GuardDuty.

The StackSets deployment was configured with auto-deployment so newly created member accounts inherit monitoring and detection baselines automatically without manual intervention. This ensured centralized audit logging, configuration tracking, and threat detection remained consistent as the organization scaled.

This phase moved the landing zone from policy-only governance into active operational visibility and continuous security monitoring.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2577e51c-ef17-4b61-8d4f-ae269f397cd2_vepbxamy)

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2577e51c-ef17-4b61-8d4f-ae269f397cd2_m1qthjlz)

### The management account blind spot and its architectural consequence

The management account required additional hardening because service-managed StackSets do not always treat it the same way as member accounts.

OU-based StackSet targeting works well for governed member accounts, but the payer and management account can become a privileged exception if they are not explicitly included through separate deployment paths and governance controls.

The architectural consequence is a residual high-privilege island where the organization’s most sensitive account can drift away from the baseline posture unless additional SCPs, audit-role validation, and dedicated management-account hardening are implemented directly.

## Centralizing Identity Governance with IAM Identity Center

### Permission sets and cross-account audit role design

I enabled IAM Identity Center and created permission sets aligned to NovaBurst’s operational access tiers.

Cross-account audit roles were deployed to allow the Security account to perform incident response and governance validation across member accounts without requiring local administrative access inside each environment. The implementation also validated how permission sets become temporary IAM roles with session expiration, reinforcing least-privilege access principles.

This centralized identity model reduced IAM sprawl while improving operational accountability and auditability.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2577e51c-ef17-4b61-8d4f-ae269f397cd2_cmxv5iip)

### Why the audit role trusts only the Security OU account

The audit role trusts only the Security OU account to minimize privilege exposure and restrict incident-response access to authorized security personnel.

Allowing the management account root or all organizational accounts to assume the role would dramatically increase blast radius because any IAM principal inside those accounts could potentially satisfy the trust relationship.

The governance objective was to preserve a clean separation between operational administration and security oversight. The Security account functions as the controlled break-glass access boundary for audit and incident-response workflows.

## Proving Compliance: SCP Smoke Tests and Operational Artifacts

### Generating evidence for the SOC 2 auditors

I executed SCP smoke tests to validate GDPR region restrictions, governance escape prevention, and cross-account audit-role assumptions from the Security account.

Additional operational artifacts included SCP precedence diagrams, StackSets drift-validation scripts, and governance runbooks documenting escalation paths and control ownership.

This created an auditable evidence trail showing that governance controls were not only deployed, but actively validated under operational conditions.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2577e51c-ef17-4b61-8d4f-ae269f397cd2_79494qq3)

### SCP evaluation overrides AdministratorAccess

Authorization inside AWS Organizations is not controlled by IAM alone. Effective permissions are evaluated as the intersection of IAM policies, SCPs, resource policies, and session boundaries.

SCPs can explicitly deny APIs even when AdministratorAccess exists locally because SCPs function as organization-wide guardrails that local IAM permissions cannot override.

The important operational takeaway is that IAM expands permissions inside an account, but SCPs define the organization’s non-negotiable governance boundaries.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2577e51c-ef17-4b61-8d4f-ae269f397cd2_n3m7vgh1)

## Delivering the Principal Architect's Engagement Report

### Generating the executive-ready HTML report with Cursor

I generated a comprehensive HTML engagement report designed for executive leadership, auditors, and governance stakeholders.

The report consolidated architecture diagrams, KPI metrics, compliance evidence, implementation timelines, risk registers, and governance decisions into a single browser-rendered artifact using Mermaid visualizations and structured reporting layouts.

This transformed the landing-zone implementation into an executive-ready governance deliverable rather than a purely technical deployment exercise.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2577e51c-ef17-4b61-8d4f-ae269f397cd2_hqq9wjmz)

### Why the Findings Remediation Matrix is critical for auditors

The Findings Remediation Matrix mapped each simulated SOC 2 finding directly to its associated AWS control, implementation state, and supporting artifact evidence.

Instead of relying on high-level narrative explanations, the matrix established a traceable audit path from identified risk to validated mitigation.

For auditors, this becomes the authoritative evidence layer proving where governance controls exist and how they are enforced operationally.

## Secret Mission: Scaling Governance for the Next Acquisition

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2577e51c-ef17-4b61-8d4f-ae269f397cd2_auuyiilj)

### Account Vending Machine: one-click governance inheritance

I extended the landing zone by building an Account Vending Machine using AWS Organizations, Service Catalog, StackSets, and Terraform automation.

The workflow automatically creates member accounts, assigns them to the correct OU, applies standardized Studio and CostCenter tags, and inherits all applicable governance controls immediately during provisioning. StackSets then deploy CloudTrail, Config, and GuardDuty baselines automatically without requiring additional onboarding work.

This converted account onboarding from a manual operational process into a deterministic governance workflow.

## Reflections and Key Takeaways

### Tools and concepts mastered

This project combined Terraform, AWS Organizations, SCPs, StackSets, IAM Identity Center, Service Catalog, AWS CLI, and Cursor IDE into a production-style governance architecture.

The concepts reinforced throughout the project included multi-account governance, identity federation, SCP enforcement, automated security baselines, account vending workflows, compliance traceability, and organization-scale operational governance.

### Time investment and challenges

This project took approximately two hours to complete. The most challenging part was designing the Account Vending Machine workflows and aligning Service Catalog provisioning logic with Terraform orchestration while preserving governance inheritance and correct OU placement.

The implementation required careful coordination between organizational hierarchy, SCP inheritance, StackSets deployment scope, and tagging enforcement.

The biggest takeaway was understanding how governance can operate as inherited infrastructure rather than manual administrative enforcement.

The next area I want to improve is cloud-cost governance through automated lifecycle enforcement, policy-driven optimization, and organization-wide operational analytics.

---

*Built with [NextWork](https://learn.nextwork.org) - [View this project](https://learn.nextwork.org/projects/2577e51c-ef17-4b61-8d4f-ae269f397cd2)*
