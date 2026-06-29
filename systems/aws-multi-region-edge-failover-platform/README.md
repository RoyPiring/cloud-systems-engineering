# AWS Multi-Region Edge Failover Sprint

> Inside the [Cloud Systems Engineering](../../README.md) portfolio · *Cloud platforms engineered for scale, reliability, and uptime.*

## Overview

This project focused on building a resilient multi-region edge architecture capable of maintaining service availability during regional failures. The environment was prepared by validating Terraform 1.15.3 and AWS CLI v2, establishing the project structure, initializing Terraform state, and coordinating work across AI-assisted contributors through Cursor Composer. These foundational steps ensured that deployment activities could proceed with a consistent and repeatable infrastructure workflow.

The objective was not only to deploy resources, but to prove operational recovery, infrastructure portability, and controlled failover through infrastructure as code. Establishing a clean deployment baseline reduced configuration drift risk and enabled reliable validation throughout the sprint.

The architecture is built across **7 phases**, anchored by **Architecting a Crisis Response: The $0.61 Sprint** on the input side and **WAF Edge Security Without Touching Application Code** at the end. Each phase is listed in the Implementation section below.

## Architecture

```mermaid
---
title: AWS Multi-Region Edge Failover Platform (Sub-Second Measured Recovery)
---
%%{init: {"theme":"base","themeVariables": {"primaryColor":"#1B4332","primaryTextColor":"#F4D03F","primaryBorderColor":"#F4D03F","secondaryColor":"#264653","tertiaryColor":"#2F5233","lineColor":"#F4D03F","fontFamily":"ui-monospace, SFMono-Regular, Menlo, Consolas, monospace","fontSize":"13px"}}}%%
flowchart LR
    classDef datastore fill:#264653,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef service fill:#1B4332,stroke:#F4D03F,stroke-width:2px,color:#F4D03F
    classDef event fill:#7B42BC,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef io fill:#0d1117,stroke:#F4D03F,stroke-width:1.5px,color:#F4D03F,font-style:italic

    Viewer[/"Global viewer"/]
    BankPartner[/"Regulated banking partner (firewall allowlist)"/]
    Architect[/"Architect (operator)"/]
    SLA[/"Measured 1.1s failover vs 60s target"/]

    subgraph Toolchain["Verified Local Toolchain"]
        Terraform("Terraform 1.15.3")
        AWSCLI("AWS CLI v2")
        CursorComposer("Cursor Composer AI contributors")
        PowerShell("PowerShell + Bash validators")
    end

    subgraph TerraformState["Single Terraform State, Two Providers"]
        ProviderUSE1("aws.use1 alias: us-east-1")
        ProviderEUW1("aws.euw1 alias: eu-west-1")
        StateFile[(Terraform state file)]
        DestroyPlan{{"terraform destroy plan: zero orphaned resources"}}
    end

    subgraph PrimaryRegion["Primary: us-east-1"]
        LambdaUSE1("Lambda origin (Function URL)")
        FunctionURLUSE1[(Public Function URL)]
        NLBUSE1("Network Load Balancer (GA endpoint type)")
    end

    subgraph SecondaryRegion["Secondary: eu-west-1"]
        LambdaEUW1("Lambda origin (Function URL)")
        FunctionURLEUW1[(Public Function URL)]
        NLBEUW1("Network Load Balancer (GA endpoint type)")
    end

    subgraph Route53Layer["Route 53 DNS Layer + Health Checks"]
        HealthChecks("Route 53 health checks")
        FailoverPolicy("Failover routing: primary + standby")
        MultivaluePolicy("Multivalue: concurrent healthy answers")
        GeoproximityPolicy("Geoproximity routing")
        LatencyPolicy("Latency routing")
        WeightedPolicy("Weighted routing")
        SAPC02Trap{{"SAP-C02 trap: Failover != Multivalue"}}
    end

    subgraph CloudFrontEdge["CloudFront Distribution"]
        CFDistribution("CloudFront distribution")
        OriginGroup{{"Origin group: us-east-1 primary -> eu-west-1 secondary"}}
        CFFunction("CloudFront Function: viewer-request strip legacy headers")
        LambdaEdge("Lambda@Edge: origin-request signed URL validation")
        CFOriginFailover{{"Origin failover independent of DNS cache"}}
    end

    subgraph WAFLayer["WAF Edge Security"]
        WebACL[(WAF WebACL)]
        GeoFilter("Geographic filtering")
        RateLimit("Rate limiting")
        BotMitigation("Bot mitigation")
        AssociationGotcha{{"WebACL alone is insufficient; must associate with distribution"}}
    end

    subgraph GlobalAccelerator["Global Accelerator: Anycast IPs"]
        GAAccelerator("Global Accelerator")
        AnycastIP1[/"Static Anycast IPv4 #1"/]
        AnycastIP2[/"Static Anycast IPv4 #2"/]
        PCIDSSAllowlist{{"PCI-DSS partner firewall allowlist"}}
    end

    subgraph LeadershipPackage["Leadership Package"]
        ADRs[(ADRs)]
        MermaidTopology[(Mermaid topology diagrams)]
        FinOpsReport[(FinOps cost analysis: $0.61 sprint)]
        ExecHTML[(Single-file HTML executive presentation)]
        PSValidator("PowerShell failover validator")
    end

    subgraph KillPrimaryTest["End-to-End Kill-Primary Test"]
        CacheBust("Cache-busting requests")
        SimulateOutage{{"Simulate primary-region loss"}}
        Measure1Point1s[/"Measured 1.1s failover"/]
        OrphanedCheck{{"Cross-check deployed vs Terraform-managed"}}
        ZeroOrphans[/"Zero orphaned resources"/]
    end

    subgraph SecretMission["Secret Mission: D2.2 Edge Security"]
        EdgeOnlyControls{{"Security at the edge, no app code change"}}
        TerraformWAF("WAF defined as Terraform")
        VersionControlled{{"Version-controlled + auditable + removable"}}
    end

    Architect --> Toolchain
    Terraform --> TerraformState
    ProviderUSE1 --> PrimaryRegion
    ProviderEUW1 --> SecondaryRegion
    StateFile -.tracks.-> ProviderUSE1
    StateFile -.tracks.-> ProviderEUW1

    LambdaUSE1 --> FunctionURLUSE1
    LambdaUSE1 --> NLBUSE1
    LambdaEUW1 --> FunctionURLEUW1
    LambdaEUW1 --> NLBEUW1

    Viewer --> Route53Layer
    HealthChecks -.gates.-> FailoverPolicy
    HealthChecks -.gates.-> MultivaluePolicy
    SAPC02Trap -.calls out.-> FailoverPolicy
    SAPC02Trap -.calls out.-> MultivaluePolicy
    Route53Layer --> CFDistribution

    CFDistribution --> OriginGroup
    OriginGroup -.primary.-> FunctionURLUSE1
    OriginGroup -.secondary.-> FunctionURLEUW1
    CFFunction -.viewer-request stage.-> CFDistribution
    LambdaEdge -.origin-request stage.-> CFDistribution
    CFOriginFailover -.before DNS expiry.-> OriginGroup

    WebACL --> GeoFilter
    WebACL --> RateLimit
    WebACL --> BotMitigation
    WebACL --> CFDistribution
    AssociationGotcha -.must associate.-> WebACL

    BankPartner --> AnycastIP1
    BankPartner --> AnycastIP2
    AnycastIP1 --> GAAccelerator
    AnycastIP2 --> GAAccelerator
    GAAccelerator --> NLBUSE1
    GAAccelerator --> NLBEUW1
    PCIDSSAllowlist -.requires.-> AnycastIP1
    PCIDSSAllowlist -.requires.-> AnycastIP2

    PSValidator --> SimulateOutage
    SimulateOutage --> CacheBust
    CacheBust --> CFDistribution
    SimulateOutage --> Measure1Point1s
    Measure1Point1s --> SLA
    Terraform --> DestroyPlan
    DestroyPlan --> OrphanedCheck
    OrphanedCheck --> ZeroOrphans

    ADRs --> MermaidTopology
    ADRs --> FinOpsReport
    MermaidTopology --> ExecHTML
    ExecHTML -.audience views.-> ExecHTML

    TerraformWAF -.defines.-> WebACL
    EdgeOnlyControls -.principle.-> WebACL
    EdgeOnlyControls -.principle.-> CFFunction
    EdgeOnlyControls -.principle.-> LambdaEdge
    VersionControlled -.via Terraform.-> TerraformWAF
    class Terraform,AWSCLI,CursorComposer,PowerShell service
    class ProviderUSE1,ProviderEUW1 service
    class LambdaUSE1,LambdaEUW1,NLBUSE1,NLBEUW1 service
    class HealthChecks,FailoverPolicy,MultivaluePolicy,GeoproximityPolicy,LatencyPolicy,WeightedPolicy service
    class CFDistribution,CFFunction,LambdaEdge service
    class GeoFilter,RateLimit,BotMitigation service
    class GAAccelerator service
    class PSValidator,CacheBust,TerraformWAF service
    class StateFile,FunctionURLUSE1,FunctionURLEUW1,WebACL,ADRs,MermaidTopology,FinOpsReport,ExecHTML datastore
    class DestroyPlan,SAPC02Trap,OriginGroup,CFOriginFailover,AssociationGotcha,PCIDSSAllowlist event
    class SimulateOutage,OrphanedCheck,EdgeOnlyControls,VersionControlled event
    class Viewer,BankPartner,Architect,SLA,AnycastIP1,AnycastIP2,Measure1Point1s,ZeroOrphans io
```

