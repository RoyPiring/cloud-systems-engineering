<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# ECS vs EKS vs Lambda Architecture

**Project Link:** [View Project](https://nextwork.ai/projects/d2b161e2-121d-47ae-887d-99d674ac1717)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/d2b161e2-121d-47ae-887d-99d674ac1717_8as2zu3c)

## Architecting Under Pressure: The Meridian Payments Mission

### Mission scope and objectives

This project focused on evaluating three AWS compute models for a payment-processing platform while maintaining security, resiliency, and deployment agility. The objective was to compare ECS on EC2, ECS on Fargate, and EKS against a serverless event-driven architecture to determine which approach best aligned with Meridian Payments' operational and compliance requirements.

The engagement also introduced a multi-agent development workflow using Cursor Composer agents, allowing infrastructure, deployment automation, testing, architecture planning, and documentation activities to progress in parallel. The outcome was a complete environment capable of demonstrating deployment patterns, security controls, workload resiliency, and architectural tradeoffs across multiple AWS services.

### Tool stack and verified versions

Rather than installing new tooling, this phase focused on validating the existing workstation environment and preparing the project workspace for implementation. Terraform, AWS CLI, and Docker Desktop were confirmed as available and ready for use, while Grafana k6 was identified as missing and requiring installation before performance testing activities could begin.

Project initialization included configuring local AWS settings, creating the required repository structure, establishing environment configuration files, and preparing source control. The setup provided a clean and reproducible foundation for infrastructure provisioning, application deployment, testing, and documentation activities without introducing unnecessary workstation modifications.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/d2b161e2-121d-47ae-887d-99d674ac1717_4u9wkin4)

### Five-agent strike team mobilization

The project used a five-agent delivery model that divided responsibilities across infrastructure, deployment automation, event architecture, testing, and program management functions. This approach allowed multiple workstreams to progress simultaneously while maintaining clear ownership boundaries.

Foundation focused on Terraform-based infrastructure components, including ECR and networking resources. Pipeline concentrated on deployment automation and release controls. Backbone developed the serverless event-processing components. Breaker established testing, chaos engineering, and validation assets. Narrator produced architectural documentation, planning artifacts, cost analysis, and executive presentation materials.

All initial deliverables were completed before additional source control commits were introduced, ensuring every workstream started from a consistent project baseline.

## Securing the Container Supply Chain with ECR and Scan Gates

### Terraform foundation and ECR setup

The platform foundation was built around Terraform-managed infrastructure with container images stored in Amazon ECR. The repository configuration included image scanning on push and lifecycle management policies designed to retain only the most recent container versions while automatically removing older artifacts.

A Python Flask application was containerized using Docker and published to ECR as the deployment artifact for downstream compute platforms. This established a single application package that could be consistently deployed across ECS, EKS, and other target environments.

Security validation was integrated directly into the deployment workflow through automated vulnerability scanning, ensuring images were assessed before progressing through release stages.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/d2b161e2-121d-47ae-887d-99d674ac1717_z7732znh)

### PCI-DSS scan gate logic and compliance impact

The scan-gate.sh process functions as a deployment control point within the software supply chain. Before an image can advance toward production, the script validates ECR scan results and checks for critical vulnerabilities.

This control reduces the risk of introducing known high-severity security issues into production workloads. By enforcing an automated block when CRITICAL findings are detected, the deployment process creates a measurable security checkpoint that supports secure software delivery practices and provides auditable evidence of vulnerability review activities.

For payment-processing environments, this type of gate strengthens governance by ensuring security validation becomes part of the release process rather than a manual post-deployment activity.

## Proving Three Compute Paths on One Workload

### ECS-EC2, ECS-Fargate, and EKS deployment strategy

This phase evaluated a single containerized application across three AWS compute platforms to understand operational complexity, deployment flexibility, and runtime characteristics. A shared VPC architecture and Application Load Balancer provided a common networking layer while weighted target groups enabled controlled traffic distribution.

Running the same workload across ECS on EC2, ECS on Fargate, and EKS created a practical comparison of infrastructure ownership models. ECS-EC2 offered maximum host-level control, Fargate reduced operational overhead through managed compute, and EKS provided Kubernetes-based orchestration capabilities for organizations requiring platform portability and advanced container management.

The architecture allowed health validation and performance testing to be performed consistently across all deployment targets.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/d2b161e2-121d-47ae-887d-99d674ac1717_6bykzds2)

