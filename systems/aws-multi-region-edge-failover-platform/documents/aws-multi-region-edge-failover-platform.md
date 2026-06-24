<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# AWS Multi-Region Edge Failover Sprint

**Project Link:** [View Project](https://nextwork.ai/projects/ab1d8ae3-e094-4679-ae15-8a83896b34c3)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/ab1d8ae3-e094-4679-ae15-8a83896b34c3_rct5ekwp)

## Architecting a Crisis Response: The $0.61 Sprint

### The mission and its stakes

This project focused on building a resilient multi-region edge architecture capable of maintaining service availability during regional failures. The environment was prepared by validating Terraform 1.15.3 and AWS CLI v2, establishing the project structure, initializing Terraform state, and coordinating work across AI-assisted contributors through Cursor Composer. These foundational steps ensured that deployment activities could proceed with a consistent and repeatable infrastructure workflow.

The objective was not only to deploy resources, but to prove operational recovery, infrastructure portability, and controlled failover through infrastructure as code. Establishing a clean deployment baseline reduced configuration drift risk and enabled reliable validation throughout the sprint.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/ab1d8ae3-e094-4679-ae15-8a83896b34c3_9ejwv4a7)

### Multi-region provider configuration

The Terraform configuration used aliased AWS providers for both us-east-1 and eu-west-1, allowing resources across multiple regions to be managed from a single codebase and state file. This approach simplified deployment coordination while preserving regional separation for failover testing.

By explicitly assigning providers within modules, regional resources such as origins, DNS records, edge services, and acceleration components remained predictable and auditable. Managing both regions from one Terraform state raised operational visibility and reduced complexity during deployment, validation, and teardown activities.

## Building Two-Region Origins and Automated DNS Failover

### Lambda origins and Route 53 routing policies

The architecture deployed identical Lambda-based origins in us-east-1 and eu-west-1 using Function URLs as public endpoints. Each function returned regional identification data, enabling clear validation of traffic routing and failover behavior during testing.

Route 53 health checks were implemented alongside Failover, Geoproximity, Latency, Weighted, and Multivalue Answer routing policies. This provided hands-on exposure to multiple DNS traffic management patterns while demonstrating how health-based routing decisions influence application availability. The design highlighted how DNS-level resilience can complement application and edge-layer recovery mechanisms.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/ab1d8ae3-e094-4679-ae15-8a83896b34c3_x9bk6ors)

### Failover vs. multivalue: the SAP-C02 trap pattern

Failover routing and multivalue routing solve different availability challenges. Failover routing designates a primary endpoint and only shifts traffic to a secondary endpoint when health checks determine the primary is unavailable.

Multivalue routing distributes healthy endpoints simultaneously and allows clients to retry against alternate records without a designated standby target. Understanding this distinction is important because failover emphasizes continuity through active-passive recovery, while multivalue emphasizes availability through multiple concurrent healthy responses.

## Deploying CloudFront with Sub-5-Second Origin Failover and Edge Compute

### CloudFront distribution, origin groups, and edge functions

CloudFront was configured with an origin group that prioritized the us-east-1 Lambda Function URL and automatically failed over to the eu-west-1 origin when failure conditions were detected. This introduced an additional layer of resilience independent of DNS routing behavior.

Edge compute capabilities were incorporated through CloudFront Functions and Lambda@Edge. CloudFront Functions handled lightweight request processing by removing legacy headers at the viewer-request stage, while Lambda@Edge performed signed URL validation at the origin-request stage. Together, these controls strengthened security and request governance before traffic reached backend services.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/ab1d8ae3-e094-4679-ae15-8a83896b34c3_cew82eu8)

### Two-layer failover: CloudFront vs. Route 53

CloudFront origin failover and Route 53 DNS failover operate at different layers of the architecture. CloudFront reacts to origin failures inside the distribution itself, enabling rapid recovery without waiting for DNS cache expiration.

Route 53 operates before requests reach CloudFront by determining which endpoint clients resolve through DNS. Combining both mechanisms provides broader protection against service interruptions by addressing separate failure domains. This layered design raised overall resilience while demonstrating how edge and DNS services work together in a highly available architecture.

## Providing Static IPs for Regulated Banking Partners with Global Accelerator and WAF

### Global Accelerator, NLB intermediaries, and WAF edge rules

Global Accelerator was introduced to provide fixed Anycast IP addresses required by regulated banking partners for firewall allowlisting, compliance validation, and operational consistency. Because Global Accelerator requires supported endpoint types, Network Load Balancers served as the bridge between accelerator endpoints and Lambda-based application logic.

WAF protection was positioned at the CloudFront layer to provide edge-based security controls including geographic filtering, rate limiting, and bot mitigation. This ensured malicious or unwanted requests could be filtered before reaching application resources, reducing exposure and strengthening security posture.

