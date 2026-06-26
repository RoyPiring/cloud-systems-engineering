<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# AWS Database Modernization Lab

**Project Link:** [View Project](https://nextwork.ai/projects/0a1064c2-636b-4a52-bc63-24955a29a2a8)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/0a1064c2-636b-4a52-bc63-24955a29a2a8_ex8pzx0e)

## Taking On the Assignment: Day 1 at Meridian Energy Trading

### The mission and why it matters

This lab evaluated modern AWS database services under a simulated tenfold increase in trading activity to determine the most appropriate architecture for different application workloads. The implementation compared Amazon Aurora Serverless v2, Amazon DynamoDB with DAX, and Amazon MemoryDB by migrating 50,000 trade records, executing k6 performance tests, and measuring latency, throughput, scalability, and operational cost. The objective was not to identify a single database for every workload, but to understand where each service delivered the greatest value while maintaining cost efficiency. The final deliverable was an executive HTML report recommending a modern AWS database architecture that raised application responsiveness, reduced operational overhead, and remained within AWS Free Tier limits throughout development.

## Architecting the Solution: The Requirement Package

### Planning before building

The implementation began by establishing a structured architectural foundation before any infrastructure was deployed. A dedicated requirement-package directory was created containing ADRs.md, architecture-diagrams.md, implementation-plan.md, risk-register.md, raci.md, and sap-c02-alignment.md. These documents captured architectural decisions, deployment sequencing, ownership responsibilities, operational risks, and alignment with AWS Solutions Architect Professional objectives. Completing this work first created a consistent engineering baseline, reduced ambiguity during implementation, and ensured that every deployment decision could be traced back to documented design requirements rather than being made during infrastructure provisioning.

### Critical ADR analysis

ADR-001 identified DynamoDB with DAX as the highest-priority architectural decision because trade lookups occur directly within the application's transaction path. Increased latency or database failures during this stage could delay order execution, affect pricing accuracy, and reduce overall platform reliability during peak trading periods. Selecting DynamoDB with DAX removed dependence on the legacy SQL Server bottleneck while reducing risks associated with a single relational database deployment. This decision demonstrated how aligning a workload with an appropriate database engine raises scalability, resiliency, and response time without requiring application redesign.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/0a1064c2-636b-4a52-bc63-24955a29a2a8_vx4a8sb0)

## Setting Up the Principal SA Workstation

### Step objectives

Before infrastructure deployment began, the workstation was prepared as a dedicated engineering environment capable of supporting infrastructure automation, benchmarking, and parallel development. Objectives included installing Terraform and k6, configuring the AWS CLI with an isolated project profile, and organizing a Cursor workspace capable of supporting multiple parallel agents. Establishing this environment early reduced deployment delays and ensured infrastructure provisioning, benchmarking, validation, and cleanup could all be executed from a consistent and repeatable engineering workstation using standardized tooling.

### Tool installation and verification

Terraform was upgraded using winget from version 1.15.2 to 1.15.7 to satisfy the minimum lab requirement. Existing installations of k6 2.0.0 and AWS CLI 2.34.32 already met project requirements, allowing those tools to be reused without additional installation. The remaining configuration involved preparing the dedicated meridian-prod AWS CLI profile with project credentials. Validating every dependency before deployment reduced troubleshooting during infrastructure creation and established confidence that the workstation could reliably support Terraform automation, benchmarking, AWS resource management, and operational verification.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/0a1064c2-636b-4a52-bc63-24955a29a2a8_7w7cln6m)

### Modular Terraform architecture rationale

Infrastructure was intentionally divided into independent Terraform modules covering networking, Aurora, DynamoDB, and MemoryDB. This modular architecture allowed separate Cursor agents to generate infrastructure simultaneously without modifying the same files, reducing merge conflicts and simplifying collaboration. Each module maintained ownership of a specific AWS service while remaining aligned with the Architecture Decision Records established during project planning. The resulting structure raised maintainability, simplified code reviews, and demonstrated how infrastructure as code scales effectively when multiple contributors or AI agents develop independent components within the same environment.

## Deploying Three Database Engines with Parallel AI Agents

### Deployment strategy

