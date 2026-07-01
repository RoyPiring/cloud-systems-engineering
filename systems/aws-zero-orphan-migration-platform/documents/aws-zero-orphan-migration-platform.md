<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# AWS Principal SA: Fix the $2.1M Pipeline

**Project Link:** [View Project](https://nextwork.ai/projects/97422248-5452-4c23-ab20-6c7dabbe3f8a)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/97422248-5452-4c23-ab20-6c7dabbe3f8a_owektzgv)

## The $2.1M Problem: Why This Project Exists

### Mission briefing and stakes

In this build, I built a Terraform-managed AWS platform for GenoVault to address a $2.1M migration pipeline problem. The system was designed to remove orphan resources, enforce strict Infrastructure as Code governance, and show how migration pipelines can lower cost while still meeting migration and compliance needs.

The main decision was to make Terraform the control plane for the environment. Resources were not created through one-off manual actions. They were declared in code, tracked in state, tagged for visibility, and tied to a clean teardown path.

This mattered because migration environments can become expensive when resources stay alive after their purpose is complete. The build proved that cost control, governance, migration design, and teardown discipline had to work as one system.

## Setting Up a Zero-Orphan Engineering Environment

### Environment goals

In this step, I set up a controlled infrastructure environment with version-pinned tools: Terraform 1.15.3, AWS CLI v2, Python 3.12, Node.js LTS, and Gpg4win. The pinned toolchain gave the build a stable base and reduced configuration drift during setup, deployment, validation, and teardown.

I then configured AWS credentials for secure access. This gave the environment a clear AWS access path without turning the AWS CLI into the main provisioning method.

I also scaffolded the main deliverables, including the React presentation and the nine-document briefing package. That gave the technical work a documentation layer from the start, so the architecture, decisions, and validation results could be explained through each workstream.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/97422248-5452-4c23-ab20-6c7dabbe3f8a_z5aw8kzb)

### Terraform version and CLI discipline

Terraform version: This build used Terraform v1.15.3 installed locally at .tools/bin/terraform.exe. The version was pinned so Terraform behavior would not shift silently during the build.

Why no AWS resources via CLI: The AWS CLI was limited to read-only checks, such as sts get-caller-identity, and to triggering tasks Terraform had already created. It was not used to provision infrastructure.

Reason: All infrastructure was declared and applied through Terraform so the environment stayed version-controlled, repeatable, and auditable. That discipline prevented resources from being created outside state, which supported the zero-orphan goal.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/97422248-5452-4c23-ab20-6c7dabbe3f8a_wjaugqep)

## Building the Network Foundation with Cost-Aware VPC Design

### VPC and Terraform foundation goals

In this step, I established the core networking foundation for the GenoVault migration by deploying a modular Terraform architecture. The VPC module created the network base with public and private subnets, an Internet Gateway, and an S3 Gateway Endpoint to keep S3 traffic on the AWS network path.

The VPC module handled public and private subnet placement, internet routing, and private S3 access through the gateway endpoint. The security groups controlled traffic between migration services, while the storage modules defined S3, EFS, FSx, and EBS with the IAM roles needed for least-privilege access.

The root module tied the network, storage, IAM, and service components together. That structure mattered because every deployed resource had to be correctly tagged, tracked in Terraform state, and ready for future lifecycle control.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/97422248-5452-4c23-ab20-6c7dabbe3f8a_ewdssdx3)

### S3 Gateway Endpoint vs NAT Gateway decision

An S3 Gateway Endpoint is free and sends S3 traffic from private subnets over the AWS network. It avoids the internet path and avoids hourly NAT charges.

A NAT Gateway costs ~$0.045/hr plus data processing, which adds up quickly in a cost-bounded lab targeting < $2 total spend. In this build, that cost model mattered because the environment had to prove the design without allowing infrastructure charges to drift.

GenoVault workloads only needed private-to-S3 access for DataSync, DMS, and Transfer Family uploads. The gateway endpoint covered that path without paying for general internet egress, which made it the right fit for the traffic pattern.

## Deploying the Genomic Workload Router and Storage Intelligence Layer

### Workload router and Storage Lens goals

In this step, I generated 5 synthetic workload profiles that represented GenoVault's genomic data patterns. Those profiles gave the system a way to test storage placement decisions against different retention, access, and retrieval needs.

I deployed a Lambda-based workload router to map those patterns to the right storage primitives. The router gave the build a decision layer instead of treating all genomic data as if it belonged in the same storage class.

