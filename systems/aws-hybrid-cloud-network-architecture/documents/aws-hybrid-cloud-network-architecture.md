<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# AWS Hybrid Cloud Network Architecture

**Project Link:** [View Project](https://learn.nextwork.org/projects/487fa828-7a41-4a1d-8ae1-ff6c7b302675)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/487fa828-7a41-4a1d-8ae1-ff6c7b302675_f6rh3d0k)

## Building a Fortune 500 Network Reference Architecture

### The $50B media conglomerate challenge

In this project, I built a production-grade AWS hybrid cloud network architecture designed for a Fortune 500 media organization supporting hundreds of engineering teams across multiple AWS accounts and on-premises datacenters.

The objective was to design a scalable, segmented, and secure hybrid networking model using Terraform and AWS-native networking services. The architecture combined Transit Gateway hub-spoke routing, hybrid VPN connectivity with BGP, centralized inspection through AWS Network Firewall, Route 53 hybrid DNS, PrivateLink connectivity, and compliance validation into a unified enterprise networking design.

## Scaffolding the Multi-Account Terraform Environment

### Project setup and objectives

I verified Terraform and AWS CLI installations, exported the validation results into env-verification.txt, and scaffolded the full project directory structure containing Terraform root files, reusable networking modules, diagram folders, and operational documentation.

The environment included dedicated modules for VPC blueprints, Transit Gateway orchestration, Network Firewall, VPN connectivity, endpoints, Route 53 resolvers, PrivateLink services, and reachability analysis workflows.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/487fa828-7a41-4a1d-8ae1-ff6c7b302675_2wfvbvi7)

### Simulating cross-account architecture

The account_b provider alias simulated a second AWS account operating within the same region. This mirrors real enterprise landing-zone architectures where networking, workloads, and security services are intentionally separated into different accounts to reduce blast radius and simplify governance boundaries.

Using multiple Terraform providers allowed the architecture to model cross-account Transit Gateway sharing, centralized inspection, and shared networking workflows without collapsing all infrastructure into a single AWS account.

## Deploying the Transit Gateway Hub-Spoke Network

### Four-VPC topology with TGW route table segmentation

I deployed a four-VPC topology connected through AWS Transit Gateway using dedicated route table segmentation for spoke traffic and inspection traffic.

The architecture separated workload routing from inspection return paths, allowing centralized firewall enforcement and deterministic east-west traffic behavior across the environment.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/487fa828-7a41-4a1d-8ae1-ff6c7b302675_vayo8m2h)

### Why explicit route table control matters at scale

Automatic Transit Gateway route propagation was intentionally disabled so attachments could be associated explicitly with either the spoke route table or the inspection route table.

This becomes critical at enterprise scale because centralized inspection architectures require predictable traffic steering rather than relying on default route behavior. Explicit routing also sharpens segmentation, reduces accidental trust paths, and simplifies operational troubleshooting during incident response.

## Establishing Hybrid Connectivity via Site-to-Site VPN with BGP

### Simulating on-premises datacenter connectivity

I deployed an EC2-based customer gateway appliance configured with strongSwan and FRRouting to simulate an on-premises datacenter environment.

A Site-to-Site VPN connection was then attached directly to the Transit Gateway with BGP enabled, allowing dynamic route exchange between the AWS environment and the simulated on-premises network.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/487fa828-7a41-4a1d-8ae1-ff6c7b302675_rvcixwvw)

### BGP dynamic routing and automatic failover

The VPN attached directly to the Transit Gateway rather than a single VPC Virtual Private Gateway, allowing the on-premises environment to communicate with all TGW-connected spoke networks simultaneously.

BGP was enabled by setting static_routes_only = false, allowing routes to be learned dynamically and updated automatically during tunnel failover events. This removed the need for manual route-table modification during outages and aligned the design with real enterprise hybrid-network behavior.

## Centralizing Egress Inspection with AWS Network Firewall

### PCI-DSS compliant centralized inspection architecture

I created an AWS Network Firewall policy using Suricata stateful inspection rules designed to block traffic to known malicious domains and unsafe destinations.

The firewall was deployed through a Transit Gateway network function attachment, and spoke route tables were updated to force outbound traffic through the centralized inspection layer before internet egress occurred.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/487fa828-7a41-4a1d-8ae1-ff6c7b302675_y4s9pe9k)

### Why Transit Gateway is required for centralized inspection

VPC peering is non-transitive and cannot support centralized inspection architectures at enterprise scale. Transit Gateway acts as the regional networking hub, allowing all spoke VPCs to route traffic through a shared inspection layer consistently.

This architecture created a single enforcement point for outbound traffic policy while avoiding the operational complexity and scaling limitations associated with large VPC peering meshes.

## Eliminating NAT Costs with Gateway Endpoints and PrivateLink

### S3/DynamoDB gateway endpoints and private SaaS connectivity

I deployed S3 and DynamoDB gateway endpoints inside the application VPC to validate NAT-free service access across the AWS backbone.

