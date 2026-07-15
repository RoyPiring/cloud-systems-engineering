<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# AWS Bedrock Multi-Tenant Routing

**Project Link:** [View Project](https://nextwork.ai/projects/332b5f56-fdaa-494f-bce4-e5f3accc76f4)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/332b5f56-fdaa-494f-bce4-e5f3accc76f4_wfkxn3mz)

## The Architecture Challenge: One Endpoint for Any Model

### What this substrate prevents

In this build, I created a config-driven AWS Bedrock routing substrate. The system gives applications one endpoint while allowing the routed model to change through configuration instead of code.

The first incident pattern it prevents is hard-coded model ID churn. When model versions change, teams should not have to edit application code, redeploy handlers, or risk stale model references across tenant paths.

The second incident pattern it prevents is an uncontained throttle event. The substrate uses routing, fallback, and circuit breaker behavior so one degraded model path does not spread across the whole platform.

## Environment Setup and Preflight Verification

### Goals for this step

In this step, I confirmed the environment was ready for the AWS Bedrock routing build. I checked the required AWS permissions, verified model access, and prepared the Terraform directory structure.

The goal was to prove the account could call the required Bedrock models before building the router. That reduced the chance of debugging application logic when the real issue was missing model access or service permission.

I also scaffolded the Terraform project so the infrastructure could be declared and managed from the start. That kept the routing substrate tied to repeatable infrastructure instead of manual console setup.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/332b5f56-fdaa-494f-bce4-e5f3accc76f4_recaf1uc)

### Model access verification

I verified access to amazon.nova-micro-v1:0 and amazon.nova-lite-v1:0.

amazon.nova-micro-v1:0 handled fast, low-cost primary routing traffic. It was the right fit for simple or high-volume paths where speed and cost mattered most.

amazon.nova-lite-v1:0 acted as the stronger failover model when quality dropped or the circuit breaker opened. That gave the substrate a fallback path without forcing callers to know which model handled the request.

## Building the Routing Core with Converse API and AppConfig

### Goals for this step

In this step, I defined the core architecture and built the routing logic for the substrate. The goal was to let callers send requests through one API while the backend selected the active Bedrock model from configuration.

The router used the Converse API so the application could talk to Bedrock through a common interface. That kept the calling path stable even when the model changed behind it.

AppConfig carried the model routing rules. This separated runtime model choice from Lambda code, which made model swaps safer during model churn, segment changes, or failover events.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/332b5f56-fdaa-494f-bce4-e5f3accc76f4_ig56mwym)

### Proving end-to-end traceability

handler.py returned the modelId from AppConfig next to the Converse response text. That let callers see which model actually handled the request.

That proved the routing path was driven by configuration. The model was not hardcoded in the application handler.

It also made debugging and audit review easier when tenant segments changed models at runtime. A response could be traced back to the active AppConfig value instead of guessing which model the code called.

## Per-Segment Cost Attribution, HIPAA Region Pinning, and Model Registry

### Goals for this step

In this step, I added the cost, compliance, and model-management layer around the router. The work focused on application inference profiles, per-segment cost tags, healthcare region pinning, and a model registry path for versioned models.

Application inference profiles gave each segment its own attribution path. Tags such as Segment, CostCenter, and Environment let Bedrock usage flow into cost reporting without relying on weak log joins.

For the healthcare segment, the routing path had to stay pinned to the approved region. The audit script checked region eligibility so HIPAA-sensitive traffic did not drift into an unsupported region. I also prepared a SageMaker Model Registry pattern with three versioned models and a warm real-time endpoint so the substrate could support future model lifecycle control.

### Why inference profiles over log reconciliation

Application inference profiles carried tags for Segment, CostCenter, and Environment. Those tags flowed into Cost Explorer and the Cost and Usage Report for native per-segment Bedrock attribution.

That mattered because bill-grade attribution needs to come from the billing path, not from best-effort log analysis. A segment should not depend on perfect log joins to explain usage cost.

Log-based reconciliation could not reliably join every invoke to a cost center with billing accuracy. Inference profiles made attribution part of the invoke path instead of a cleanup step after the fact.

## Circuit Breaker, Blast Radius Containment, and Observability

### Goals for this step

In this step, I built the circuit breaker pattern for segment-level blast radius control. The design used DynamoDB for circuit state and AWS Step Functions Express for high-speed orchestration.

The DynamoDB table tracked circuit status by segment and used TTL attributes to clear open circuits after a 30-second window. That gave each segment a temporary protection state instead of a permanent failure mode.

The Step Functions Express workflow checked circuit state before invocation, handled throttle events, and routed fallback calls inside the two-second latency target. API Gateway was then updated to send requests through the orchestrated path so the circuit breaker could protect live traffic instead of sitting outside the main request flow.

### How the circuit breaker protects the platform

