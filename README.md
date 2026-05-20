# Cloud Systems Engineering

[![License: MIT](https://img.shields.io/badge/license-MIT-1B4332?style=flat-square&labelColor=0d1117)](./LICENSE) [![Systems](https://img.shields.io/badge/systems-7-2F5233?style=flat-square&labelColor=0d1117)](./INDEX.md) [![Updated](https://img.shields.io/badge/updated-2026--05--19-264653?style=flat-square&labelColor=0d1117)](./INDEX.md)

> *What's the technical foundation when AI is not the headline?*

Cloud platforms engineered for scale, reliability, and uptime. Each system in this domain ships with a Mermaid architecture diagram, a numbered implementation map, and a checkmark list of build outcomes verified end-to-end. The original source document is kept per system.

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

- **[AWS FinOps Command Center](./systems/aws-finops-command-center/)**: 9-phase FinOps stack with synthetic spike test that ties anomaly to stopped EC2 and DDB log
- **[AWS Hybrid Cloud Network Architecture](./systems/aws-hybrid-cloud-network-architecture/)**: Hub-spoke TGW with measured 38s BGP reconvergence under a 60s target during VPN failover drill
- **[Architect an AWS Landing Zone from Scratch](./systems/aws-landing-zone-soc2-governance/)**: 10-phase landing zone with SCP smoke tests and StackSet drift feeding a SOC 2 evidence matrix
- **[Get Promoted: Build AWS DR Worth $1M](./systems/aws-multi-region-dr-tiering/)**: Tiered DR with measured 23s checkout RTO, ransomware vault-lock drill, and 63x ROI on $74K spend
- **[Zero-Key AWS Security Architecture](./systems/zero-key-aws-security-architecture/)**: 13-phase zero-trust build with 4 red-team attacks blocked by named KMS, ABAC, and Object Lock controls

_+ 2 other systems in the full catalog: [`INDEX.md`](./INDEX.md)._


