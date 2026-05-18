# AWS FinOps Command Center

> Inside the [Cloud Systems Engineering](../../README.md) portfolio · *Cloud platforms engineered for scale, reliability, and uptime.*

## Overview

In this project, I built an enterprise FinOps command center designed to improve cloud cost visibility, automate governance actions, and provide leadership-ready financial reporting for a Fortune 10 scale environment.

The architecture combined CUR 2.0 exports, Glue, Athena, Grafana, Cost Anomaly Detection, Lambda remediation workflows, Budget Actions, Organizations Tag Policies, and executive dashboards into a unified FinOps operating model. The objective was to move from reactive billing analysis toward automated cost intelligence and operational governance.

The architecture is built across **9 phases**, anchored by **Building an Enterprise FinOps Command Center** on the input side and **Carbon Footprint Overlay and Automated Weekly FinOps Cadence** at the end. Each phase is listed in the Implementation section below.

## Architecture

```mermaid
---
title: AWS FinOps Command Center
---
%%{init: {"theme":"base","themeVariables": {"primaryColor":"#1B4332","primaryTextColor":"#F4D03F","primaryBorderColor":"#F4D03F","secondaryColor":"#264653","tertiaryColor":"#2F5233","lineColor":"#F4D03F","fontFamily":"ui-monospace, SFMono-Regular, Menlo, Consolas, monospace","fontSize":"13px"}}}%%
flowchart LR
    classDef datastore fill:#264653,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef service fill:#1B4332,stroke:#F4D03F,stroke-width:2px,color:#F4D03F
    classDef event fill:#7B42BC,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef io fill:#0d1117,stroke:#F4D03F,stroke-width:1.5px,color:#F4D03F,font-style:italic

    subgraph Toolchain["FinOps Toolchain"]
        Terraform(Terraform Root)
        AWSCLI(AWS CLI)
        Docker(Docker Desktop)
        GrafanaOSS(Grafana OSS)
        AthenaPlugin(Athena Datasource Plugin)
    end

    subgraph Org["AWS Organizations Management"]
        MgmtAcct(Management Account)
        AllFeatures(ALL Feature Set)
        SCPs(Service Control Policies)
        TagPolicies(Tag Policies: CostCenter, Environment, Owner)
        CostAllocTags(Cost Allocation Tags)
    end

    subgraph PreArtifacts["Architecture Pre-Artifacts"]
        ADRs[(ADRs: Grafana vs QuickSight, CUR 2.0 vs Cost Explorer, Tag Policies vs manual)]
        ArchDiagrams[(Mermaid Architecture Diagrams)]
        RiskRegister[(Risk Register + FinOps Cadence)]
    end

    subgraph DataPipeline["CUR 2.0 to Athena Pipeline"]
        CUR(CUR 2.0 Export us-east-1)
        S3CUR[(S3 CUR Parquet Storage)]
        GlueCatalog(AWS Glue Catalog)
        GlueCrawler(Glue Crawlers)
        AthenaWG(Athena Workgroups)
        ChargebackView[(Chargeback Analysis Views)]
    end

    subgraph Dashboards["Grafana Executive Dashboards"]
        ChargebackDash(Chargeback Dashboard)
        ExecKPI(Executive KPI Dashboard)
        Forecast12(12-Month Cost Projection)
        TemplateVars(Template Variables: cost-center, account, window)
    end

    subgraph Anomaly["Anomaly Detection + Remediation Loop"]
        CAD(Cost Anomaly Detection Monitors)
        SNSAnomaly(SNS Anomaly Topic)
        EBRule(EventBridge Rule)
        Remediator(Lambda Remediator)
        DDBLog[(DynamoDB Remediation Log)]
        DashAnnot(Grafana Dashboard Annotations)
    end

    subgraph DualPattern["Dual Remediation Pattern"]
        EnvTag{{Environment Tag Check}}
        AutoStop(STOP_NON_PROD_EC2: Auto-Stop Non-Prod)
        ManualApprove{{Production: Manual Approval Gate}}
    end

    subgraph Guardrails["Layered Budget Guardrails"]
        BudgetActions(AWS Budgets: Static Forecast Threshold)
        IAMRestrict(IAM Restriction Auto-Attach)
    end

    subgraph Validation["End-to-End Spike Test"]
        SpikeInstance[/T3.Micro Spike Instance/]
        SyntheticEvent{{Synthetic Anomaly Payload}}
        DirectInvoke(Direct Lambda Invoke)
        StateValidation(Stopped Instance + DDB Log + Annotation Verified)
    end

    subgraph WeeklyCadence["Secret Mission: Automated Weekly Digest"]
        EBScheduler(EventBridge Scheduler)
        DigestLambda(Weekly Digest Lambda)
        AthenaQuery(Athena KPI Queries)
        SNSDigest(SNS Weekly Digest)
        KPISet[(KPIs: Spend, Tag Coverage, Savings Plans, Rightsizing, Anomaly Count)]
    end

    subgraph Executive["CFO Executive Pitch"]
        PitchDeck(Pitch Deck)
        WellArch[(Well-Architected Cost Pillar COST01-COST11)]
        TechDebt[(Tech-Debt Scorecard)]
        Roadmap[(Optimization Roadmap)]
    end

    Terraform --> MgmtAcct
    Terraform --> CUR
    Terraform --> CAD
    Terraform --> EBScheduler
    Docker --> GrafanaOSS
    GrafanaOSS --> AthenaPlugin

    MgmtAcct --> AllFeatures
    AllFeatures --> SCPs
    AllFeatures --> TagPolicies
    TagPolicies -.enforces at creation.-> CostAllocTags
    CostAllocTags -.surfaces in.-> CUR

    ADRs --> ArchDiagrams
    ArchDiagrams --> RiskRegister

    CUR --> S3CUR
    S3CUR --> GlueCatalog
    GlueCrawler --> GlueCatalog
    GlueCatalog --> AthenaWG
    AthenaWG --> ChargebackView

    AthenaPlugin --> AthenaWG
    ChargebackView --> ChargebackDash
    ChargebackView --> ExecKPI
    ChargebackView --> Forecast12
    TemplateVars -.scopes queries.-> ChargebackDash

    CAD --> SNSAnomaly
    SNSAnomaly --> EBRule
    EBRule --> Remediator
    Remediator --> EnvTag
    EnvTag -.non-prod.-> AutoStop
    EnvTag -.production.-> ManualApprove
    Remediator --> DDBLog
    Remediator --> DashAnnot
    DashAnnot --> ChargebackDash

    BudgetActions --> IAMRestrict
    BudgetActions -.static backstop.-> CAD
    Remediator -.dynamic responder.-> AutoStop

    SyntheticEvent --> DirectInvoke
    DirectInvoke --> Remediator
    SpikeInstance -.stopped by.-> AutoStop
    StateValidation -.verified post-run.-> DDBLog

    EBScheduler --> DigestLambda
    DigestLambda --> AthenaQuery
    AthenaQuery --> ChargebackView
    DigestLambda --> KPISet
    KPISet --> SNSDigest

    PitchDeck --> WellArch
    PitchDeck --> TechDebt
    PitchDeck --> Roadmap
    ChargebackDash -.live data.-> PitchDeck

    class Terraform,AWSCLI,Docker io
    class GrafanaOSS,AthenaPlugin,MgmtAcct,AllFeatures,SCPs,TagPolicies,CostAllocTags service
    class CUR,GlueCatalog,GlueCrawler,AthenaWG,ChargebackDash,ExecKPI,Forecast12,TemplateVars service
    class CAD,SNSAnomaly,Remediator,EnvTag,AutoStop,BudgetActions,IAMRestrict service
    class EBScheduler,DigestLambda,AthenaQuery,SNSDigest,PitchDeck service
    class ADRs,ArchDiagrams,RiskRegister,S3CUR,ChargebackView,DDBLog,DashAnnot,WellArch,TechDebt,Roadmap,KPISet datastore
    class SpikeInstance io
    class EBRule,ManualApprove,SyntheticEvent,DirectInvoke,StateValidation event
```

