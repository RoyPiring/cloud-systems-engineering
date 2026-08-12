# Error Budget Control Room

> Inside the [Cloud Systems Engineering](../../README.md) portfolio · *Cloud platforms engineered for scale, reliability, and uptime.*

## Overview

I built the control room to derive and validate error-budget burn-rate thresholds instead of accepting generic alert values without understanding them.

The thresholds matter because bad alert design creates two opposite risks. Alerts that fire too often create noise and fatigue, while alerts that react too slowly can let meaningful SLO consumption continue without intervention.

Deriving the values manually gave me a way to connect the alert configuration back to the underlying SRE math. The result was a system where the on-call behavior could be traced to explicit assumptions about budget consumption, SLO period, and alert windows.

The architecture is built across **7 phases**, anchored by **Building an Error Budget Control Room from First Principles** on the input side and **Scoring the AI: After-Action Review** at the end. Each phase is listed in the Implementation section below.

## Architecture

```mermaid
---
title: Error Budget Control Room
---
%%{init: {"theme":"base","themeVariables": {"primaryColor":"#1B4332","primaryTextColor":"#F4D03F","primaryBorderColor":"#F4D03F","secondaryColor":"#264653","tertiaryColor":"#2F5233","lineColor":"#F4D03F","fontFamily":"ui-monospace, SFMono-Regular, Menlo, Consolas, monospace","fontSize":"13px"}}}%%
flowchart TD
    classDef datastore fill:#264653,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef service fill:#1B4332,stroke:#F4D03F,stroke-width:2px,color:#F4D03F
    classDef event fill:#7B42BC,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef io fill:#0d1117,stroke:#F4D03F,stroke-width:1.5px,color:#F4D03F,font-style:italic

    Engineer[/SRE deriving burn-rate thresholds/]
    OnCall[/Defensible on-call behavior/]

    subgraph Stack["Containerized observability stack"]
        Compose(Docker Compose environment)
        Prometheus(Prometheus, metrics and alert evaluation)
        Grafana(Grafana visualization)
        K6(k6 load and error injection)
    end

    subgraph Delivery["Delivery trail"]
        Repo[(GitHub repo)]
        Board[(Six board issues: setup, design, build, prove, deliver, AAR)]
        GreenGate{{Setup issue Done only after the stack verified green}}
    end

    subgraph Design["Design before build"]
        SLICandidates[(SLI candidates)]
        CandidateC{{Candidate C rejected: no shared request id, cannot prove per-request AND}}
        MeasurableSLI[(Measurable SLI contract)]
        ADRs[(Architecture Decision Records)]
    end

    subgraph Spec["Declared source of truth"]
        OpenSLO[(OpenSLO specification)]
        Sloth(Sloth rule generation)
        PromRules[(Generated recording and alerting rules)]
    end

    subgraph Math["Independent derivation"]
        Workbook[(Google SRE Workbook: r_th = p x T / w)]
        FastCalc{{Fast burn: 2% of 30-day budget in 1h = 14.4x}}
        SlowCalc{{Slow burn: 5% of 30-day budget in 6h = 6x}}
        CrossCheck{{Hand-derived values must agree with generated rules}}
    end

    subgraph Proof["Firing-order proof"]
        InjectErrors(Inject a 10% error rate)
        FastAlert{{Fast-burn alert fires first}}
        SlowAlert{{Slow-burn alert fires later}}
        OrderProven{{Observed order matches the derived design}}
    end

    subgraph Lab["Compressed lab windows"]
        WindowSwap(1h/5m to 5m/25s, 6h/30m to 30m/2m30s)
        RatioKept{{12:1 long-to-short ratio and 14.4x / 6x preserved}}
        HonestLimit{{30-day budget period NOT compressed, so budget-percentage meaning does not carry}}
    end

    subgraph Deliver["Artifact set"]
        DerivationDoc[(Burn-rate derivation)]
        Readout[(Stakeholder readout)]
        TeachBack[(Teach-back)]
    end

    subgraph AAR["Scoring the AI"]
        Score1{{Score 1: one rejection, unmeasurable SLI proposed}}
        Score2{{Score 2: 2 of 6 wrong, multipliers missing}}
        ReviewChanged{{Review changed the implementation, not just approved it}}
    end

    Engineer -- "stands up" --> Compose
    Compose -- "runs" --> Prometheus
    Compose -- "runs" --> Grafana
    Compose -- "runs" --> K6
    Engineer -- "tracks work in" --> Repo
    Repo -- "carries" --> Board
    Prometheus -- "verified green before design" --> GreenGate
    GreenGate -- "unblocks" --> Board
    Engineer -- "drafts" --> SLICandidates
    SLICandidates -- "screened for measurability" --> CandidateC
    CandidateC -- "removed, leaving" --> MeasurableSLI
    MeasurableSLI -- "reasoning recorded in" --> ADRs
    MeasurableSLI -- "declared as" --> OpenSLO
    OpenSLO -- "generates through" --> Sloth
    Sloth -- "emits" --> PromRules
    Workbook -- "yields" --> FastCalc
    Workbook -- "yields" --> SlowCalc
    FastCalc -- "checked against" --> CrossCheck
    SlowCalc -- "checked against" --> CrossCheck
    PromRules -- "must agree with" --> CrossCheck
    PromRules -- "loaded into" --> Prometheus
    K6 -- "drives" --> InjectErrors
    InjectErrors -- "consumes budget seen by" --> Prometheus
    Prometheus -- "shorter window elapses first" --> FastAlert
    Prometheus -- "longer window elapses later" --> SlowAlert
    FastAlert -- "pages before" --> SlowAlert
    SlowAlert -- "confirms" --> OrderProven
    WindowSwap -- "shortens windows while" --> RatioKept
    RatioKept -- "makes observable" --> OrderProven
    WindowSwap -- "records" --> HonestLimit
    CrossCheck -- "documented in" --> DerivationDoc
    OrderProven -- "evidence for" --> Readout
    DerivationDoc -- "explained through" --> TeachBack
    HonestLimit -- "stated in" --> Readout
    CandidateC -- "counted as" --> Score1
    CrossCheck -- "counted as" --> Score2
    Score1 -- "shows AI not authoritative" --> ReviewChanged
    Score2 -- "shows AI not authoritative" --> ReviewChanged
    Readout -- "hands over" --> OnCall
    TeachBack -- "hands over" --> OnCall
    ReviewChanged -- "makes the system defensible to" --> OnCall

    class Repo,Board,SLICandidates,MeasurableSLI,ADRs,OpenSLO,PromRules,Workbook,DerivationDoc,Readout,TeachBack datastore
    class Compose,Prometheus,Grafana,K6,Sloth,InjectErrors,WindowSwap service
    class GreenGate,CandidateC,FastCalc,SlowCalc,CrossCheck,FastAlert,SlowAlert,OrderProven,RatioKept,HonestLimit,Score1,Score2,ReviewChanged event
    class Engineer,OnCall io
```

