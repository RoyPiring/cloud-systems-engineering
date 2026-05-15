<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Zero-Key AWS Security Architecture

**Project Link:** [View Project](https://learn.nextwork.org/projects/2474ada8-c282-4e55-8421-ea5270c427f4)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2474ada8-c282-4e55-8421-ea5270c427f4_9y70ngee)

## Building a Zero-Trust Security Architecture for Healthcare

### The mission: eliminating long-lived credentials across 3,000 users

In this project, I designed and validated a zero-trust AWS security architecture for a healthcare environment supporting approximately 3,000 federated users. The objective was to eliminate long-lived credentials and replace them with short-lived, auditable identity workflows built around federation, temporary access, encryption governance, and centralized access control.

The environment combined AWS Organizations, IAM Identity Center, GitHub OIDC federation, multi-region KMS, Secrets Manager rotation, WorkSpaces failover architecture, and ABAC authorization into a unified security model. Instead of relying on static IAM users, SSH keys, or manually distributed credentials, the architecture enforced temporary, policy-driven access where trust is continuously validated.

This project simulated an enterprise healthcare security modernization effort where identity, encryption, auditability, and governance are treated as foundational infrastructure.

### Presenting the architecture to stakeholders

I created an executive-ready presentation package containing SOC 2 control mappings, architecture topology diagrams, SAP-C02 domain references, and a phased 90-day maturity roadmap.

The goal was not only to deploy the architecture technically, but also to communicate operational risk reduction clearly to leadership audiences. The presentation translated complex AWS security concepts into business outcomes such as credential elimination, ransomware resistance, audit readiness, and operational resiliency.

The diagrams were intentionally designed to remain understandable for both engineers and non-technical stakeholders. Every major architecture component mapped directly to a governance or operational objective instead of appearing as isolated AWS services.

## Scaffolding the Multi-Account Infrastructure

### Environment and project goals

I validated OpenTofu, AWS CLI, Git, and GitHub CLI versions before scaffolding the multi-account infrastructure environment.

The OpenTofu project structure included multi-region provider aliases, reusable modules, environment separation, and account-level deployment logic. AWS CLI access was validated against the management account before provisioning began to ensure organization-level APIs and cross-account orchestration workflows would function correctly.

This phase established the foundational control plane the rest of the architecture depended on. The infrastructure was intentionally built as reproducible infrastructure-as-code rather than manual console configuration to preserve auditability and operational consistency.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2474ada8-c282-4e55-8421-ea5270c427f4_sel930xs)

### Multi-region provider configuration

The multi-region provider aliases created two separately configured AWS provider clients inside the same root module.

Each alias pinned a different default region so infrastructure could target the correct API endpoint explicitly without mixing regional deployments accidentally. Resources requiring deterministic placement, such as KMS replicas and disaster-recovery services, referenced the appropriate provider alias directly during deployment.

This pattern becomes critical in enterprise-scale environments because regional separation affects encryption, replication, failover behavior, compliance scope, and service availability. Explicit provider targeting removes ambiguity from infrastructure orchestration and improves deployment predictability.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2474ada8-c282-4e55-8421-ea5270c427f4_40uwpch8)

## Establishing AWS Organizations and Cross-Account Trust

### Multi-account networking strategy

I created an AWS Organization, enabled IAM Identity Center, provisioned VPCs with public and private subnets across multiple accounts, and validated cross-account role assumption using temporary credentials.

The networking design separated management, shared-services, and workload environments into distinct trust boundaries while preserving centralized governance visibility. Temporary STS credentials replaced persistent IAM access patterns during operational testing to reinforce short-lived identity principles across the environment.

This phase established the organizational backbone required for federation, account isolation, centralized governance, and cross-account operational workflows.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2474ada8-c282-4e55-8421-ea5270c427f4_5iq25iff)

### Trust policy placement: the critical pattern

The trust policy for cross-account role assumption was attached to the role inside the target account rather than the calling account.

This distinction is operationally important because the target account owns the resource and therefore defines who is allowed to assume it. The caller account only requires permission to invoke sts:AssumeRole against the role ARN.

In practice, this creates a clean separation between authorization and trust. The spoke account controls who may enter, while the caller controls who may attempt access. Understanding this relationship is critical when designing secure multi-account federation architectures at enterprise scale.

