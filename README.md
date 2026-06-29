# Cloud Systems Engineering

[![License: MIT](https://img.shields.io/badge/license-MIT-1B4332?style=flat-square&labelColor=0d1117)](./LICENSE) [![Systems](https://img.shields.io/badge/systems-10-2F5233?style=flat-square&labelColor=0d1117)](./INDEX.md) [![Updated](https://img.shields.io/badge/updated-2026--06--25-264653?style=flat-square&labelColor=0d1117)](./INDEX.md)

> *What's the technical foundation when AI is not the headline?*

Cloud platforms engineered for scale, reliability, and uptime. Each system in this domain ships with a Mermaid architecture diagram, a numbered implementation map, and a checkmark list of documented build phases. The original source document is kept per system.

## Who this is for

| Reader | What to look at | Time |
|--------|----------------|------|
| Recruiter | This README, badges | 10 sec |
| Hiring Manager | Systems list below, per-system Validation | 2 min |
| Engineer | Per-system Architecture + Implementation | 15 min |
| Learner | Per-system end-to-end source content | 30 min |

## What this domain covers

Cloud, SRE, DevOps, and platform engineering across the full technology stack. Includes infrastructure depth, reliability practices, and platform tooling that holds under production load.

**What it isn't.** A vendor-neutral comparison. A guarantee of operational scale.

## Featured Systems

- **[AWS Hybrid Cloud Network Architecture](./systems/aws-hybrid-cloud-network-architecture/)**: TGW hub-spoke with BGP failover measured at 38s reconvergence under a 60s target
- **[Architect an AWS Landing Zone from Scratch](./systems/aws-landing-zone-soc2-governance/)**: SOC 2 evidence chain with 6 SCPs, StackSets baseline, and Service Catalog account vending
- **[Get Promoted: Build AWS DR Worth $1M](./systems/aws-multi-region-dr-tiering/)**: 23s measured Checkout RTO, 4 of 4 gameday scenarios pass, Vault Lock blocks ransomware
- **[AWS Multi-Region Edge Failover Sprint](./systems/aws-multi-region-edge-failover-platform/)**: 1.1s measured failover vs 60s target, zero orphaned resources, $0.61 sprint
- **[Zero-Key AWS Security Architecture](./systems/zero-key-aws-security-architecture/)**: Zero-trust healthcare stack with 4 red-team attacks blocked across STS, KMS, S3 Lock, and ABAC

_+ 5 other systems in the full catalog: [`INDEX.md`](./INDEX.md)._


