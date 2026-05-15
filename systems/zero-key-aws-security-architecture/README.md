# Zero-Key AWS Security Architecture

> Inside the [Cloud Systems Engineering](../../README.md) portfolio · *Cloud platforms engineered for scale, reliability, and uptime.*

## Overview

In this project, I designed and validated a zero-trust AWS security architecture for a healthcare environment supporting approximately 3,000 federated users. The objective was to eliminate long-lived credentials and replace them with short-lived, auditable identity workflows built around federation, temporary access, encryption governance, and centralized access control.

The environment combined AWS Organizations, IAM Identity Center, GitHub OIDC federation, multi-region KMS, Secrets Manager rotation, WorkSpaces failover architecture, and ABAC authorization into a unified security model. Instead of relying on static IAM users, SSH keys, or manually distributed credentials, the architecture enforced temporary, policy-driven access where trust is continuously validated.

This project simulated an enterprise healthcare security modernization effort where identity, encryption, auditability, and governance are treated as foundational infrastructure.

The architecture is built across **13 phases**, anchored by **Building a Zero-Trust Security Architecture for Healthcare** on the input side and **Red-Teaming the Architecture: Four Attacks, Four Blocks** at the end. Each phase is listed in the Implementation section below.

## Architecture

```mermaid
---
title: Zero-Key AWS Security Architecture
---
%%{init: {"theme":"base","themeVariables": {"primaryColor":"#1B4332","primaryTextColor":"#F4D03F","primaryBorderColor":"#F4D03F","secondaryColor":"#264653","tertiaryColor":"#2F5233","lineColor":"#F4D03F","fontFamily":"ui-monospace, SFMono-Regular, Menlo, Consolas, monospace","fontSize":"13px"}}}%%
flowchart LR
    classDef datastore fill:#264653,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef service fill:#1B4332,stroke:#F4D03F,stroke-width:2px,color:#F4D03F
    classDef event fill:#7B42BC,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef io fill:#0d1117,stroke:#F4D03F,stroke-width:1.5px,color:#F4D03F,font-style:italic

    subgraph IaC["OpenTofu Multi-Region Bootstrap"]
        Tofu(OpenTofu Project)
        ProviderA(us-east-1 Provider Alias)
        ProviderB(us-west-2 Provider Alias)
        ADRs[(Module Boundaries + Provider Pins)]
    end

    subgraph Org["AWS Organization Multi-Account"]
        OrgRoot(Organization Root)
        MgmtAcct(Management Payer Account)
        SharedAcct(Shared Services Account)
        WorkloadAcct(Workload Account)
        STS(STS AssumeRole Cross-Account)
    end

    subgraph Identity["Federated Identity for 3,000 Users"]
        ThreeKUsers[("3,000 Healthcare Users")]
        SambaAD(Samba 4 Domain Controller on EC2)
        ADConnector(AD Connector Small)
        IdC(IAM Identity Center)
        PermSets[(Permission Sets)]
    end

    subgraph ZeroSSH["Zero-SSH Operations"]
        SSM(Systems Manager Session Manager)
        InstanceProfile(IAM Instance Profile)
        CTLog[(CloudTrail Audit Trail)]
    end

    subgraph CICD["Zero-Key CI/CD"]
        GHActions(GitHub Actions Workflow)
        GHOIDC(GitHub OIDC Identity Provider)
        DeployRole(Deployment Role)
        PermBoundary(Permission Boundary)
        DenyList[(Explicit Denies: iam:CreateUser, cloudtrail:StopLogging)]
    end

    subgraph ABAC["ABAC Authorization"]
        SessionTags(Session Tags Project-Scoped)
        STSEval(STS Trust Policy + Tag Eval)
        BucketPolicy(S3 Bucket Policy: PrincipalTag == ResourceTag)
    end

    subgraph KMS["Multi-Region KMS + Envelope Encryption"]
        KMSPrimary(KMS Multi-Region Primary)
        KMSReplica(KMS Multi-Region Replica)
        DataKey(AES-256 Data Key)
        KMSGrant(Temporary KMS Grants)
        PHI[(PHI Sample File)]
    end

    subgraph Secrets["Secrets Rotation"]
        SecretsMgr(Secrets Manager)
        RDS(RDS MySQL)
        RotationLambda(Rotation Lambda)
        CredVersions[(AWSCURRENT + AWSPREVIOUS + AWSPENDING)]
    end

    subgraph Storage["Ransomware-Resistant Storage"]
        S3Bucket(S3 Versioned Bucket)
        ObjectLock(Object Lock Compliance Mode)
        BucketKeys(S3 Bucket Keys)
        MFADelete(MFA Delete)
    end

    subgraph Workforce["Remote Workforce Failover"]
        WorkSpacesUS1(WorkSpaces us-east-1)
        WorkSpacesUS2(WorkSpaces us-west-2)
        Route53(Route 53 Failover Policy)
        ConnAlias(Connection Alias FQDN)
    end

    subgraph RedTeam["Red-Team Validation"]
        Attack1{{Cross-Account Escalation Block}}
        Attack2{{KMS Bypass Attempt Denied}}
        Attack3{{S3 Object Delete Blocked}}
        Attack4{{Session-Tag Bypass Denied}}
    end

    Tofu --> OrgRoot
    ProviderA -.-> WorkloadAcct
    ProviderB -.-> WorkloadAcct
    OrgRoot --> MgmtAcct
    OrgRoot --> SharedAcct
    OrgRoot --> WorkloadAcct

    ThreeKUsers --> SambaAD
    SambaAD --> ADConnector
    ADConnector --> IdC
    IdC --> PermSets
    PermSets --> STS
    STS --> WorkloadAcct

    SambaAD -.managed via.-> SSM
    SSM --> InstanceProfile
    SSM --> CTLog

    GHActions --> GHOIDC
    GHOIDC --> DeployRole
    DeployRole --> PermBoundary
    PermBoundary --> DenyList
    DeployRole -->|short-lived creds| WorkloadAcct

    SessionTags --> STSEval
    STSEval --> BucketPolicy
    BucketPolicy --> S3Bucket

    KMSPrimary -.replicates.-> KMSReplica
    KMSPrimary --> DataKey
    DataKey --> PHI
    KMSGrant -.short-lived delegation.-> DataKey
    KMSPrimary --> CTLog

    SecretsMgr --> RotationLambda
    RotationLambda --> RDS
    SecretsMgr --> CredVersions

    S3Bucket --> ObjectLock
    S3Bucket --> BucketKeys
    S3Bucket --> MFADelete

    WorkSpacesUS1 -.region failover.-> WorkSpacesUS2
    Route53 --> ConnAlias
    ConnAlias --> WorkSpacesUS1
    ADConnector --> WorkSpacesUS1

    Attack1 -.blocks.-> STS
    Attack2 -.blocks.-> KMSPrimary
    Attack3 -.blocks.-> ObjectLock
    Attack4 -.blocks.-> BucketPolicy

    class Tofu,ProviderA,ProviderB io
    class OrgRoot,MgmtAcct,SharedAcct,WorkloadAcct,STS service
    class SambaAD,ADConnector,IdC,SSM,InstanceProfile,GHActions,GHOIDC,DeployRole,PermBoundary,SecretsMgr,RotationLambda,KMSPrimary,KMSReplica,KMSGrant,SessionTags,STSEval,BucketPolicy,S3Bucket,ObjectLock,BucketKeys,MFADelete,WorkSpacesUS1,WorkSpacesUS2,Route53,ConnAlias service
    class ADRs,PermSets,ThreeKUsers,CTLog,DenyList,CredVersions,RDS,DataKey,PHI datastore
    class Attack1,Attack2,Attack3,Attack4 event
```