## Deploying a Zero-SSH Active Directory Domain Controller

### Samba 4 on EC2 with Session Manager only

I deployed a Samba 4 Active Directory Domain Controller on EC2 using AWS Systems Manager Session Manager instead of SSH-based administration.

The instance was configured with an IAM instance profile for Session Manager connectivity, and test users and groups were provisioned to simulate healthcare workforce identities and organizational structure.

This deployment model eliminated the need for inbound administrative ports while preserving centralized auditing and controlled administrative access. The Domain Controller functioned as the identity source for downstream federation workflows across AWS services.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2474ada8-c282-4e55-8421-ea5270c427f4_vjqvj7jh)

### Secure access without inbound ports

Session Manager removed the need to expose inbound TCP/22 access on the instance and replaced SSH key management with IAM-authenticated, short-lived administrative sessions.

Every session becomes centrally logged and auditable through Systems Manager and CloudTrail instead of relying on unmanaged private keys stored on operator workstations. This dramatically reduces operational exposure because there are no standing SSH listeners, no long-lived private keys, and no unmanaged credential distribution.

The broader architectural objective was reducing persistent trust anchors across the environment wherever possible.

## Federating 3,000 Users Through AD Connector and Identity Center

### Connecting on-prem AD to AWS identity

I deployed a Small AD Connector integrated with the Samba Domain Controller and connected it to IAM Identity Center to synchronize users and groups into AWS federation workflows.

Permission Sets were then created and mapped to federated identities to validate secure access into workload accounts without relying on locally managed IAM users.

This architecture centralizes identity management while allowing AWS accounts to inherit enterprise authentication standards automatically. The model also improves operational scalability because identity lifecycle management remains tied to Active Directory rather than duplicated across cloud accounts manually.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2474ada8-c282-4e55-8421-ea5270c427f4_lxebjjz7)

## Achieving Zero-Key CI/CD with GitHub OIDC and Permission Boundaries

### Replacing stored secrets with federated identity

I created a GitHub OIDC identity provider and configured a deployment role that allows GitHub Actions workflows to authenticate using short-lived federated credentials instead of stored AWS secrets.

The deployment pipeline assumed the role dynamically during execution and inherited scoped permissions through trust policies and permission boundaries. This removed the need for long-lived access keys inside CI/CD systems entirely.

The architectural benefit is significant because the CI/CD platform no longer stores reusable credentials that could be leaked, copied, or abused outside the intended execution window.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2474ada8-c282-4e55-8421-ea5270c427f4_0n63kl4w)

### Capping deployment role permissions with boundaries

The Permission Boundary established the maximum permissions the deployment role could ever receive regardless of what inline or managed IAM policies were attached later.

Explicit deny statements blocked actions such as iam:CreateUser and cloudtrail:StopLogging even though the role still retained broad operational deployment permissions elsewhere.

This design demonstrates layered privilege control. IAM policies grant capability, but permission boundaries constrain the maximum operational blast radius regardless of future policy drift or accidental privilege expansion.

## Enforcing ABAC Isolation Across Projects Without Policy Rewrites

### Session tags as the access control mechanism

I implemented attribute-based access control using session tags as the primary authorization mechanism across project environments.

The deployment role inherited project-scoped session tags during role assumption, and downstream policies evaluated those tags dynamically during authorization decisions. This removed the need to maintain separate IAM policies for every project individually.

The environment became policy-scalable because authorization logic moved from static IAM policy sprawl into reusable tag-driven controls.

### Why ABAC scales where per-project policies fail

Traditional per-project IAM policies become operationally difficult to manage as environments scale because each new project introduces additional policy duplication and maintenance overhead.

ABAC avoids this by using reusable policy logic that evaluates principal tags and resource tags dynamically at request time. Instead of creating hundreds of near-identical policies, one policy framework can govern many projects consistently through metadata alignment.

This dramatically improves operational maintainability while reducing policy fragmentation across large organizations.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2474ada8-c282-4e55-8421-ea5270c427f4_s1ezbdg5)

### Where the isolation check happens

The isolation enforcement occurred at multiple layers simultaneously.

STS evaluated the role trust policy and session-tag requirements during AssumeRole execution, while S3 evaluated bucket-policy conditions during GetObject requests. The bucket policy explicitly denied access whenever the principal tag and object tag values did not match.

