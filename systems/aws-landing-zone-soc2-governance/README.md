# Architect an AWS Landing Zone from Scratch

> Inside the [Cloud Systems Engineering](../../README.md) portfolio · *Cloud platforms engineered for scale, reliability, and uptime.*

## Overview

In this project, I built a production-style AWS landing zone designed to support a rapidly growing gaming company operating under SOC 2 compliance requirements. The objective was to establish a scalable multi-account governance model that could standardize security controls, automate account onboarding, and reduce operational drift as new business units and acquisitions were integrated into the environment.

The architecture combined AWS Organizations, Terraform, Service Control Policies, StackSets, IAM Identity Center, and Service Catalog into a centralized governance framework. Instead of treating cloud accounts as isolated deployments, the environment was designed as a governed operating platform where security, monitoring, and compliance controls inherit automatically through organizational structure.

The architecture is built across **10 phases**, anchored by **Architecting NovaBurst's Governance Foundation** on the input side and **Scaling Governance for the Next Acquisition** at the end. Each phase is listed in the Implementation section below.

## Architecture

```mermaid
---
title: Architect an AWS Landing Zone from Scratch
---
%%{init: {"theme":"base","themeVariables": {"primaryColor":"#1B4332","primaryTextColor":"#F4D03F","primaryBorderColor":"#F4D03F","secondaryColor":"#264653","tertiaryColor":"#2F5233","lineColor":"#F4D03F","fontFamily":"ui-monospace, SFMono-Regular, Menlo, Consolas, monospace","fontSize":"13px"}}}%%
flowchart LR
    classDef datastore fill:#264653,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef service fill:#1B4332,stroke:#F4D03F,stroke-width:2px,color:#F4D03F
    classDef event fill:#7B42BC,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef io fill:#0d1117,stroke:#F4D03F,stroke-width:1.5px,color:#F4D03F,font-style:italic

    subgraph IaC["Infrastructure-as-Code Bootstrap"]
        Cursor(Cursor Composer Scaffold)
        Terraform(Terraform Org Module)
        ADRs[(Architecture Decision Records)]
    end

    subgraph Org["AWS Organization Root"]
        MgmtAcct(Management Payer Account)
        FullAccess(FullAWSAccess Baseline SCP)
        OrgRoot(Organization Root)
    end

    subgraph OUs["4-OU Topology"]
        ProdOU(Production OU)
        SandboxOU(Sandbox OU)
        SecurityOU(Security OU)
        SharedOU(Shared Services OU)
    end

    subgraph SCPLib["SCP Library - 6 Preventive Controls"]
        SCPRegions(Restricted-Regions SCP)
        SCPEscape(Governance-Escape Prevention SCP)
        SCPServices(Unauthorized-Service Block SCP)
        SCPBoundaries(Operational-Boundary SCP)
        SCPRedTeam{{Cursor Auditor Red-Team Review}}
    end

    subgraph Identity["IAM Identity Center"]
        IdC(IAM Identity Center)
        PermSets[(Permission Sets - Operational Tiers)]
        AuditRole(Cross-Account Audit Role)
    end

    subgraph Baseline["StackSets Security Baseline - Auto-Deploy"]
        StackSets(CloudFormation StackSets)
        CloudTrail(CloudTrail)
        Config(AWS Config)
        GuardDuty(GuardDuty)
    end

    subgraph Vending["Account Vending Machine - Secret Mission"]
        SvcCatalog(Service Catalog)
        AVM(Account Vending Workflow)
        TagPolicy(Studio and CostCenter Tags)
        NewMember[/New Member Account/]
    end

    subgraph Evidence["SOC 2 Evidence Layer"]
        SmokeTests(SCP Smoke-Test Suite)
        FRM[(Findings Remediation Matrix)]
        HtmlReport[/Executive HTML Engagement Report/]
        DriftCheck{{StackSet Drift Validation}}
    end

    Cursor -->|scaffolds| Terraform
    Terraform -->|reviewed against| ADRs
    ADRs -->|approve before apply| Terraform
    Terraform -->|provisions| OrgRoot
    Terraform -->|enables policy types on| OrgRoot
    MgmtAcct -->|owns| OrgRoot
    FullAccess -->|baseline allow-list at| OrgRoot
    OrgRoot -->|contains| ProdOU
    OrgRoot -->|contains| SandboxOU
    OrgRoot -->|contains| SecurityOU
    OrgRoot -->|contains| SharedOU
    SCPRegions -->|attached to| ProdOU
    SCPEscape -->|attached to| OrgRoot
    SCPServices -->|attached to| ProdOU
    SCPBoundaries -->|attached to| SandboxOU
    SCPRedTeam -->|validates each policy before deploy| SCPRegions
    SCPRedTeam -->|validates each policy before deploy| SCPEscape
    SCPRedTeam -->|validates each policy before deploy| SCPServices
    SCPRedTeam -->|validates each policy before deploy| SCPBoundaries
    IdC -->|federates identities into| OrgRoot
    PermSets -->|hydrates session-bound roles via| IdC
    AuditRole -->|trust limited to| SecurityOU
    AuditRole -->|cross-account read into| ProdOU
    AuditRole -->|cross-account read into| SandboxOU
    AuditRole -->|cross-account read into| SharedOU
    StackSets -->|auto-deploys| CloudTrail
    StackSets -->|auto-deploys| Config
    StackSets -->|auto-deploys| GuardDuty
    CloudTrail -->|landing zone audit log into| SecurityOU
    Config -->|configuration drift into| SecurityOU
    GuardDuty -->|threat findings into| SecurityOU
    SvcCatalog -->|exposes| AVM
    AVM -->|provisions| NewMember
    AVM -->|applies| TagPolicy
    NewMember -->|placed under| ProdOU
    NewMember -->|inherits SCPs through| OrgRoot
    StackSets -.->|auto-onboards baselines into| NewMember
    SmokeTests -->|GDPR region + escape-prevention assertions feed| FRM
    DriftCheck -->|stack-state evidence into| FRM
    FRM -->|sourced into| HtmlReport
    HtmlReport -->|delivered to executives and SOC 2 auditors| Evidence
    class ADRs,PermSets,FRM datastore
    class Cursor,Terraform,MgmtAcct,FullAccess,OrgRoot,ProdOU,SandboxOU,SecurityOU,SharedOU,SCPRegions,SCPEscape,SCPServices,SCPBoundaries,IdC,AuditRole,StackSets,CloudTrail,Config,GuardDuty,SvcCatalog,AVM,TagPolicy,SmokeTests service
    class SCPRedTeam,DriftCheck event
    class NewMember,HtmlReport io
```