I used S3 Storage Lens to validate the routing pipeline through test object uploads. That validation step mattered because the storage logic had to be checked through actual object placement, not only described in architecture notes.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/97422248-5452-4c23-ab20-6c7dabbe3f8a_590xc3ll)

### Glacier Deep Archive vs Flexible Retrieval for compliance workloads

The cold-compliance workload had 10-year retention and a 12-hour retrieval SLA with annual access. Deep Archive was the lowest-cost storage class at ~$0.00099/GB-month that still met that SLA.

Glacier Flexible Retrieval costs more to store at ~$0.0036/GB-month and is meant for workloads that need more frequent or faster restores. That made it the wrong fit for rarely touched compliance archives because it would pay for retrieval speed the workload did not need.

The router rule retention_years >= 10 and retrieval_sla_hours >= 12 mapped directly to Deep Archive. That aligned to SAP-C02 D2.6 by matching the storage class to access frequency instead of over-provisioning retrieval speed.

## Validating the DataSync NFS-to-S3 Migration Pipeline

### DataSync pipeline goals

In this step, I simulated the migration of GenoVault's 80 TB legacy data to the cloud. I deployed a Terraform-managed EC2 instance as a mock NFS server, then configured AWS DataSync to automate file transfer into the S3 bucket's raw/ prefix.

This created a controlled NFS-to-S3 migration path without needing the real legacy source system. The mock NFS server gave the build a testable source, while DataSync handled the managed transfer workflow.

After the migration, I validated that the objects landed in the target prefix and documented the architecture decision in ADR-005. That gave the pipeline both technical proof and decision traceability.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/97422248-5452-4c23-ab20-6c7dabbe3f8a_grstic1c)

### DataSync vs Snowball decision criteria for 80 TB migration

When choosing between DataSync and Snowball for an 80 TB migration, the deciding factor was the network path. Snowball made more sense when sustained bandwidth was below ~100 Mbps, because an online transfer would run longer than ~2 weeks and miss the timeline.

Snowball also made more sense when the source was air-gapped or had no reliable network path to AWS. In that case, physical shipping would be more reliable than trying to push 80 TB through a weak or unstable connection.

DataSync stayed the default when the environment had ≥1 Gbps sustained connectivity, which supported an 8-day transfer for 80 TB. It also supported incremental sync during migration and kept total cost lower at $1,000 versus device and shipping overhead.

## Migrating SQL Server to PostgreSQL with DMS and Schema Conversion

### DMS and Schema Conversion goals

In this step, I architected and deployed a heterogeneous database migration pipeline. I provisioned RDS SQL Server and PostgreSQL instances and stored their credentials in AWS Secrets Manager so database access could be managed securely.

I set up the DMS replication infrastructure and performed schema conversion to map SQL Server structures to PostgreSQL. This step mattered because the source and target database engines do not use the same data types, syntax, or object definitions.

I configured a full-load plus Change Data Capture task to support real-time data synchronization. That gave the database migration both an initial load path and an ongoing replication path.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/97422248-5452-4c23-ab20-6c7dabbe3f8a_cj2mu1tz)

### Why Schema Conversion is mandatory for heterogeneous migration

SQL Server and PostgreSQL use different data types, syntax, and object definitions. The target cannot safely receive rows until a PostgreSQL-compatible schema exists.

Schema Conversion turns the source DDL into PostgreSQL DDL and flags objects that do not translate cleanly before migration. This made schema readiness an explicit step instead of assuming the data movement tool could solve engine differences on its own.

DMS then loads and replicates data into those existing target tables. It does not fully build a heterogeneous schema for you, so schema conversion had to come before reliable data movement.

## Deploying and Destroying Transfer Family SFTP with PGP Decryption

### Transfer Family and Lambda PGP decrypt goals

In this step, I deployed the Transfer Family SFTP server and the supporting Lambda function for PGP decryption through Terraform. This created a managed file-ingest path where encrypted files could arrive through SFTP and be processed in S3.

The next validation step was to upload an encrypted test file through SFTP and confirm that automatic decryption worked as expected in S3. That test checked the handoff between Transfer Family, S3, and Lambda.

After testing, I destroyed the Transfer Family resources right away to control cost. That kept the high-cost service tied to the validation window instead of allowing it to keep billing after its purpose was complete.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/97422248-5452-4c23-ab20-6c7dabbe3f8a_i36xjd0e)

### Deploy-test-destroy pattern for cost governance