This layered evaluation model is important because access enforcement does not happen in a single location. Identity trust, session metadata, and resource authorization all participate in the final decision path.

## Implementing Multi-Region KMS Keys and Envelope Encryption

### Multi-region key replication strategy

I created a multi-region KMS key, replicated it across regions, and implemented envelope encryption workflows using OpenSSL and KMS-generated AES-256 data keys.

Temporary KMS grants were also created and retired dynamically to simulate short-lived Lambda access patterns without permanently expanding the key policy.

This architecture improves resiliency because encrypted workloads can fail over between regions while maintaining cryptographic continuity and centralized governance.

### The envelope encryption flow for PHI data

KMS generated a plaintext AES-256 data key alongside a KMS-encrypted ciphertext copy of the same key.

The plaintext key was used locally with OpenSSL to encrypt the PHI sample file, while the encrypted data key was stored safely alongside the ciphertext. During decryption, KMS recovered the plaintext key bytes temporarily so the file could be restored.

The plaintext data key file was deleted immediately after use because anyone possessing that file could bypass KMS entirely and decrypt the protected data directly.

This reinforces an important operational principle: KMS is only effective if plaintext key exposure is tightly controlled.

### Key policies vs grants: choosing the right tool

Key policies define durable, long-term authorization rules attached directly to the CMK itself.

Grants provide temporary delegated permissions for specific principals without modifying the central key policy. This makes grants ideal for short-lived workloads such as Lambda execution or transient automation jobs.

The architectural distinction matters because stable organization-wide rules belong in the key policy, while temporary operational access patterns are safer and easier to manage through grants.

## Exposing the Parameter Store Trap and Automating Secrets Rotation

### Why Secrets Manager wins the rotation question

I demonstrated the operational limitation of Parameter Store SecureString by validating that it lacks native secrets-rotation orchestration comparable to Secrets Manager.

To address this, I deployed an RDS MySQL instance integrated with AWS Secrets Manager managed rotation workflows and forced a manual rotation cycle to validate zero-downtime credential rollover.

Secrets Manager models rotation as a first-class capability with lifecycle state management, scheduling, and orchestration support built directly into the service.

This becomes operationally critical in regulated environments where password rotation cannot depend on custom scripts or manual administrative workflows.

### The critical differentiator for the SAP-C02 exam

Secrets Manager supports native automated credential rotation workflows for supported AWS services such as RDS.

Parameter Store SecureString functions primarily as encrypted configuration storage with versioning support, but it does not provide equivalent built-in rotation orchestration, scheduling, or lifecycle-state tracking.

For architectures requiring automatic credential rotation and operational governance, Secrets Manager becomes the correct design choice because rotation is treated as a managed product feature rather than a custom engineering responsibility.

### Proving zero-downtime rotation with staging labels

The forced rotation workflow produced multiple secret versions carrying labels such as AWSCURRENT, AWSPREVIOUS, and AWSPENDING.

During rotation, the pending version temporarily coexisted with the current production credential until the database completed credential synchronization successfully. Once finalized, the new credential inherited the AWSCURRENT label while the old credential transitioned to AWSPREVIOUS.

This demonstrated how Secrets Manager supports seamless credential transitions without introducing application downtime during password rollover events.

## Proving Ransomware Resistance with S3 Object Lock

### Object Lock, Bucket Keys, and MFA Delete combined

I created a versioned S3 bucket with Object Lock enabled in Compliance Mode, activated S3 Bucket Keys, and configured MFA Delete to simulate ransomware-resistant storage protection.

Attack simulations attempted both object overwrite and deletion operations. The environment blocked both actions successfully because retention policies and immutability controls prevented destructive modification.

This architecture demonstrates defense-in-depth storage protection where encryption, retention, and administrative friction combine to reduce ransomware impact significantly.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2474ada8-c282-4e55-8421-ea5270c427f4_azs7ahos)

### Compliance mode: the override that has no override

Compliance Mode enforces regulatory-grade immutability where no administrator, role, or operational user can shorten retention windows or delete protected object versions before expiration.

Governance Mode still allows privileged users with bypass permissions to override retention when necessary for operational recovery scenarios. Compliance Mode intentionally removes that escape path entirely.