### Static Anycast IPs for PCI-DSS allowlisting

Global Accelerator provisioned two permanent Anycast IPv4 addresses that served as the architecture's stable public entry points. These addresses remained constant even as backend resources changed, simplifying external integrations and compliance requirements.

For regulated environments, static IPs are often mandatory for security reviews, contractual controls, and partner connectivity requirements. Global Accelerator provided this consistency while preserving regional failover capabilities behind the acceleration layer.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/ab1d8ae3-e094-4679-ae15-8a83896b34c3_74lomgbd)

## Delivering the Leadership Package: ADRs, FinOps, and the Director Presentation

### Validation scripts, ADRs, Mermaid topology, and HTML presentation

The project included operational validation tooling, architecture documentation, cost analysis, implementation planning, and executive communication artifacts. A PowerShell-based failover validation process was designed to simulate outages, measure recovery time, and verify restoration behavior.

Architectural Decision Records documented key design choices, while Mermaid diagrams visualized service relationships across regions. FinOps analysis provided cost visibility, and the single-file HTML presentation enabled both technical and executive stakeholders to review the architecture through audience-specific views.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/ab1d8ae3-e094-4679-ae15-8a83896b34c3_ye7qk2ba)

### SLA math and the 60-second failover target

The recovery objective targeted a failover window below 60 seconds, aligning technical implementation with executive expectations around service continuity. Validation focused on confirming that traffic successfully transitioned between regions within this threshold.

This target balanced practical infrastructure recovery timelines with business uptime requirements. The exercise reinforced the relationship between recovery objectives, downtime budgets, and measurable service-level outcomes in highly available cloud environments.

## Proving Sub-60-Second Failover and Zero Orphaned Resources

### End-to-end integration test and kill-primary scenario

End-to-end testing simulated the loss of the primary region while cache-busting requests continuously evaluated CloudFront behavior. The validation process confirmed that failover mechanisms responded correctly under failure conditions and that service availability was maintained.

The testing methodology focused on observable recovery rather than theoretical design assumptions, providing evidence that the architecture could meet stated resilience objectives during real operational events.

### Measured failover duration and clean teardown verification

Testing recorded a failover duration of approximately 1.1 seconds, significantly outperforming the 60-second recovery objective. This demonstrated the effectiveness of the CloudFront origin failover design and validated the recovery workflow under controlled conditions.

Terraform destroy planning confirmed that all managed infrastructure remained under state control. Cross-checking deployed resources against Terraform-managed assets and project tags verified that no orphaned resources remained after validation, supporting strong operational hygiene and cost governance practices.

## Secret Mission: WAF Edge Security Without Touching Application Code

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/ab1d8ae3-e094-4679-ae15-8a83896b34c3_09xysype)

### D2.2 security controls proven at the edge

This exercise demonstrated how security controls can be implemented and managed entirely at the edge without requiring application code changes. WAF policies were defined through Terraform, allowing security controls to remain version-controlled, auditable, and removable through the same infrastructure lifecycle process.

The architecture also demonstrated layered security by combining CloudFront, Lambda@Edge, and WAF protections. A key finding was that creating a WebACL alone is insufficient; association with the CloudFront distribution is required before enforcement can be validated in live traffic scenarios.

## Reflections: Tools, Concepts, and What Principal-Level Looks Like

### Key tools and architectural concepts

Terraform served as the foundation for infrastructure deployment, governance, and lifecycle management. AWS CLI enabled direct validation of deployed resources, while Cursor Composer accelerated coordination and documentation activities across parallel AI-assisted workflows. PowerShell and Bash supported operational testing and verification tasks.

Key concepts reinforced during the project included edge security, automated failover, Anycast networking, infrastructure-as-code discipline, and multi-region resiliency design. Together, these components demonstrated how modern cloud architectures can raise availability, security, and operational confidence through automation-driven practices.

### Time investment and challenges

This project was completed in approximately three hours. The most complex aspect involved validating failover sequencing between CloudFront origin groups, Route 53 health checks, and regional service availability. Troubleshooting timing behavior across multiple layers required careful observation of recovery events and routing transitions.

Additional effort was spent validating WAF rule behavior, ensuring Terraform state accurately reflected deployed resources, and confirming complete teardown of infrastructure. The final result demonstrated the full lifecycle of design, deployment, validation, recovery testing, and cleanup within a controlled cloud environment.

I completed this project to learn how to design and validate a resilient multi-region edge architecture capable of minimizing downtime while supporting compliance-focused operational requirements through automated infrastructure management. I would like to continue building on these skills by exploring blue/green deployment strategies for serverless workloads, enabling zero-downtime application updates with automated rollback and deployment governance.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/ab1d8ae3-e094-4679-ae15-8a83896b34c3)*
