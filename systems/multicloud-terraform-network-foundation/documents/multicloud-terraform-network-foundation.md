<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Multi-Cloud Network Foundation with Terraform

**Project Link:** [View Project](https://learn.nextwork.org/projects/e49e273f-6ad5-4abb-b987-430241b0de80)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/e49e273f-6ad5-4abb-b987-430241b0de80_p4rj7bwt)

## The Multi-Cloud Vision

### Project goals and architecture rationale

This project establishes a multi-cloud network foundation using Terraform, designed to standardize infrastructure patterns across AWS, Azure, and GCP.

The goal is not just provisioning resources, but defining a consistent interface for networking across providers. Each cloud has different primitives, so the architecture separates provider-specific implementation into modules while maintaining a shared structure at the root level.

## Setting Up the Multi-Cloud Toolkit

### Terraform and cloud CLI installation

This phase prepares the execution environment.

Terraform is used as the control layer for infrastructure, while AWS, Azure, and GCP CLIs provide authentication and access. The setup ensures that all providers can be managed from a single terminal, enabling consistent deployment workflows.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/e49e273f-6ad5-4abb-b987-430241b0de80_rn4t8bvx)

### Authenticating to AWS, Azure, and GCP

Terraform needs separate authentication because each cloud provider (AWS, Azure, GCP) has its own security system and authentication methods. Terraform uses your local CLI credentials to communicate with each provider to create and manage resources on your behalf, so it must be able to verify your identity and permissions with each one individually.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/e49e273f-6ad5-4abb-b987-430241b0de80_xt9c4hdn)

## Scaffolding the Terraform Project

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/e49e273f-6ad5-4abb-b987-430241b0de80_r3w8n5jt)

### Provider configuration and shared variables

Each provider is defined within the Terraform configuration with its required settings.

Shared variables create a unified input model across all modules. This ensures that each provider receives consistent inputs, even though the underlying implementations differ.

## Building the AWS VPC Module

### Module interface design

The AWS module establishes the baseline design pattern for all providers.

It defines inputs, outputs, and resource structure in a way that can be replicated across Azure and GCP. This ensures consistency in how networks are created and referenced across environments.

### VPC, subnets, gateways, and routing

The AWS network includes public and private subnets with controlled routing.

Public subnets route through an internet gateway, while private subnets route through a NAT gateway. This design separates external and internal traffic, which is a standard pattern for secure network segmentation.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/e49e273f-6ad5-4abb-b987-430241b0de80_p3yt9fkd)

### Wiring the module and reviewing the plan

The module is integrated into the root configuration and validated using a Terraform plan.

The plan output confirms resource creation, including VPCs, subnets, gateways, and routing components. This step ensures that the configuration produces the expected infrastructure before deployment.

## Extending to Azure and GCP

### Azure VNet and GCP VPC module construction

The architecture is extended by implementing equivalent modules for Azure and GCP.

Each provider uses different networking constructs, so the modules adapt to those differences while maintaining a consistent interface. This allows the system to treat all providers uniformly at the orchestration level.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/e49e273f-6ad5-4abb-b987-430241b0de80_xt5j9amw)

### Full three-cloud plan output

The combined plan reflects all resources across AWS, Azure, and GCP.

Differences in implementation are visible, such as Azure’s default internet access model and GCP’s use of Cloud Router and Cloud NAT. These variations highlight why abstraction at the module level is necessary.

## Deploying Across Three Clouds and Validating with AI

### Running terraform apply across all providers

Deployment is executed through a single Terraform apply, provisioning resources across all providers.

This demonstrates that a unified configuration can manage multiple cloud environments simultaneously, reducing operational complexity.

### Drift detection and state verification

Post-deployment validation ensures that the infrastructure matches the defined configuration.

Terraform state is used to detect drift, confirming that deployed resources remain aligned with the code. This is critical for maintaining consistency over time.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/e49e273f-6ad5-4abb-b987-430241b0de80_xt9w2fma)

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/e49e273f-6ad5-4abb-b987-430241b0de80_bq5y8gzr)

### Claude Code cross-cloud consistency review

Claude Code is used to review the Terraform modules for consistency and security.

It identified inconsistencies in variable definitions and flagged overly permissive firewall rules in GCP. Standardizing variable structures and tightening security rules improved alignment across providers and reduced risk.

## Secret Mission: Cross-Cloud VPN Between AWS and Azure

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/e49e273f-6ad5-4abb-b987-430241b0de80_kx7m2p9w)

### IPsec VPN tunnel managed with Terraform

This extension introduces connectivity between cloud environments.

An IPsec VPN tunnel is configured between AWS and Azure, enabling secure communication across networks. Claude Code recommended stronger encryption settings, such as AES-256, to improve security for production use.

## Cleaning Up and Reflecting on What Was Built

### Destroying resources and cost management

After validation, resources are destroyed using Terraform to avoid unnecessary cost across all providers.

This reinforces that multi-cloud infrastructure should be treated as ephemeral unless needed. Teardown is part of the workflow, not an afterthought.

This project took about 70 minutes. The main challenge was configuring the cross-cloud VPN, specifically aligning pre-shared keys and IP settings between AWS and Azure.

### Key takeaways and reusable module patterns

This project shows how to design reusable Terraform modules with a consistent interface across providers.

The key takeaway is that standardizing inputs and isolating modules is required to manage multi-cloud complexity.

I did this project to understand how to build a consistent network foundation across clouds. The next step is going deeper into infrastructure design and security patterns.

---

*Built with [NextWork](https://learn.nextwork.org) - [View this project](https://learn.nextwork.org/projects/e49e273f-6ad5-4abb-b987-430241b0de80)*