The difference reflects two separate trust models. Governance assumes trusted administrators may require break-glass capabilities. Compliance assumes no one inside the organization should be capable of bypassing retention protections during the enforcement window.

## Deploying WorkSpaces with Cross-Region Failover Architecture

### Remote desktop failover for the remote workforce

I integrated WorkSpaces with the AD Connector directory and deployed Ubuntu-based WorkSpaces desktops using AutoStop mode to simulate remote workforce operations.

A WorkSpaces Connection Alias and failover-ready DNS architecture were then configured to support regional failover behavior through Route 53.

This design supports operational continuity for distributed healthcare staff while maintaining centralized authentication and identity governance across regions.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2474ada8-c282-4e55-8421-ea5270c427f4_zh6kiu96)

### How Route 53 uses the ConnectionIdentifier for failover

How Route 53 uses the ConnectionIdentifier for failover

The WorkSpaces Connection Alias generated a region-specific ConnectionIdentifier value associated with the registered directory.

Route 53 TXT records mapped the alias FQDN to these identifiers using failover routing policies and health checks. During failover events, clients resolve the healthy region automatically without requiring end users to change connection details manually.

This creates a user-transparent failover mechanism where identity routing adapts dynamically to regional availability conditions.

## Presenting the Architecture to Leadership

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2474ada8-c282-4e55-8421-ea5270c427f4_fb5bi8tz)

### SOC-2 mapping and the 90-day roadmap

The executive presentation package translated technical implementation details into operational outcomes aligned to SOC 2 governance objectives.

The roadmap structured maturity improvements into phased delivery milestones while architecture diagrams visually mapped trust boundaries, encryption domains, identity flows, and operational protections.

Every section focused on communicating risk reduction and operational value clearly instead of overwhelming leadership audiences with raw AWS terminology.

## Red-Teaming the Architecture: Four Attacks, Four Blocks

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/2474ada8-c282-4e55-8421-ea5270c427f4_cb50lyfg)

### What the error responses reveal about AWS enforcement

The red-team phase validated how AWS authorization engines communicate enforcement decisions across IAM, KMS, and S3.

The S3 denial path clearly identified explicit resource-policy denial behavior, while KMS responses highlighted the complexity of effective authorization evaluation where IAM, key policies, and session tags intersect simultaneously.

The exercise reinforced that AWS authorization decisions are rarely controlled by a single policy type. Effective access emerges from the combined evaluation of IAM policies, SCPs, resource policies, trust relationships, permission boundaries, grants, and session attributes together.

## Reflections and Key Takeaways

### Tools and concepts mastered

This project combined OpenTofu, AWS CLI, Git, GitHub CLI, Cursor IDE, AWS Organizations, IAM Identity Center, KMS, Secrets Manager, S3 Object Lock, WorkSpaces, Session Manager, GitHub OIDC federation, and ABAC authorization models into a production-style zero-trust architecture.

The concepts reinforced throughout the build included identity federation, multi-account governance, short-lived credentials, envelope encryption, ransomware resistance, secrets rotation, attribute-based access control, compliance mapping, and security validation workflows.

### Time and challenges

This project required approximately six hours of focused engineering effort to complete successfully.

The most challenging aspect was understanding and validating the interaction boundaries between the large number of AWS services participating in the architecture simultaneously. Many controls appeared simple individually, but their combined behavior across trust policies, KMS evaluation, STS federation, SCP enforcement, permission boundaries, and resource policies required careful validation to ensure the architecture behaved predictably under both operational and adversarial conditions.

This project reinforced how enterprise cloud security is fundamentally about controlling trust relationships and authorization flow rather than simply enabling individual services.

I completed this project to deepen my understanding of enterprise-grade AWS security architecture, eliminate long-lived credentials across cloud workflows, validate zero-trust operational patterns, and build executive-ready governance reporting aligned to regulated healthcare environments.

The next area I want to improve is advanced AWS network-security architecture, particularly around VPC Flow Logs, distributed intrusion detection systems, east-west traffic inspection, and automated threat-hunting workflows across multi-account environments.

---

*Built with [NextWork](https://learn.nextwork.org) - [View this project](https://learn.nextwork.org/projects/2474ada8-c282-4e55-8421-ea5270c427f4)*
