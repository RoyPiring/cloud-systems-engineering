# Build an E-Commerce Analytics Lakehouse

> Inside the [Cloud Systems Engineering](../../README.md) portfolio · *Cloud platforms engineered for scale, reliability, and uptime.*

## Overview

This project builds an e-commerce analytics lakehouse using Apache Iceberg, Trino, Spark, and Polaris.

The goal is to create a system that supports large-scale data storage, real-time querying, and ACID-compliant updates without relying on proprietary data warehouses. Iceberg provides table-level versioning and schema evolution, while Trino enables interactive queries across the dataset. This combination allows the system to behave like a warehouse while maintaining the flexibility of a data lake.

The architecture is built across **9 phases**, anchored by **Configuring the Development Environment** on the input side and **Wrapping Up and Cleaning Down** at the end. Each phase is listed in the Implementation section below.

## Architecture

```mermaid
---
title: Build an E-Commerce Analytics Lakehouse
---
%%{init: {"theme":"base","themeVariables": {"primaryColor":"#1B4332","primaryTextColor":"#F4D03F","primaryBorderColor":"#F4D03F","secondaryColor":"#264653","tertiaryColor":"#2F5233","lineColor":"#F4D03F","fontFamily":"ui-monospace, SFMono-Regular, Menlo, Consolas, monospace","fontSize":"13px"}}}%%
flowchart LR
    classDef datastore fill:#264653,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef service fill:#1B4332,stroke:#F4D03F,stroke-width:2px,color:#F4D03F
    classDef event fill:#7B42BC,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef io fill:#0d1117,stroke:#F4D03F,stroke-width:1.5px,color:#F4D03F,font-style:italic







    User[/Analyst via Jupyter/]
    Cursor(Cursor IDE Agent Mode)

    subgraph Orchestration
      Compose(Docker Compose)
    end

    subgraph Compute
      Spark(Spark)
      Trino(Trino)
      Jupyter(Jupyter)
    end

    subgraph Catalog
      Polaris(Polaris Iceberg Catalog)
    end

    subgraph Storage
      MinIO[(MinIO Object Store)]
      Iceberg[(Iceberg Tables - orders, customers)]
      Snapshots[(Iceberg Snapshots & Metadata)]
    end

    Cursor -->|generates docker-compose, Trino config, Polaris bootstrap| Compose
    Compose -->|launches services| Spark
    Compose -->|launches services| Trino
    Compose -->|launches services| Jupyter
    Compose -->|launches services| Polaris
    Compose -->|launches services| MinIO

    Spark -->|writes DDL & sample data, runs CDC| Iceberg
    Trino -->|interactive SQL queries| Iceberg
    Jupyter -->|submits queries| Trino
    User -->|notebooks| Jupyter

    Trino -->|resolves tables via| Polaris
    Spark -->|registers tables via| Polaris
    Polaris -->|tracks table state in| Snapshots

    Iceberg -->|data & metadata files| MinIO
    Snapshots -->|stored in| MinIO
    Spark -->|compaction & snapshot expiration| Iceberg
    Iceberg -->|time-travel reads of prior snapshots| Trino
    class MinIO,Iceberg,Snapshots datastore
    class Cursor,Compose,Spark,Trino,Jupyter,Polaris service
    class User io
```

The diagram shows the topology and data flow of the system as built. The full architectural narrative, with screenshots and prose, lives in [`documents/lakehouse-iceberg-trino-spark.md`](./documents/lakehouse-iceberg-trino-spark.md).

## Implementation

This system is built across **9 phases**:

1. **Configuring the Development Environment**
2. **Scaffolding Infrastructure with Parallel Agents**
3. **Launching and Verifying the Lakehouse Stack**
4. **Designing the E-Commerce Data Model**
5. **Loading Data and Running Analytical Queries**
6. **Simulating Real-World Change Data Capture**
7. **Running Compaction, Expiration, and Generating Reports**
8. **Time-Travel Disaster Recovery**
9. **Wrapping Up and Cleaning Down**

For the full walkthrough with screenshots and step-by-step content, see [`documents/lakehouse-iceberg-trino-spark.md`](./documents/lakehouse-iceberg-trino-spark.md).

## Validation

Each build phase below is documented in [`documents/lakehouse-iceberg-trino-spark.md`](./documents/lakehouse-iceberg-trino-spark.md), with screenshots, configuration, and notes as captured during the build:

- ✅ Configuring the Development Environment
- ✅ Scaffolding Infrastructure with Parallel Agents
- ✅ Launching and Verifying the Lakehouse Stack
- ✅ Designing the E-Commerce Data Model
- ✅ Loading Data and Running Analytical Queries
- ✅ Simulating Real-World Change Data Capture
- ✅ Running Compaction, Expiration, and Generating Reports
- ✅ Time-Travel Disaster Recovery
- ✅ Wrapping Up and Cleaning Down