Infrastructure deployment combined parallel AI-assisted development with centralized engineering validation. Four Cursor agents independently generated Terraform modules for networking, Aurora, DynamoDB, and MemoryDB while maintaining clear separation between infrastructure domains. Each generated configuration was reviewed for IAM permissions, networking, security groups, and AWS Free Tier considerations before Terraform initialized and applied the complete environment. This workflow accelerated infrastructure development while preserving engineering oversight over security, deployment quality, and architectural consistency.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/0a1064c2-636b-4a52-bc63-24955a29a2a8_p95qxn55)

### Resources deployed and principal SA operating rhythm

Terraform successfully provisioned 52 AWS resources supporting the complete modernization environment, including networking components, PostgreSQL on EC2, AWS DMS endpoints, Aurora Serverless v2 with RDS Proxy, DynamoDB with DAX, MemoryDB, IAM roles, security groups, VPC endpoints, and CloudWatch dashboards. Parallel Cursor agents generated the initial Terraform modules while the primary workflow reviewed and corrected deployment issues involving Aurora engine versions, DMS authentication, IAM configuration, and MemoryDB user settings until every resource deployed successfully. This process demonstrated effective coordination between AI-assisted code generation and engineering validation during complex AWS infrastructure deployments.

## Migrating Data and Verifying All Three Engines

### Migration objectives

This phase automated migration and validation across multiple database platforms to establish consistent datasets before benchmarking began. AWS Database Migration Service performed a full-load migration from the EC2-hosted PostgreSQL source into Aurora Serverless v2 while identical datasets were seeded into DynamoDB and MemoryDB. Validation relied exclusively on AWS CLI commands instead of the AWS Management Console, reinforcing a repeatable workflow suitable for future automation. The objective was not only successful migration but also confidence that each platform contained equivalent information before performance testing.

### Cross-engine data verification with AWS CLI

Validation confirmed that DynamoDB successfully contained all 50,000 migrated records using AWS CLI count operations. Aurora and MemoryDB verification remained limited because both services were accessible only within the VPC, preventing direct validation through local psql and redis-cli utilities. Helper EC2 instances were launched to perform these checks, but cloud-init failures prevented successful execution, and DMS statistics continued reporting zero rows loaded into Aurora. Although complete parity verification was not achieved, the exercise highlighted practical networking and validation challenges commonly encountered during database modernization efforts operating inside private cloud environments.

## Simulating an OPEC Announcement: The 10x Spike Benchmark

### Benchmark scenario and goals

This phase evaluated how each database platform responded during a simulated market surge similar to an unexpected OPEC announcement that rapidly increased trading activity. Using k6 load-testing scripts, the environment generated a tenfold traffic increase while Amazon CloudWatch continuously monitored latency, throughput, resource utilization, and error rates across Aurora Serverless v2, DynamoDB with DAX, and MemoryDB. Operational metrics were also collected to estimate infrastructure costs at an expected workload of 500,000 reads per minute. Combining performance measurements with cost projections ensured architectural recommendations were supported by measurable data rather than assumptions.

### Performance results per engine

Each database engine demonstrated strengths for different workload patterns. MemoryDB delivered the lowest latency with approximately 5 ms p50 and 39 ms p99 response times during cache-style SET and GET operations, making it well suited for workloads requiring consistently low response times despite client throughput limitations. DynamoDB achieved the highest throughput during benchmarking at approximately 64 requests per second while maintaining stable latency and zero errors during GSI queries, demonstrating strong performance for high-volume keyed lookups. Aurora Serverless v2 operating through RDS Proxy maintained zero errors while exhibiting higher latency during SQL queries, reinforcing that relational databases remain the preferred choice when complex query capabilities outweigh raw response speed.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/0a1064c2-636b-4a52-bc63-24955a29a2a8_ypi3zly8)

## Delivering the Executive Report to the CTO

### Report objectives

The final deliverable translated technical benchmark results into executive-level recommendations suitable for engineering leadership. Performance metrics, benchmark charts, cost comparisons, and architectural findings were consolidated into an HTML report, while a decision matrix documented the strengths of Aurora Serverless v2, DynamoDB with DAX, and MemoryDB for future engineering reference. After all deliverables were completed, Terraform destroyed the deployed infrastructure to eliminate unnecessary AWS charges. This phase emphasized responsible cloud cost management by ensuring temporary benchmarking environments were removed immediately after project completion while preserving all required documentation and analysis.