The diagram shows the topology and data flow of the system as built. The full architectural narrative, with screenshots and prose, lives in [`documents/zero-key-aws-security-architecture.md`](./documents/zero-key-aws-security-architecture.md).

## Implementation

This system is built across **13 phases**:

1. **Building a Zero-Trust Security Architecture for Healthcare**
2. **Scaffolding the Multi-Account Infrastructure**
3. **Establishing AWS Organizations and Cross-Account Trust**
4. **Deploying a Zero-SSH Active Directory Domain Controller**
5. **Federating 3,000 Users Through AD Connector and Identity Center**
6. **Achieving Zero-Key CI/CD with GitHub OIDC and Permission Boundaries**
7. **Enforcing ABAC Isolation Across Projects Without Policy Rewrites**
8. **Implementing Multi-Region KMS Keys and Envelope Encryption**
9. **Exposing the Parameter Store Trap and Automating Secrets Rotation**
10. **Proving Ransomware Resistance with S3 Object Lock**
11. **Deploying WorkSpaces with Cross-Region Failover Architecture**
12. **Presenting the Architecture to Leadership**
13. **Red-Teaming the Architecture: Four Attacks, Four Blocks**

For the full walkthrough with screenshots and step-by-step content, see [`documents/zero-key-aws-security-architecture.md`](./documents/zero-key-aws-security-architecture.md).

## Validation

Build outcomes verified end-to-end. Each phase below is captured with screenshots, configuration, and observable behavior in [`documents/zero-key-aws-security-architecture.md`](./documents/zero-key-aws-security-architecture.md):

- ✅ Building a Zero-Trust Security Architecture for Healthcare
- ✅ Scaffolding the Multi-Account Infrastructure
- ✅ Establishing AWS Organizations and Cross-Account Trust
- ✅ Deploying a Zero-SSH Active Directory Domain Controller
- ✅ Federating 3,000 Users Through AD Connector and Identity Center
- ✅ Achieving Zero-Key CI/CD with GitHub OIDC and Permission Boundaries
- ✅ Enforcing ABAC Isolation Across Projects Without Policy Rewrites
- ✅ Implementing Multi-Region KMS Keys and Envelope Encryption
- ✅ Exposing the Parameter Store Trap and Automating Secrets Rotation
- ✅ Proving Ransomware Resistance with S3 Object Lock
- ✅ Deploying WorkSpaces with Cross-Region Failover Architecture
- ✅ Presenting the Architecture to Leadership
- ✅ Red-Teaming the Architecture: Four Attacks, Four Blocks
