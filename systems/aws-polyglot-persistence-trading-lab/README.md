# AWS Database Modernization Lab

> Inside the [Cloud Systems Engineering](../../README.md) portfolio · *Cloud platforms engineered for scale, reliability, and uptime.*

## Overview

This lab evaluated modern AWS database services under a simulated tenfold increase in trading activity to determine the most appropriate architecture for different application workloads. The implementation compared Amazon Aurora Serverless v2, Amazon DynamoDB with DAX, and Amazon MemoryDB by migrating 50,000 trade records, executing k6 performance tests, and measuring latency, throughput, scalability, and operational cost. The objective was not to identify a single database for every workload, but to understand where each service delivered the greatest value while maintaining cost efficiency. The final deliverable was an executive HTML report recommending a modern AWS database architecture that raised application responsiveness, reduced operational overhead, and remained within AWS Free Tier limits throughout development.

The architecture is built across **9 phases**, anchored by **Taking On the Assignment: Day 1 at Meridian Energy Trading** on the input side and **Mission Accomplished: Lessons from the Lab** at the end. Each phase is listed in the Implementation section below.

## Architecture

```mermaid
---
title: AWS Polyglot Persistence Trading Lab (Aurora vs DynamoDB vs MemoryDB under 10x Spike)
---
%%{init: {"theme":"base","themeVariables": {"primaryColor":"#1B4332","primaryTextColor":"#F4D03F","primaryBorderColor":"#F4D03F","secondaryColor":"#264653","tertiaryColor":"#2F5233","lineColor":"#F4D03F","fontFamily":"ui-monospace, SFMono-Regular, Menlo, Consolas, monospace","fontSize":"13px"}}}%%
flowchart LR
    classDef datastore fill:#264653,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef service fill:#1B4332,stroke:#F4D03F,stroke-width:2px,color:#F4D03F
    classDef event fill:#7B42BC,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef io fill:#0d1117,stroke:#F4D03F,stroke-width:1.5px,color:#F4D03F,font-style:italic

    Architect[/"Principal SA (operator)"/]
    CTO[/"CTO (executive sponsor)"/]
    OPECSpike[/"Simulated OPEC announcement: 10x trading surge"/]

    subgraph RequirementPackage["Architecture-First Requirement Package"]
        ADRs[(ADRs.md)]
        ArchDiagrams[(architecture-diagrams.md)]
        ImplPlan[(implementation-plan.md)]
        RiskRegister[(risk-register.md)]
        RACI[(raci.md)]
        SAPC02[(sap-c02-alignment.md)]
        ADR001[(ADR-001: DynamoDB + DAX highest priority, in transaction path)]
    end

    subgraph Workstation["Principal SA Workstation"]
        Terraform("Terraform 1.15.7")
        K6("k6 2.0.0")
        AWSCLI("AWS CLI 2.34.32 (meridian-prod profile)")
        CursorWorkspace("Cursor workspace: 4 parallel agents")
    end

    subgraph ModularTerraform["Modular Terraform (parallel-agent owned)"]
        NetworkingModule("networking module")
        AuroraModule("aurora module")
        DynamoModule("dynamodb module")
        MemoryDBModule("memorydb module")
        FourAgents{{"4 Cursor agents generate modules simultaneously"}}
        FiftyTwoResources[(52 AWS resources provisioned)]
    end

    subgraph SourceData["Migration Source + DMS"]
        PostgresEC2[(PostgreSQL on EC2: 50K trade records)]
        DMS("AWS DMS full-load")
        DMSEndpoints("DMS source + target endpoints")
    end

    subgraph AuroraEngine["Aurora Serverless v2"]
        AuroraCluster("Aurora Serverless v2")
        RDSProxy("RDS Proxy")
        IOOptimized{{"I/O-Optimized storage: break-even when I/O dominates cost"}}
        Babelfish("Babelfish TDS endpoint: T-SQL compatibility")
        AuroraResult[/"Zero errors, higher SQL latency, complex-query strength"/]
    end

    subgraph DynamoEngine["DynamoDB + DAX"]
        DynamoTable[(DynamoDB table: 50K records verified)]
        DAX("DAX cache")
        GSI("Global Secondary Index queries")
        DynamoResult[/"64 req/s peak, zero errors, stable latency"/]
    end

    subgraph MemoryEngine["MemoryDB"]
        MemoryDBCluster("MemoryDB cluster")
        MemoryResult[/"5ms p50, 39ms p99 SET/GET"/]
    end

    subgraph Validation["Cross-Engine Validation (AWS CLI only)"]
        CLICount("AWS CLI count operations")
        DynamoVerified{{"DynamoDB: 50K records confirmed"}}
        VPCBlocked{{"Aurora + MemoryDB VPC-only; helper EC2 cloud-init failed"}}
        DMSZeroRows{{"DMS reported 0 rows into Aurora (known gap)"}}
    end

    subgraph Benchmark["10x Spike Benchmark"]
        K6Scripts("k6 load-test scripts")
        CloudWatch("CloudWatch: latency + throughput + utilization + errors")
        CostProjection[(Cost projection at 500K reads/min)]
    end

    subgraph ExecReport["Executive Report to CTO"]
        HTMLReport[(HTML report + benchmark charts)]
        DecisionMatrix[(Polyglot decision matrix)]
        TerraformDestroy{{"Terraform destroy: free-tier cost discipline"}}
    end

    subgraph PolyglotRecommendation["Polyglot Persistence Recommendation"]
        DynamoForLookups("DynamoDB + DAX: latency-sensitive trade lookups")
        AuroraForAnalytics("Aurora Serverless v2 + RDS Proxy: relational analytics")
        MemoryForPrices("MemoryDB: real-time durable price distribution")
    end

    subgraph SecretMission["Secret Mission: Revenue API"]
        RevenueAPI("Lightweight JSON API on MemoryDB")
        PriceFeed("Live commodity price feed")
        SixHundredK[/"$600K/yr external partner revenue product"/]
    end

    Architect --> RequirementPackage
    ADRs --> ADR001
    Architect --> Workstation
    Workstation --> ModularTerraform
    CursorWorkspace --> FourAgents
    FourAgents --> NetworkingModule
    FourAgents --> AuroraModule
    FourAgents --> DynamoModule
    FourAgents --> MemoryDBModule
    ADR001 -.guides.-> DynamoModule
    ModularTerraform --> FiftyTwoResources

    AuroraModule --> AuroraCluster
    DynamoModule --> DynamoTable
    MemoryDBModule --> MemoryDBCluster
    AuroraCluster --> RDSProxy
    AuroraCluster --> IOOptimized
    AuroraCluster --> Babelfish
    DynamoTable --> DAX
    DynamoTable --> GSI

    PostgresEC2 --> DMS
    DMS --> DMSEndpoints
    DMS --> AuroraCluster
    PostgresEC2 -.seeded into.-> DynamoTable
    PostgresEC2 -.seeded into.-> MemoryDBCluster

    CLICount --> DynamoVerified
    DynamoVerified --> DynamoTable
    VPCBlocked -.blocks.-> AuroraCluster
    VPCBlocked -.blocks.-> MemoryDBCluster
    DMSZeroRows -.affects.-> AuroraCluster

    OPECSpike --> K6Scripts
    K6Scripts --> AuroraCluster
    K6Scripts --> DynamoTable
    K6Scripts --> MemoryDBCluster
    CloudWatch -.monitors.-> K6Scripts
    K6Scripts --> CostProjection
    AuroraCluster --> AuroraResult
    DynamoTable --> DynamoResult
    MemoryDBCluster --> MemoryResult

    AuroraResult --> HTMLReport
    DynamoResult --> HTMLReport
    MemoryResult --> HTMLReport
    CostProjection --> HTMLReport
    HTMLReport --> DecisionMatrix
    DecisionMatrix --> CTO
    HTMLReport --> TerraformDestroy

    DecisionMatrix --> DynamoForLookups
    DecisionMatrix --> AuroraForAnalytics
    DecisionMatrix --> MemoryForPrices

    MemoryDBCluster --> RevenueAPI
    RevenueAPI --> PriceFeed
    PriceFeed --> SixHundredK

    class Terraform,K6,AWSCLI,CursorWorkspace service
    class NetworkingModule,AuroraModule,DynamoModule,MemoryDBModule service
    class AuroraCluster,RDSProxy,Babelfish,DAX,GSI,MemoryDBCluster service
    class DMS,DMSEndpoints,CLICount,K6Scripts,CloudWatch,RevenueAPI,PriceFeed service
    class DynamoForLookups,AuroraForAnalytics,MemoryForPrices service
    class ADRs,ArchDiagrams,ImplPlan,RiskRegister,RACI,SAPC02,ADR001,FiftyTwoResources datastore
    class PostgresEC2,DynamoTable,CostProjection,HTMLReport,DecisionMatrix datastore
    class FourAgents,IOOptimized,DynamoVerified,VPCBlocked,DMSZeroRows,TerraformDestroy event
    class Architect,CTO,OPECSpike,AuroraResult,DynamoResult,MemoryResult,SixHundredK io
```