The diagram shows the topology and data flow of the system as built. The full architectural narrative, with screenshots and prose, lives in [`documents/aws-finops-command-center.md`](./documents/aws-finops-command-center.md).

## Implementation

This system is built across **9 phases**:

1. **Building an Enterprise FinOps Command Center**
2. **Setting Up the FinOps Toolchain**
3. **Architecting with Pre-Artifacts: ADRs, Diagrams, and Governance Plans**
4. **Deploying the CUR 2.0 → Glue → Athena Data Pipeline**
5. **Building the Grafana Executive Dashboard**
6. **Automating Anomaly Detection and Remediation**
7. **Validating the End-to-End Pipeline with a Spike Test**
8. **Delivering the CFO Executive Pitch Deck**
9. **Carbon Footprint Overlay and Automated Weekly FinOps Cadence**

For the full walkthrough with screenshots and step-by-step content, see [`documents/aws-finops-command-center.md`](./documents/aws-finops-command-center.md).

## Validation

Build outcomes verified end-to-end. Each phase below is captured with screenshots, configuration, and observable behavior in [`documents/aws-finops-command-center.md`](./documents/aws-finops-command-center.md):

- ✅ Building an Enterprise FinOps Command Center
- ✅ Setting Up the FinOps Toolchain
- ✅ Architecting with Pre-Artifacts: ADRs, Diagrams, and Governance Plans
- ✅ Deploying the CUR 2.0 → Glue → Athena Data Pipeline
- ✅ Building the Grafana Executive Dashboard
- ✅ Automating Anomaly Detection and Remediation
- ✅ Validating the End-to-End Pipeline with a Spike Test
- ✅ Delivering the CFO Executive Pitch Deck
- ✅ Carbon Footprint Overlay and Automated Weekly FinOps Cadence
