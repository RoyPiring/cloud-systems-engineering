# AWS Principal SA: Fix the $2.1M Pipeline

> Inside the [Cloud Systems Engineering](../../README.md) portfolio · *Cloud platforms engineered for scale, reliability, and uptime.*

## Overview

In this build, I built a Terraform-managed AWS platform for GenoVault to address a $2.1M migration pipeline problem. The system was designed to remove orphan resources, enforce strict Infrastructure as Code governance, and show how migration pipelines can lower cost while still meeting migration and compliance needs.

The main decision was to make Terraform the control plane for the environment. Resources were not created through one-off manual actions. They were declared in code, tracked in state, tagged for visibility, and tied to a clean teardown path.

This mattered because migration environments can become expensive when resources stay alive after their purpose is complete. The build proved that cost control, governance, migration design, and teardown discipline had to work as one system.

The architecture is built across **9 phases**, anchored by **The $2.1M Problem: Why This Project Exists** on the input side and **NIH-Grade Compliance Vault with Object Lock** at the end. Each phase is listed in the Implementation section below.

## Architecture

```mermaid
---
title: AWS Zero-Orphan Migration Platform - Terraform Governance
---
%%{init: {"theme":"base","themeVariables": {"primaryColor":"#1B4332","primaryTextColor":"#F4D03F","primaryBorderColor":"#F4D03F","secondaryColor":"#264653","tertiaryColor":"#2F5233","lineColor":"#F4D03F","fontFamily":"ui-monospace, SFMono-Regular, Menlo, Consolas, monospace","fontSize":"13px"}}}%%
flowchart LR
    classDef datastore fill:#264653,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef service fill:#1B4332,stroke:#F4D03F,stroke-width:2px,color:#F4D03F
    classDef event fill:#7B42BC,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef io fill:#0d1117,stroke:#F4D03F,stroke-width:1.5px,color:#F4D03F,font-style:italic

    LegacyData[/GenoVault 80TB NFS + SQL Server source/]

    subgraph Toolchain["Local Toolchain - version-pinned"]
        Terraform(Terraform 1.15.3)
        AwsCli(AWS CLI v2 - read-only)
        Python(Python 3.12)
        NodeJs(Node.js LTS)
        Gpg(Gpg4win - PGP)
    end

    subgraph ControlPlane["Terraform Control Plane"]
        RootModule(Root Module)
        State[(Terraform State)]
        Tags[(Tagging Governance)]
        NetModule(network module)
        StorageModule(storage module)
        IamModule(IAM module - least privilege)
        ServiceModule(service module)
    end

    subgraph Network["VPC - Cost-Aware Network"]
        PublicSubnet(Public Subnet)
        PrivateSubnet(Private Subnet)
        Igw(Internet Gateway)
        S3Endpoint(S3 Gateway Endpoint - free)
        SecGroups(Security Groups)
    end
    NatDecision{{S3 Gateway Endpoint over NAT - saves 0.045/hr}}

    subgraph Storage["Storage Layer"]
        S3[(S3 buckets)]
        Efs[(EFS)]
        Fsx[(FSx)]
        Ebs[(EBS)]
    end

    subgraph Router["Workload Router + Storage Intelligence"]
        Profiles[(5 synthetic genomic profiles)]
        LambdaRouter(Lambda Workload Router)
        StorageLens(S3 Storage Lens)
        DeepArchive[(Glacier Deep Archive - 10yr, 12h SLA)]
        FlexRetrieval[(Glacier Flexible Retrieval)]
    end
    StorageDecision{{Deep Archive over Flexible - match SLA to access}}

    subgraph DataSyncPipe["DataSync Migration"]
        MockNfs(Mock NFS EC2)
        DataSync(AWS DataSync)
        RawPrefix[(S3 raw/ prefix)]
    end
    SnowballDecision{{DataSync over Snowball - 1Gbps, 8-day, 1000 dollars}}

    subgraph DmsPipe["DMS Database Migration"]
        SqlServer[(RDS SQL Server - source)]
        Postgres[(RDS PostgreSQL - target)]
        Secrets[(Secrets Manager)]
        SchemaConv(Schema Conversion)
        DmsTask(DMS full-load + CDC)
    end

    subgraph TransferFam["Transfer Family - deploy-test-destroy"]
        Sftp(Transfer Family SFTP)
        PgpLambda(Lambda PGP decrypt)
        TfFlag{{transfer_family_enabled flag}}
    end

    subgraph Governance["Governance + Teardown"]
        Gate4{{Gate 4 - tag query returns empty}}
        Destroy(terraform destroy - 95 resources)
        ZeroOrphans{{Zero orphans - state empty}}
    end

    subgraph Compliance["Secret Mission - Compliance Vault"]
        ComplianceWs(terraform-compliance workspace)
        ObjectLock[(S3 Object Lock - COMPLIANCE 10yr)]
    end

    subgraph Deliverables["Deliverables"]
        React[/React presentation/]
        Briefing[/9-document briefing/]
    end

    Terraform -->|declares all infra| RootModule
    AwsCli -.->|read-only checks| RootModule
    Gpg -->|PGP keys for| PgpLambda
    RootModule -->|calls| NetModule
    RootModule -->|calls| StorageModule
    RootModule -->|calls| IamModule
    RootModule -->|calls| ServiceModule
    RootModule -->|tracks in| State
    RootModule -->|applies| Tags

    NetModule -->|creates| PublicSubnet
    NetModule -->|creates| PrivateSubnet
    NetModule -->|creates| Igw
    NetModule -->|creates| S3Endpoint
    NetModule -->|creates| SecGroups
    S3Endpoint -.->|chosen for cost| NatDecision
    PrivateSubnet -->|private path to| S3Endpoint
    S3Endpoint -->|keeps traffic on AWS| S3

    StorageModule -->|defines| S3
    StorageModule -->|defines| Efs
    StorageModule -->|defines| Fsx
    StorageModule -->|defines| Ebs
    IamModule -->|least-privilege roles| StorageModule

    ServiceModule -->|deploys| LambdaRouter
    Profiles -->|feed| LambdaRouter
    LambdaRouter -->|routes by retention| DeepArchive
    LambdaRouter -->|routes by access| FlexRetrieval
    LambdaRouter -->|hot data| S3
    DeepArchive -.->|SLA match| StorageDecision
    StorageLens -->|validates placement| LambdaRouter

    LegacyData -->|80TB file source| MockNfs
    ServiceModule -->|deploys| MockNfs
    MockNfs -->|NFS mount| DataSync
    DataSync -->|managed transfer| RawPrefix
    RawPrefix -->|lands in| S3
    DataSync -.->|chosen over Snowball| SnowballDecision

    LegacyData -->|SQL Server schema| SchemaConv
    ServiceModule -->|deploys| SqlServer
    ServiceModule -->|deploys| Postgres
    Secrets -->|credentials for| DmsTask
    SchemaConv -->|PostgreSQL DDL| Postgres
    SqlServer -->|source rows| DmsTask
    DmsTask -->|full-load + CDC| Postgres

    ServiceModule -->|deploys when flagged| Sftp
    TfFlag -->|gates| Sftp
    Sftp -->|encrypted upload| PgpLambda
    PgpLambda -->|decrypted object| S3
    TfFlag -.->|false after test| Destroy

    Tags -->|queried by| Gate4
    State -->|empty after| Destroy
    Destroy -->|removes 95 resources| ZeroOrphans
    Gate4 -->|confirms| ZeroOrphans

    RootModule -.->|separate workspace| ComplianceWs
    ComplianceWs -->|isolated from teardown| ObjectLock
    ObjectLock -.->|survives| Destroy

    ZeroOrphans -->|proven in| React
    React -->|expands into| Briefing

    class State,Tags,S3,Efs,Fsx,Ebs,Profiles,DeepArchive,FlexRetrieval,RawPrefix,SqlServer,Postgres,Secrets,ObjectLock datastore
    class Terraform,AwsCli,Python,NodeJs,Gpg,RootModule,NetModule,StorageModule,IamModule,ServiceModule,PublicSubnet,PrivateSubnet,Igw,S3Endpoint,SecGroups,LambdaRouter,StorageLens,MockNfs,DataSync,SchemaConv,DmsTask,Sftp,PgpLambda,Destroy,ComplianceWs service
    class NatDecision,StorageDecision,SnowballDecision,TfFlag,Gate4,ZeroOrphans event
    class LegacyData,React,Briefing io
```