The diagram shows the topology and data flow of the system as built. The full architectural narrative, with screenshots and prose, lives in [`documents/aws-landing-zone-soc2-governance.md`](./documents/aws-landing-zone-soc2-governance.md).

## Implementation

This system is built across **10 phases**:

1. **Architecting NovaBurst's Governance Foundation**
2. **Environment Setup and Terraform Initialization**
3. **Designing the Multi-Account OU Topology**
4. **Bootstrapping the AWS Organization with Terraform**
5. **Building the SCP Library to Close Every Auditor Finding**
6. **Deploying the StackSets Security Baseline**
7. **Centralizing Identity Governance with IAM Identity Center**
8. **Proving Compliance: SCP Smoke Tests and Operational Artifacts**
9. **Delivering the Principal Architect's Engagement Report**
10. **Scaling Governance for the Next Acquisition**

For the full walkthrough with screenshots and step-by-step content, see [`documents/aws-landing-zone-soc2-governance.md`](./documents/aws-landing-zone-soc2-governance.md).

## Validation

Each build phase below is documented in [`documents/aws-landing-zone-soc2-governance.md`](./documents/aws-landing-zone-soc2-governance.md), with screenshots, configuration, and notes as captured during the build:

- ✅ Architecting NovaBurst's Governance Foundation
- ✅ Environment Setup and Terraform Initialization
- ✅ Designing the Multi-Account OU Topology
- ✅ Bootstrapping the AWS Organization with Terraform
- ✅ Building the SCP Library to Close Every Auditor Finding
- ✅ Deploying the StackSets Security Baseline
- ✅ Centralizing Identity Governance with IAM Identity Center
- ✅ Proving Compliance: SCP Smoke Tests and Operational Artifacts
- ✅ Delivering the Principal Architect's Engagement Report
- ✅ Scaling Governance for the Next Acquisition
