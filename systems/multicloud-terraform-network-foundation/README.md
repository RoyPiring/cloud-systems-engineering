# Multi-Cloud Network Foundation with Terraform

> Inside the [Cloud Systems Engineering](../../README.md) portfolio · *Cloud platforms engineered for scale, reliability, and uptime.*

## Overview

This project establishes a multi-cloud network foundation using Terraform, designed to standardize infrastructure patterns across AWS, Azure, and GCP.

The goal is not just provisioning resources, but defining a consistent interface for networking across providers. Each cloud has different primitives, so the architecture separates provider-specific implementation into modules while maintaining a shared structure at the root level.

The architecture is built across **7 phases**, anchored by **The Multi-Cloud Vision** on the input side and **Cross-Cloud VPN Between AWS and Azure** at the end. Each phase is listed in the Implementation section below.

## Architecture

```mermaid
---
title: Multi-Cloud Network Foundation with Terraform
---
%%{init: {"theme":"base","themeVariables": {"primaryColor":"#1B4332","primaryTextColor":"#F4D03F","primaryBorderColor":"#F4D03F","secondaryColor":"#264653","tertiaryColor":"#2F5233","lineColor":"#F4D03F","fontFamily":"ui-monospace, SFMono-Regular, Menlo, Consolas, monospace","fontSize":"13px"}}}%%
flowchart LR
    classDef datastore fill:#264653,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef service fill:#1B4332,stroke:#F4D03F,stroke-width:2px,color:#F4D03F
    classDef event fill:#7B42BC,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef io fill:#0d1117,stroke:#F4D03F,stroke-width:1.5px,color:#F4D03F,font-style:italic







    Vars[/Shared Variables/]
    TF(Terraform Root Config)
    State[(Terraform State)]

    subgraph IaC["IaC Control Layer"]
        TF
        State
        AWSMod(AWS VPC Module)
        AzMod(Azure VNet Module)
        GCPMod(GCP VPC Module)
    end

    subgraph AWS["AWS"]
        VPC(VPC + Public/Private Subnets)
        IGW{{Internet Gateway}}
        NAT{{NAT Gateway}}
        AWSRT(Route Tables)
    end

    subgraph Azure["Azure"]
        VNet(VNet + Subnets)
        AzDef{{Default Internet Egress}}
    end

    subgraph GCP["GCP"]
        GVPC(GCP VPC + Subnets)
        CRouter{{Cloud Router}}
        CNAT{{Cloud NAT}}
        FW(Firewall Rules)
    end

    VPN{{IPsec VPN Tunnel AES-256}}
    Claude(Claude Code Consistency Review)

    Vars -->|inputs| TF
    TF -->|invokes| AWSMod
    TF -->|invokes| AzMod
    TF -->|invokes| GCPMod
    TF <-->|drift detection| State

    AWSMod -->|provisions| VPC
    VPC --> IGW
    VPC --> NAT
    VPC --> AWSRT

    AzMod -->|provisions| VNet
    VNet --> AzDef

    GCPMod -->|provisions| GVPC
    GVPC --> CRouter
    CRouter --> CNAT
    GCPMod -->|provisions| FW

    VPC <-->|cross-cloud IPsec| VPN
    VPN <-->|cross-cloud IPsec| VNet

    Claude -.->|flags loose FW + weak crypto| GCPMod
    Claude -.->|recommends AES-256| VPN
    class State datastore
    class TF,AWSMod,AzMod,GCPMod,VPC,AWSRT,VNet,GVPC,FW,Claude service
    class IGW,NAT,AzDef,CRouter,CNAT,VPN event
    class Vars io
```

The diagram shows the topology and data flow of the system as built. The full architectural narrative, with screenshots and prose, lives in [`documents/multicloud-terraform-network-foundation.md`](./documents/multicloud-terraform-network-foundation.md).

## Implementation

This system is built across **7 phases**:

1. **The Multi-Cloud Vision**
2. **Setting Up the Multi-Cloud Toolkit**
3. **Scaffolding the Terraform Project**
4. **Building the AWS VPC Module**
5. **Extending to Azure and GCP**
6. **Deploying Across Three Clouds and Validating with AI**
7. **Cross-Cloud VPN Between AWS and Azure**

For the full walkthrough with screenshots and step-by-step content, see [`documents/multicloud-terraform-network-foundation.md`](./documents/multicloud-terraform-network-foundation.md).

## Validation

Each build phase below is documented in [`documents/multicloud-terraform-network-foundation.md`](./documents/multicloud-terraform-network-foundation.md), with screenshots, configuration, and notes as captured during the build:

- ✅ The Multi-Cloud Vision
- ✅ Setting Up the Multi-Cloud Toolkit
- ✅ Scaffolding the Terraform Project
- ✅ Building the AWS VPC Module
- ✅ Extending to Azure and GCP
- ✅ Deploying Across Three Clouds and Validating with AI
- ✅ Cross-Cloud VPN Between AWS and Azure