### awsvpc isolation and PCI-DSS network security

The awsvpc networking model was selected to provide task-level network isolation. Each ECS task receives its own Elastic Network Interface and associated security controls, allowing network access policies to be enforced at the workload level instead of the host level.

This design supports segmentation requirements by limiting inbound access paths and reducing unnecessary lateral communication between services. Payment-related workloads remain isolated within private subnets while accepting traffic only from approved application entry points.

From an operational perspective, task-level isolation raises visibility, simplifies security reviews, and creates clearer audit boundaries when validating access controls within regulated environments.

## Refactoring the Webhook Dispatcher into a Serverless Event Backbone

### API Gateway, Lambda SnapStart, Step Functions, and EventBridge

The legacy webhook dispatcher was redesigned into an event-driven architecture using managed AWS services. API Gateway serves as the public ingestion layer, while Lambda handles request acceptance and event publication into the processing backbone.

EventBridge, Step Functions, and DynamoDB separate ingestion from downstream processing activities, allowing individual components to evolve independently without impacting external integrations. This architecture raises scalability and reduces coupling between services.

The design objective was to support high transaction volumes while maintaining operational simplicity and minimizing infrastructure management requirements. Using managed services also reduced the number of servers and runtime components requiring ongoing maintenance.

### Migration strategy and Express vs Standard decision

The migration approach used controlled traffic shifting through Lambda aliases and weighted routing. This allowed new application versions to receive traffic incrementally while maintaining rollback capability throughout the deployment process.

Lambda SnapStart was used to reduce initialization latency during version transitions, helping maintain consistent performance characteristics during rollout events.

Step Functions Express was selected because the workflow consisted primarily of short-lived event processing operations. The workload pattern favored rapid execution and high transaction volume rather than long-running orchestration. As a result, Express aligned more closely with both performance objectives and operational efficiency requirements.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/d2b161e2-121d-47ae-887d-99d674ac1717_3ev97t62)

## Deploying with Zero-Downtime Blue/Green Pipelines

### CodeDeploy canary configuration and auto-rollback

Deployment automation was implemented using AWS CodeDeploy blue/green release patterns. Canary traffic shifting allowed new application versions to be validated under live traffic conditions before receiving full production load.

Automatic rollback protections were included to reduce deployment risk. If health checks or validation metrics indicated instability, traffic could immediately return to the previously known-good environment.

This deployment model supports continuous delivery objectives while reducing the operational impact of release failures. It also provides a repeatable deployment framework that can be applied consistently across future application versions.

### Traffic shifting strategy and deployment group details

The deployment group used the CodeDeployDefault.ECSCanary10Percent5Minutes strategy. Under this configuration, 10% of incoming traffic is directed to the new task set and monitored for five minutes before broader rollout occurs.

If the deployment remains healthy during the observation window, traffic transitions completely to the green environment and the previous task set is retired. If issues are detected, rollback procedures return traffic to the blue environment without requiring manual intervention.

This strategy balances deployment velocity with operational safety by validating production readiness before committing to a full release.

## Chaos-Testing the Platform and Proving Enterprise SLA

### Load testing, chaos drill, and DLQ validation

Performance and resiliency testing were performed to validate the platform under both normal and failure conditions. Load testing verified traffic distribution behavior, while chaos testing intentionally introduced failures to observe recovery characteristics.

Dead-letter queue validation ensured malformed webhook events were captured and isolated rather than disrupting normal processing workflows. This approach protects the primary event pipeline while preserving failed messages for investigation and remediation.

Together, these tests provided evidence that the platform could sustain expected operational conditions while maintaining predictable behavior during fault scenarios.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/d2b161e2-121d-47ae-887d-99d674ac1717_mcwfteof)

### Fargate recovery proof and SLA implications

Testing demonstrated that ECS automatically replaced failed Fargate tasks and restored desired service capacity without manual intervention. The recovery process confirmed that workload availability remained governed by service definitions rather than individual task health.

This behavior is important for enterprise environments where recovery speed directly affects service availability objectives. Automated replacement reduces operational response requirements and raises confidence that transient failures will not result in prolonged service disruption.

The exercise validated the platform's ability to self-heal and reinforced the value of managed orchestration services in production environments.

## Delivering the Board Presentation and Leadership Package

### ADRs, FinOps report, and 8-slide briefing