Expensive services like Transfer Family bill per hour while they are running. The pattern placed those resources behind a flag, transfer_family_enabled, so they only existed during the validation window.

I deployed the resources for testing, ran the validation, then set the flag to false and ran terraform apply to stop hourly charges right away. This made the cost-control step part of the infrastructure workflow instead of a manual reminder.

Always-on, low-cost components such as Lambda, IAM, and Secrets Manager stayed in place. Only the temporary, metered resources were destroyed, which kept the system useful while removing the expensive idle layer.

## Red-Team Validation, Lessons Learned Presentation, and Terraform Destroy

### Final validation and teardown goals

The deploy-test-destroy pattern prevented cost overruns when working with high-cost AWS services such as Transfer Family. Infrastructure as Code made the pattern repeatable because the resources were intentional, declared, and temporary.

This mattered because expensive services should not survive past their test window. Terraform gave the build a clean way to create them, validate them, and remove them without relying on manual cleanup.

The teardown step also tested whether the governance model held under real constraints. A clean destroy result proved that resources were tracked in state, tagged for review, and tied to a controlled lifecycle.

### What zero orphans proves about IaC-enforced architecture

terraform destroy (434869) completed successfully. 95 resources were destroyed, the Terraform state was empty, and the Gate 4 tag query returned ResourceTagMappingList: [].

Re-run destroy + ARN validation (901651) reported Destroy complete! Resources: 0 destroyed because the state was already empty. Exit code 254 came from expected NotFound errors while validating deleted EC2/VPC endpoint ARNs in the stale tagging index. It did not come from a failed destroy.

Vite dev servers (619355, 593066) both started normally at localhost:5173 / 5174 and were stopped intentionally after the presentation and Gate 4 screenshot work. The non-zero exit codes came from manual process termination, not application failures.

## Secret Mission: NIH-Grade Compliance Vault with Object Lock

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/97422248-5452-4c23-ab20-6c7dabbe3f8a_zcsk001k)

### Separate Terraform workspace for long-lived compliance resources

The main workspace was torn down with terraform destroy during Gate 4 lab teardown, but the compliance vault had to survive that process. COMPLIANCE-mode Object Lock locks objects for 10 years, so the vault could not follow the same lifecycle as the temporary lab environment.

Even root cannot delete COMPLIANCE-mode locked objects early. That meant the bucket could not be safely managed in a workspace that was routinely destroyed, because the destroy workflow and the retention requirement had opposite goals.

A separate terraform-compliance/ workspace kept permanent, regulated storage outside the temporary lab lifecycle. This separated long-lived compliance storage from the main environment while still keeping it under Terraform management.

## Reflections: Tools, Concepts, and the Principal SA Mindset

### Key tools and concepts mastered

The key tools I used included Terraform for Infrastructure as Code, AWS DataSync for migration, DMS with Schema Conversion for database migration tasks, Transfer Family for SFTP, and S3 Batch Operations for lifecycle management.

Key concepts I learned included strict tagging governance to prevent orphan resource buildup, separate Terraform workspaces for persistent compliance-locked data, and migration pipelines that used automated storage right-sizing to control cost.

The larger lesson was that architecture is not only about making services run. It is about making sure every service has a reason to exist, a cost boundary, a security model, and a lifecycle path.

### Time and challenge reflection

This build took me approximately 10 hours. That time went into more than deployment work. It included setting up the environment, shaping Terraform modules, connecting AWS services, validating migration paths, documenting architecture decisions, preparing the presentation, and proving teardown results.

The most challenging part was managing cross-service dependencies inside the Terraform modules, especially making sure the DataSync agent and the Compliance-mode S3 bucket were correctly isolated while still fitting into the broader architecture. The system had temporary resources, long-lived compliance resources, migration services, database services, and documentation outputs that all had to stay aligned.

Understanding why the compliance vault required a separate Terraform workspace was a key learning point. It showed how infrastructure governance protects long-lived regulated storage from accidental deletion while still allowing the main lab environment to be destroyed cleanly.

### Learning goals and next steps

I completed this build today to master Infrastructure as Code governance, with a focus on preventing resource sprawl and cloud waste through strict Terraform lifecycle management and required tagging.

Moving forward, I want to expand my skills by learning how to build automated cost-anomaly detection and proactive budget alerting. That would extend the same governance model into a production-scale environment where cost signals, budget thresholds, and anomaly detection can catch waste before it grows.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/97422248-5452-4c23-ab20-6c7dabbe3f8a)*