The diagram shows the topology and data flow of the system as built. The full architectural narrative, with screenshots and prose, lives in [`documents/aws-polyglot-persistence-trading-lab.md`](./documents/aws-polyglot-persistence-trading-lab.md).

## Implementation

This system is built across **9 phases**:

1. **Taking On the Assignment: Day 1 at Meridian Energy Trading**
2. **Architecting the Solution: The Requirement Package**
3. **Setting Up the Principal SA Workstation**
4. **Deploying Three Database Engines with Parallel AI Agents**
5. **Migrating Data and Verifying All Three Engines**
6. **Simulating an OPEC Announcement: The 10x Spike Benchmark**
7. **Delivering the Executive Report to the CTO**
8. **I/O-Optimized, Babelfish, and the Revenue API**
9. **Mission Accomplished: Lessons from the Lab**

For the full walkthrough with screenshots and step-by-step content, see [`documents/aws-polyglot-persistence-trading-lab.md`](./documents/aws-polyglot-persistence-trading-lab.md).

## Validation

Build outcomes verified end-to-end. Each phase below is captured with screenshots, configuration, and observable behavior in [`documents/aws-polyglot-persistence-trading-lab.md`](./documents/aws-polyglot-persistence-trading-lab.md):

- ✅ Taking On the Assignment: Day 1 at Meridian Energy Trading
- ✅ Architecting the Solution: The Requirement Package
- ✅ Setting Up the Principal SA Workstation
- ✅ Deploying Three Database Engines with Parallel AI Agents
- ✅ Migrating Data and Verifying All Three Engines
- ✅ Simulating an OPEC Announcement: The 10x Spike Benchmark
- ✅ Delivering the Executive Report to the CTO
- ✅ I/O-Optimized, Babelfish, and the Revenue API
- ✅ Mission Accomplished: Lessons from the Lab