Project outputs extended beyond implementation to include executive-level decision support materials. Architecture Decision Records documented the rationale behind major technical choices, creating traceability between requirements, constraints, and implementation outcomes.

Supporting artifacts included topology diagrams, implementation planning documents, dependency mapping across the five-agent workflow, cost analysis, and an executive presentation package.

These deliverables transformed technical findings into formats suitable for leadership review, helping stakeholders understand both architectural tradeoffs and business implications.

### CISO-targeted ADR and business consequence framing

ADR-002 documented the decision to adopt awsvpc networking for ECS workloads. The record focused on security isolation requirements and the need for workload-level access control boundaries.

The decision explicitly linked technical implementation choices to compliance outcomes by documenting why alternative networking modes were rejected. This format helps security leadership understand both the control being implemented and the operational risk being reduced.

Capturing these decisions formally raises governance, supports future audits, and provides historical context for architectural reviews.

## Final Validation: Scan Gate Demo and Real Numbers in the Presentation

### End-to-end deployment proof and validation sweep

Final validation activities combined deployment testing, security controls verification, and documentation updates into a single end-to-end review cycle. A new application version was deployed through the blue/green pipeline to validate traffic shifting behavior and deployment stability.

Security controls were then tested by introducing a deliberately vulnerable image to confirm the ECR scan gate correctly prevented progression through the release process.

The final validation sweep ensured operational metrics, deployment outcomes, and testing results were reflected accurately within project documentation and presentation materials.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/d2b161e2-121d-47ae-887d-99d674ac1717_4o17yp0x)

### Scan gate evidence log and PCI-DSS audit trail

The validation logs demonstrate that the deployment pipeline successfully identified critical vulnerabilities and prevented a vulnerable image from reaching downstream compute environments.

While this does not constitute complete PCI-DSS certification evidence, it does provide verifiable proof that a security control operated as designed. The logs show detection, evaluation, and enforcement actions occurring before production deployment.

This distinction is important because compliance programs require multiple layers of evidence, whereas the scan gate demonstrates only one specific control within a broader security framework.

## Secret Mission: Service Connect, EventBridge Pipes, and Phase 2 Positioning

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/d2b161e2-121d-47ae-887d-99d674ac1717_9bcogzlh)

### Eliminating Lambda glue with EventBridge Pipes

EventBridge Pipes simplified the integration architecture by replacing custom polling and message-handling logic with a managed AWS service. The solution connected source and destination services while reducing the amount of operational code requiring maintenance.

Compared with a custom Lambda integration layer, Pipes removed responsibilities such as polling management, scaling behavior, retry orchestration, and portions of error handling logic.

This reduced operational complexity while allowing engineering effort to remain focused on business functionality instead of integration infrastructure.

## Reflections: Tools, Concepts, and What Comes Next

### Key tools and architectural concepts mastered

This project provided practical experience across multiple AWS architecture patterns and delivery models. Core technologies included Terraform, AWS CLI, Docker, ECS, EKS, API Gateway, Lambda, Step Functions Express, EventBridge Pipes, and k6.

The most valuable concepts were understanding compute tradeoffs, designing deployment safety mechanisms, implementing automated security controls, and validating resiliency through structured testing. Comparing container-based and serverless approaches highlighted how operational ownership, scalability characteristics, and cost considerations influence architectural decisions.

The use of AI-assisted development workflows also demonstrated how specialized agents can accelerate implementation while maintaining separation of responsibilities.

### Time to completion and toughest challenges

The project was completed in approximately four hours. The most challenging aspect involved troubleshooting IAM permissions associated with EventBridge Pipes and validating communication paths between SQS, Lambda, and Step Functions.

Diagnosing why the pipe remained in a STOPPED state required careful review of execution roles, resource permissions, and service integration requirements. The experience reinforced the importance of least-privilege design and highlighted how service-to-service permissions can become a significant factor in event-driven architectures.

### Learning goals and next steps

I completed this project to gain hands-on experience evaluating ECS, EKS, and Lambda deployment models while building a resilient event-driven architecture capable of supporting production-style workloads.

My next area of focus is observability and compliance automation. Building deeper expertise in monitoring, tracing, security controls, and governance workflows will help extend these architectures beyond deployment and into long-term operational management. Understanding how to continuously measure reliability, performance, and compliance will be the natural progression from the foundations established in this project.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/d2b161e2-121d-47ae-887d-99d674ac1717)*