I also created a PrivateLink producer service backed by an internal Network Load Balancer and consumed the service from a separate VPC to validate fully private service-to-service communication without internet exposure.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/487fa828-7a41-4a1d-8ae1-ff6c7b302675_evhx1q0z)

### Gateway vs. interface endpoint decision criteria

Gateway endpoints were used for S3 and DynamoDB because they integrate directly with route tables, eliminate NAT traversal costs, and keep traffic entirely on the AWS backbone.

Interface endpoints were reserved for scenarios requiring private connectivity to services beyond S3 and DynamoDB, including cross-VPC access, on-premises integration, security-group enforcement, and PrivateLink-based service consumption.

## Cross-Account Governance and Bidirectional Hybrid DNS

### AWS RAM TGW sharing and Route 53 Resolver configuration

I shared the Transit Gateway across accounts using AWS RAM, then deployed Route 53 Resolver inbound and outbound endpoints to support bidirectional DNS resolution across AWS and on-premises networks.

The environment validated end-to-end DNS resolution through the VPN tunnel, proving workloads in AWS could resolve on-premises domains while on-premises systems could resolve private AWS records.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/487fa828-7a41-4a1d-8ae1-ff6c7b302675_bgkaqdi4)

### Inbound vs. outbound resolver endpoint direction

Inbound resolver endpoints allow external systems such as on-premises DNS servers to resolve private Route 53 records hosted inside AWS.

Outbound resolver endpoints perform the opposite role by forwarding DNS queries from AWS workloads toward external or on-premises DNS infrastructure. Hybrid architectures typically require both directions simultaneously to maintain consistent name resolution across environments.

## Proving Compliance with Reachability Analyzer and Data-Plane Tests

### Automated path validation as a compliance gate

I created Reachability Analyzer assertions for multiple critical traffic paths and intentionally introduced security-group drift to validate failure detection behavior.

The project also included live data-plane testing to confirm actual traffic behavior across gateway endpoints, PrivateLink services, hybrid VPN routing, and centralized firewall inspection workflows.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/487fa828-7a41-4a1d-8ae1-ff6c7b302675_1yq76wa6)

### ENI_SG_RULES_MISMATCH and security group drift detection

Reachability Analyzer evaluates the control-plane path hop by hop across route tables, NACLs, Transit Gateway attachments, and security groups without sending real packets.

ENI_SG_RULES_MISMATCH identified the exact network interface security-group rule responsible for blocking the path. This provided compliance-grade validation that the deny control itself enforced the block rather than the failure occurring elsewhere in the routing chain.

### Why static analysis and data-plane tests are both required

Reachability Analyzer validates whether traffic should theoretically succeed according to configuration state, but it cannot validate stateful firewall inspection behavior or application-layer responses.

Data-plane testing confirmed real operational behavior such as Suricata packet drops, S3 endpoint traffic, PrivateLink connectivity, and hybrid routing functionality. Both approaches are required because a green control-plane path alone is not sufficient for production sign-off.

## VPN Failover Drill: Proving BGP Reconvergence Under 60 Seconds

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/487fa828-7a41-4a1d-8ae1-ff6c7b302675_5bcssg8r)

### Measured convergence time and operational runbook data

I simulated a tunnel failure using ipsec auto --down tunnel1 and monitored BGP reconvergence behavior across the environment.

The measured reconvergence time was approximately 38 seconds from tunnel failure to successful packet recovery, remaining within the project’s 60-second recovery target.

Packet-loss observations during failover

During failover testing, three ICMP packets timed out before traffic resumed successfully through the secondary VPN tunnel.

Tunnel state validation confirmed that the failed tunnel transitioned to DOWN status while the secondary tunnel maintained four accepted BGP routes and preserved operational connectivity across the hybrid environment.

## Project Reflection and Key Takeaways

### Tools and concepts mastered

This project combined Terraform, AWS CLI, Transit Gateway, Site-to-Site VPN, Route 53 Resolver, AWS Network Firewall, PrivateLink, Reachability Analyzer, strongSwan, and FRRouting into a unified enterprise hybrid networking architecture.

The concepts reinforced throughout the build included hub-spoke segmentation, dynamic BGP routing, centralized inspection, cross-account governance, hybrid DNS, endpoint tuning, and compliance-oriented path validation.

### Time investment and challenges

This project required approximately six hours to complete successfully. The most difficult part was troubleshooting the Gateway Load Balancer and GENEVE-related inspection workflows, particularly around security-group behavior and validating packet traversal across the inspection chain.

Multiple routing, inspection, and failover scenarios required iterative validation before the architecture behaved consistently across all networking paths.

I completed this project to deepen my understanding of enterprise-grade hybrid cloud networking and scalable AWS network governance patterns.

The next area I want to sharpen is advanced intrusion detection and traffic-analysis workflows, particularly around distributed IDS systems, east-west traffic visibility, and automated threat detection inside large VPC environments.

---

*Built with [NextWork](https://learn.nextwork.org) - [View this project](https://learn.nextwork.org/projects/487fa828-7a41-4a1d-8ae1-ff6c7b302675)*