### Recommendations and cost discipline

Benchmark results supported a polyglot persistence architecture that aligned each workload with the most appropriate AWS database service. DynamoDB with DAX was recommended for latency-sensitive trade lookups, Aurora Serverless v2 with RDS Proxy for relational analytics and reporting, and MemoryDB for real-time price distribution requiring durable, low-latency access. After reports and documentation were completed, Terraform destroy removed Aurora, DAX, MemoryDB, networking resources, and supporting infrastructure. Immediate cleanup prevented unnecessary charges while demonstrating disciplined cloud operations and responsible infrastructure lifecycle management.

## Secret Mission: I/O-Optimized, Babelfish, and the Revenue API

### Aurora I/O-Optimized break-even analysis

The lab extended beyond benchmarking by evaluating Aurora storage pricing strategies under production-scale workloads. Although benchmark performance remained comparable between Standard and I/O-Optimized storage, cost analysis showed that I/O-Optimized becomes significantly more economical when storage operations dominate database expenses. Meridian's projected analytical workload generated sufficient read activity for I/O charges to represent nearly all variable database costs, making Aurora I/O-Optimized the preferred configuration despite higher ACU pricing. This analysis demonstrated that storage architecture decisions should be driven by workload characteristics and long-term operational cost rather than benchmark latency alone.

### Babelfish T-SQL compatibility proof

Babelfish compatibility testing demonstrated that existing SQL Server workloads could execute against Aurora PostgreSQL without requiring immediate application rewrites. A legacy stored procedure successfully executed through sqlcmd from a VPC client using the TDS endpoint while returning commodity summary data from the migrated environment. This validation addressed concerns that hundreds of SQL Server stored procedures would require complete redevelopment before migration. Instead, the exercise demonstrated that compatibility testing could be performed incrementally while allowing existing SQL Server clients to continue operating throughout the migration process.

### Turning a cost project into a $600K/yr revenue product

The implementation also explored how infrastructure deployed for modernization could support new business opportunities without significant architectural changes. MemoryDB's durable, sub-millisecond read performance enabled the same internal commodity price feed to be exposed externally through a lightweight JSON API serving partner applications. Proof-of-concept testing successfully returned live commodity pricing directly from seeded MemoryDB data, demonstrating that the deployed infrastructure could support external consumption. This shifted the initiative beyond infrastructure cost reduction by illustrating how the modernized platform could also support future revenue-generating services.

## Mission Accomplished: Lessons from the Lab

### Tools and concepts mastered

This implementation strengthened practical experience with Terraform infrastructure as code, Cursor parallel agents, k6 performance testing, and AWS CLI automation while deploying Aurora Serverless v2, RDS Proxy, DynamoDB with DAX, MemoryDB, AWS DMS, CloudWatch, IAM, and supporting networking resources. Beyond individual services, the lab reinforced architectural concepts including polyglot persistence, Architecture Decision Records, Babelfish compatibility, workload-driven database selection, and AWS SAP-C02 design principles. Working through deployment, migration, benchmarking, troubleshooting, and infrastructure cleanup provided end-to-end experience managing a complete cloud modernization workflow using AWS-native services.

### Time and reflections

This implementation required approximately ten hours to complete and combined architectural planning, infrastructure deployment, migration, benchmarking, troubleshooting, and documentation into a single engineering workflow. The most challenging aspect involved rebuilding infrastructure as code while resolving dependency issues across Aurora, DynamoDB, MemoryDB, networking components, IAM roles, and security groups. Investigating Terraform provider logs and correcting deployment failures provided practical insight into how complex AWS environments behave during automated provisioning while reinforcing the importance of validating dependencies throughout multi-service cloud deployments.

### What's next

This lab provided practical experience benchmarking multiple AWS database services while reinforcing the importance of selecting the appropriate database engine for each workload rather than relying on a single platform. Deploying Aurora Serverless v2, DynamoDB with DAX, and MemoryDB demonstrated how specialized services complement one another when supporting unpredictable, high-concurrency applications. The next area of focus is implementing automated disaster recovery, cross-region replication, and resiliency strategies that extend these architectures beyond performance tuning and prepare them for highly available production environments spanning multiple AWS Regions.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/0a1064c2-636b-4a52-bc63-24955a29a2a8)*