The diagram shows the topology and data flow of the system as built. The full architectural narrative, with screenshots and prose, lives in [`documents/error-budget-control-room.md`](./documents/error-budget-control-room.md).

## Implementation

This system is built across **7 phases**:

1. **Building an Error Budget Control Room from First Principles**
2. **Proving the Stack is Green**
3. **Designing with AI: Arguing the Denominator**
4. **Generating and Verifying Burn-Rate Rules**
5. **Proving Firing Order: The Fast Burn Pages First**
6. **Deriving and Delivering the Full Artifact Set**
7. **Scoring the AI: After-Action Review**

For the full walkthrough with screenshots and step-by-step content, see [`documents/error-budget-control-room.md`](./documents/error-budget-control-room.md).

## Validation

Each build phase below is documented in [`documents/error-budget-control-room.md`](./documents/error-budget-control-room.md), with screenshots, configuration, and notes as captured during the build:

- ✅ Building an Error Budget Control Room from First Principles
- ✅ Proving the Stack is Green
- ✅ Designing with AI: Arguing the Denominator
- ✅ Generating and Verifying Burn-Rate Rules
- ✅ Proving Firing Order: The Fast Burn Pages First
- ✅ Deriving and Delivering the Full Artifact Set
- ✅ Scoring the AI: After-Action Review