When the primary invoke failed after retries, the workflow opened the circuit by writing an open row to DynamoDB. It then called the segment’s fallback model instead of letting the failed primary path keep absorbing requests.

Recovery was driven by TTL. After roughly 30 seconds, the open circuit row expired, and the next request could try the primary model again.

While the circuit was open, the caller still received a normal HTTP 200 response with an answer. The response also included fallback: true and the fallback modelId, so the platform remained usable while still exposing that fallback routing occurred.

## End-to-End Validation and Documentation Artifacts

### Goals for this step

In this step, I validated the substrate across the four bars that mattered most: zero-code model swap, resilience, compliance, and latency. The validation had to prove that the router worked under runtime changes, failure paths, regulated routing rules, and live request load.

The zero-code swap tested whether a segment could change models through AppConfig without changing Lambda code. The resilience test checked whether an open circuit triggered fallback routing fast enough for the caller. The compliance test verified healthcare region pinning. The latency test confirmed that the API stayed within the target response window.

I also finalized the documentation artifacts. The technical guide explained how the substrate works, the architecture diagram showed the request and control paths, and the review deck translated the technical results into a leadership-ready readout.

### Four validation bars: swap, resilience, compliance, latency

The zero-code swap passed. The Hospitality segment was changed in AppConfig to amazon.nova-lite-v1:0, and the invoke response returned that model ID without any Lambda code change.

The resilience bar passed. The Retail circuit was seeded open, and fallback completed in 1.28 seconds with fallback: true and amazon.nova-lite-v1:0.

The compliance and latency bars also passed. Healthcare Converse traffic stayed in us-east-1, and audit-region.sh returned PASS: Zero HIPAA region violations detected. The latency test ran 10 invokes across 4 segments for 40 total requests. All returned HTTP 200, and API Gateway p95 latency was 868.72 ms, below the 3000 ms target.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/332b5f56-fdaa-494f-bce4-e5f3accc76f4_hgedqjzm)

### Hardest validation and lessons learned

Compliance took the longest to pass. The first reason was that CloudTrail needed time before the relevant events were available for audit.

The second reason was that the first audit looked for InvokeModel, but the substrate used Converse. The script also had to handle empty event results without failing incorrectly.

After the script was fixed and the healthcare profile was tested again, the audit passed. That made compliance slower than the swap, resilience, and latency bars, but it also made the validation stronger because the script matched the real Bedrock call path.

## Secret Mission: Intelligent Prompt Routing to Reduce Spend

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/332b5f56-fdaa-494f-bce4-e5f3accc76f4_a3matw8t)

### Routing simple vs complex prompts to reduce spend

Intelligent Prompt Routing sent simple Hospitality prompts to a cheaper model when predicted quality was close enough. That reduced spend on simple traffic without sending every request to the more capable model.

Complex prompts still went to the stronger model. That protected quality where the prompt required more reasoning, nuance, or higher answer quality.

The routing charge was $1 per 1,000 requests and only applied on invoke. The lab estimate showed up to about 30% token savings on simple traffic, which made prompt routing useful when the traffic mix included enough low-complexity requests.

## Reflections and Takeaways

### Tools and concepts mastered

The key tools I used included Terraform for infrastructure provisioning, AWS Bedrock with the Converse API and Prompt Router for model management, AWS Lambda for serverless routing logic, EventBridge and Step Functions for circuit breaker flow, AppConfig for dynamic routing configuration, and CloudWatch for observability and metrics.

The main concepts I learned included config-driven routing, application inference profiles for per-segment cost attribution, circuit breakers for blast radius control, and region pinning through Infrastructure as Code for regulated workloads.

The larger lesson was that model routing should be treated like platform infrastructure. A reliable substrate needs configuration control, traceability, fallback behavior, billing attribution, compliance checks, and validation evidence.

### Time and challenges

This build took me approximately 3 hours. That time covered environment verification, model access checks, Terraform scaffolding, Converse routing, AppConfig integration, inference profiles, region pinning, circuit breaker logic, validation, and documentation.

The hardest part was configuring Intelligent Prompt Routing so the logic consistently separated simple prompts from complex prompts. The threshold had to line up with the AppConfig deployment and the Lambda handler so routing decisions behaved as expected.

The compliance audit was also a real challenge because the first script checked for the wrong event type. Fixing the script to match Converse made the validation honest instead of forcing the evidence to fit the first assumption.

I completed this build to gain hands-on experience building a config-driven routing substrate for AWS Bedrock. The system showed how to route multi-tenant traffic, contain model failures, attribute cost by segment, and apply region controls for regulated use cases.

The next skill I want to build is connecting this routing pattern to advanced RAG workflows. That would let the substrate control both model selection and retrieval paths while keeping cost, response quality, and compliance boundaries under control.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/332b5f56-fdaa-494f-bce4-e5f3accc76f4)*
