<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Build an E-Commerce Analytics Lakehouse

**Project Link:** [View Project](https://learn.nextwork.org/projects/06254081-9b9c-45a6-8086-f39af90d752a)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/06254081-9b9c-45a6-8086-f39af90d752a_r4w8n3jf)

## Why Build an Iceberg Lakehouse?

### The case for open lakehouse architecture

This project builds an e-commerce analytics lakehouse using Apache Iceberg, Trino, Spark, and Polaris.

The goal is to create a system that supports large-scale data storage, real-time querying, and ACID-compliant updates without relying on proprietary data warehouses. Iceberg provides table-level versioning and schema evolution, while Trino enables interactive queries across the dataset. This combination allows the system to behave like a warehouse while maintaining the flexibility of a data lake.

## Configuring the Development Environment

### Docker Desktop and Cursor IDE setup

The environment establishes the execution layer for the entire lakehouse stack.

Docker Desktop is configured with sufficient memory to support multiple services running concurrently, including storage, query engines, and orchestration tools. Cursor is used as the development interface, with Agent Mode enabling automated file generation and command execution. This setup allows infrastructure and configuration to be created and validated without manual repetition.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/06254081-9b9c-45a6-8086-f39af90d752a_r7nq4w8e)

### Creating the .cursorrules file

The .cursorrules file defines how Cursor agents behave during execution.

It enables autonomous workflows where the agent plans, executes, and validates tasks in a loop. This reduces manual intervention and ensures consistent generation of configuration files across services.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/06254081-9b9c-45a6-8086-f39af90d752a_k5jm8v3a)

### Using Cursor Plan Mode for system design

Plan Mode is used to define architecture before execution.

This separates design from implementation, ensuring that system structure is validated before resources are created. It reduces rework and enforces a deliberate approach to building the lakehouse.

## Scaffolding Infrastructure with Parallel Agents

### Generating Docker Compose, Trino config, and Polaris bootstrap simultaneously

Infrastructure is generated using parallel agents to accelerate setup.

Docker Compose defines the service topology, Trino configuration establishes query capabilities, and Polaris initializes the Iceberg catalog. Running these tasks in parallel reduces setup time while maintaining consistency across components.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/06254081-9b9c-45a6-8086-f39af90d752a_r3n7w9bv)

### Cross-checking credentials and ports with Cursor

Parallel generation introduces risk of misalignment.

Cross-checking ensures that credentials, ports, and service endpoints are consistent across all configuration files. Without this step, the system may fail silently due to mismatched dependencies.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/06254081-9b9c-45a6-8086-f39af90d752a_h6y4f1dq)

## Launching and Verifying the Lakehouse Stack

### Starting all five services with Docker Compose

The stack is launched as a unified system.

Services including MinIO, Polaris, Trino, Spark, and Jupyter are started together, creating a complete data platform environment. This validates that all components can communicate and operate as expected.

### Automated health checks across MinIO, Polaris, Trino, and Jupyter

Health checks confirm system readiness.

Certain setup containers exit after completing initialization tasks, which is expected behavior. The remaining services stay active, confirming that the lakehouse stack is running correctly.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/06254081-9b9c-45a6-8086-f39af90d752a_h2c9v7wd)

## Designing the E-Commerce Data Model

### Schema design with Cursor Plan Mode

The data model defines how business data is structured.

Tables are designed to represent customers, orders, and related entities, ensuring that analytical queries can be executed efficiently. Schema design is driven by query patterns rather than storage convenience.

### Generating and executing Iceberg table DDL

DDL statements define table structure and optimization settings.

Partitioning is applied where it improves query performance, such as time-based partitions for transactional data. Compression settings are configured to balance storage efficiency and read performance.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/06254081-9b9c-45a6-8086-f39af90d752a_c9r5t1y7)

### Verifying tables and metadata in MinIO

Verification ensures that tables are correctly created and stored.

Metadata files in MinIO confirm that Iceberg is managing table state, including schema and snapshot information. This validates that the system is operating as a true lakehouse rather than simple file storage.

## Loading Data and Running Analytical Queries

### Generating sample data and business queries in parallel

Sample data is loaded to simulate real-world usage.

Queries are executed to validate that the system can support analytical workloads, including aggregations and joins across tables.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/06254081-9b9c-45a6-8086-f39af90d752a_rn3w8v2f)

### Capturing query baseline and execution times

Baseline metrics establish performance expectations.

Query execution times provide a reference point for future optimization, ensuring that changes to the system can be measured against initial performance.

### Inspecting Iceberg snapshots and file statistics

Snapshots track table history and changes over time.

File counts indicate storage layout and performance characteristics. These metrics are used to determine when maintenance operations are required.

## Simulating Real-World Change Data Capture

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/06254081-9b9c-45a6-8086-f39af90d752a_r4w8n3jt)

### Analyzing CDC impact on file count and snapshot growth

CDC operations introduce the small file problem.

Frequent changes increase file count and snapshot complexity, which can degrade query performance. This highlights the need for maintenance processes.

## Running Compaction, Expiration, and Generating Reports

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/06254081-9b9c-45a6-8086-f39af90d752a_r3t8w5bn)

### Before-and-after compaction metrics

Metrics validate the impact of maintenance.

Reduced file counts improve query performance, while snapshot management ensures that metadata remains manageable as the dataset grows.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/06254081-9b9c-45a6-8086-f39af90d752a_m8j3f6xa)

### Validation report and industry benchmark comparison

The system is evaluated against expected performance patterns.

Comparing results to industry practices ensures that the lakehouse operates within acceptable performance and scalability thresholds.

## Secret Mission: Time-Travel Disaster Recovery

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/06254081-9b9c-45a6-8086-f39af90d752a_kw7m3xp2)

### Recovering deleted orders from a previous Iceberg snapshot

Logical deletes do not immediately remove data from storage.

Previous snapshots retain references to older data files, allowing queries to access historical states. This provides safety and auditability, with cleanup handled later through maintenance operations.

## Wrapping Up and Cleaning Down

### Resource summary and cost breakdown

This project deployed a local lakehouse using MinIO, Polaris, Trino, Spark, and Jupyter, orchestrated through Cursor.

There is no cloud cost since everything runs locally. Resource usage is limited to CPU and memory across the Docker services.

### Options to keep, pause, or tear down the stack

The stack can stay running for continued testing, paused by stopping containers, or removed with Docker Compose.

Teardown is the default after validation to free system resources.

### What you proved and what comes next

This project shows how to build and operate an Iceberg lakehouse locally with full data lifecycle control.

Next step is going deeper into disaster recovery and data platform operations.

---

*Built with [NextWork](https://learn.nextwork.org) - [View this project](https://learn.nextwork.org/projects/06254081-9b9c-45a6-8086-f39af90d752a)*