The diagram shows the topology and data flow of the system as built. The full architectural narrative, with screenshots and prose, lives in [`documents/aws-zero-orphan-migration-platform.md`](./documents/aws-zero-orphan-migration-platform.md).

## Implementation

This system is built across **9 phases**:

1. **The $2.1M Problem: Why This Project Exists**
2. **Setting Up a Zero-Orphan Engineering Environment**
3. **Building the Network Foundation with Cost-Aware VPC Design**
4. **Deploying the Genomic Workload Router and Storage Intelligence Layer**
5. **Validating the DataSync NFS-to-S3 Migration Pipeline**
6. **Migrating SQL Server to PostgreSQL with DMS and Schema Conversion**
7. **Deploying and Destroying Transfer Family SFTP with PGP Decryption**
8. **Red-Team Validation, Lessons Learned Presentation, and Terraform Destroy**
9. **NIH-Grade Compliance Vault with Object Lock**

For the full walkthrough with screenshots and step-by-step content, see [`documents/aws-zero-orphan-migration-platform.md`](./documents/aws-zero-orphan-migration-platform.md).

## Validation

Each build phase below is documented in [`documents/aws-zero-orphan-migration-platform.md`](./documents/aws-zero-orphan-migration-platform.md), with screenshots, configuration, and notes as captured during the build:

- ✅ The $2.1M Problem: Why This Project Exists
- ✅ Setting Up a Zero-Orphan Engineering Environment
- ✅ Building the Network Foundation with Cost-Aware VPC Design
- ✅ Deploying the Genomic Workload Router and Storage Intelligence Layer
- ✅ Validating the DataSync NFS-to-S3 Migration Pipeline
- ✅ Migrating SQL Server to PostgreSQL with DMS and Schema Conversion
- ✅ Deploying and Destroying Transfer Family SFTP with PGP Decryption
- ✅ Red-Team Validation, Lessons Learned Presentation, and Terraform Destroy
- ✅ NIH-Grade Compliance Vault with Object Lock