The diagram shows the topology and data flow of the system as built. The full architectural narrative, with screenshots and prose, lives in [`documents/aws-multi-region-edge-failover-platform.md`](./documents/aws-multi-region-edge-failover-platform.md).

## Implementation

This system is built across **7 phases**:

1. **Architecting a Crisis Response: The $0.61 Sprint**
2. **Building Two-Region Origins and Automated DNS Failover**
3. **Deploying CloudFront with Sub-5-Second Origin Failover and Edge Compute**
4. **Providing Static IPs for Regulated Banking Partners with Global Accelerator and WAF**
5. **Delivering the Leadership Package: ADRs, FinOps, and the Director Presentation**
6. **Proving Sub-60-Second Failover and Zero Orphaned Resources**
7. **WAF Edge Security Without Touching Application Code**

For the full walkthrough with screenshots and step-by-step content, see [`documents/aws-multi-region-edge-failover-platform.md`](./documents/aws-multi-region-edge-failover-platform.md).

## Validation

Each build phase below is documented in [`documents/aws-multi-region-edge-failover-platform.md`](./documents/aws-multi-region-edge-failover-platform.md), with screenshots, configuration, and notes as captured during the build:

- ✅ Architecting a Crisis Response: The $0.61 Sprint
- ✅ Building Two-Region Origins and Automated DNS Failover
- ✅ Deploying CloudFront with Sub-5-Second Origin Failover and Edge Compute
- ✅ Providing Static IPs for Regulated Banking Partners with Global Accelerator and WAF
- ✅ Delivering the Leadership Package: ADRs, FinOps, and the Director Presentation
- ✅ Proving Sub-60-Second Failover and Zero Orphaned Resources
- ✅ WAF Edge Security Without Touching Application Code
