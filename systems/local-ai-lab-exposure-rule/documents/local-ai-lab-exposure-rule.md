<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Secure a Local Kubernetes AI Lab

**Project Link:** [View Project](https://nextwork.ai/projects/afc592f0-72b6-49dd-b73b-3064769828dc)

**Author:** Roy Piring: Cloud Platform Engineer | Build Master  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/builtwithroy/uploads/afc592f0-72b6-49dd-b73b-3064769828dc_jn51iezk)

## Defining a Safer Local AI Lab

### The exposure problem and deterministic decision gate

I built a local kind lab that ran Ollama inside Kubernetes while making it reachable from the Windows host only through 127.0.0.1:21434. I then added a Go check that inspected the cluster configuration and Kubernetes Services before returning either PASS or OPEN. The verdict described network exposure, not model quality.

The local qwen3:1.7b model answered ten supplied questions and produced performance evidence, but it never decided whether the lab was safe. Go tests and explicit policy rules controlled that decision. This separation mattered because a model can answer questions well while the surrounding Kubernetes configuration exposes its port to other machines.

## Preparing a Reproducible Local Kubernetes Environment

### Setting up the lab foundation

I checked the existing tools before installing or changing anything, then created a clean laptop-ai-lab kind cluster. The required toolchain included Go, kind, kubectl, Git, GitHub CLI, Ollama with qwen3:1.7b, and Docker. After confirming those components, I ran go mod tidy and go run . up to prepare the application and cluster.

The Ollama Service started as ClusterIP on port 11434. The kind configuration mapped it to the Windows host through 127.0.0.1:21434, keeping the host-side listener on the loopback interface. This provided local access for the model tests without intentionally binding the port to every network interface on the laptop.

### Verified development environment

The verified workstation used Windows with PowerShell inside the editor terminal. The checks recorded Go 1.27.1, kind 0.33.0, kubectl 1.36.1, Git 2.55, GitHub CLI 2.101, Ollama 0.40.9, and Docker 29.8.2. GitHub CLI was already authenticated, and Ollama had the qwen3:1.7b model available.

Docker listed the required kind node image and ollama/ollama:0.40.8. Running go run . up completed with the cluster ready, confirming that the orchestration code could create the lab using the checked environment. Recording these versions made the setup easier to compare if a later run behaved differently on another workstation.

![Image](https://nextwork.ai/builtwithroy/uploads/afc592f0-72b6-49dd-b73b-3064769828dc_01h7hd0r)

![Image](https://nextwork.ai/builtwithroy/uploads/afc592f0-72b6-49dd-b73b-3064769828dc_4u05iq1x)

## Publishing a Security-First Open Source Design

### Publishing the design and contribution plan

I published the design before adding the exposure-check implementation. The first commit, Add the design docs, contained five documents describing the goals, architecture, policy, testing, and delivery plan. I then committed the license and contribution materials, including the required sign-off, before creating the public laptop-ai-lab repository.

Issue 1 was added to the public GitHub Project to connect the planned check with visible work tracking. Preserving the design as the oldest commit showed reviewers what the policy intended before they saw code written to satisfy it. This reduced the chance that the documentation would be rewritten afterward to match whatever behavior the implementation happened to produce.

### Service exposure decisions

The design required the model to run inside the cluster and be accessed from the host only through 127.0.0.1. It also treated an omitted kind listenAddress as 0.0.0.0, because kind’s default host binding accepts traffic on every interface. A port mapping using that default therefore received an OPEN result under the written policy.

The documented rules covered the safe loopback mapping, an open kind mapping, ClusterIP, and NodePort. They did not define whether a LoadBalancer Service or .spec.externalIPs should count as OPEN. Leaving those cases undocumented made them explicit policy gaps rather than silently assigning behavior that had not been reviewed.

![Image](https://nextwork.ai/builtwithroy/uploads/afc592f0-72b6-49dd-b73b-3064769828dc_2n8exr0n)

## Measuring Model Performance Without Surrendering Security Decisions

### Establishing the baseline and intended test failure

I ran qwen3:1.7b against the ten supplied questions before implementing the exposure policy. The model answered 6 of 10 correctly and generated output at 24.4 tokens per second. These values created a local model baseline, but they did not describe whether another device could connect to the Ollama port.

I then ran the supplied Go tests while checkExposure and runCheck were still empty starter functions. The suite failed intentionally because the policy code did not yet meet the test contract. Capturing this result showed that the tests were capable of rejecting the unfinished implementation before the later passing output existed.

![Image](https://nextwork.ai/builtwithroy/uploads/afc592f0-72b6-49dd-b73b-3064769828dc_ucenncbm)

### Why deterministic checks decide PASS or OPEN

The model score and token rate described how qwen3:1.7b handled a fixed set of questions on this machine. They could support decisions about model usefulness or performance, but they did not inspect the kind port mappings or live Service objects. A higher model score would not make an externally reachable port safer.

PASS or OPEN came from check.go, which read deterministic configuration values from the kind setup and Kubernetes Services. This allowed a score of 6/10 to appear beside a PASS verdict without conflict because the numbers answered different questions. The model generated answers, while the Go policy decided whether the lab matched the reviewed exposure rules.

## Implementing and Rebuilding the Exposure Check

### Building the deterministic Go policy check

I implemented checkExposure and runCheck so the supplied tests could evaluate the kind configuration and Service state using repeatable rules. After the implementation, go test reported ok laptop-ai-lab. Running go run . check then printed PASS for lab/ollama and returned exit status 0.

I stopped the lab and created it again to test whether the local image workflow could support another build. The second go run . up completed successfully without printing the original image-saving step. This gave me separate evidence for policy correctness and lab reconstruction instead of treating one passing check as proof of every part of the workflow.

![Image](https://nextwork.ai/builtwithroy/uploads/afc592f0-72b6-49dd-b73b-3064769828dc_2w289cah)

### Evidence of a reusable image and timed rebuild

The first go run . up output included saving ollama/ollama:0.40.8 and reported lab is up in 337.1 s. This showed that the initial setup performed the image-saving work before declaring the cluster ready. I retained the output rather than relying on a general claim that the image was prepared.

The second run reported lab is up in 228.0 s and did not contain the saving line. Within this workflow, the missing line was the recorded evidence that the previously saved archive was reused instead of being saved again. The shorter time supported that observation, but the console output was the direct evidence used by the close-out.

## Proving the Rule Against Safe and Open Service States

### Testing safe, exposed, and restored states

I completed each test-plan row with the exact terminal result produced by the corresponding state. With the Ollama Service left as ClusterIP and the host mapping bound to loopback, the check printed PASS and returned exit status 0. This represented the intended local-only configuration under the written rules.

The open kind mapping and planted NodePort Service each caused the check to print OPEN and return exit status 1. After removing the NodePort Service, the result returned to PASS with exit status 0. Testing safe, exposed, and restored states showed that the verdict changed with the inspected configuration instead of remaining fixed after the first run.

![Image](https://nextwork.ai/builtwithroy/uploads/afc592f0-72b6-49dd-b73b-3064769828dc_z4yezz8a)

### Live evidence from ClusterIP and NodePort changes

I applied testdata/ollama-nodeport.yaml, creating the planted Service lab/ollama-open. Running go run . check inspected the live Service, printed OPEN for that object, and returned exit status 1. This demonstrated that the policy recognized a Service type designed to expose a port through the Kubernetes nodes.

I deleted the same Service using its manifest and ran the check again. The command then printed PASS for lab/ollama and returned exit status 0. Watching PASS change to OPEN and back to PASS proved that the checker read current cluster state. The result was not a cached verdict from the previous invocation or a hard-coded response tied to the test filename.

## Closing the Open Source Contribution Loop

### Documented coverage, limits, and responsible contribution

The accepted evidence showed PASS for the safe ClusterIP state and OPEN with exit status 1 for both planted exposure paths. Deleting the NodePort Service restored PASS, which confirmed that the policy followed the live Service state. I preserved those results in the contribution trail so reviewers could see the healthy path, failure paths, and recovery.

I also documented what the checker did not cover. The current release did not evaluate hostNetwork, hostPort, Ingress resources, LoadBalancer, .spec.externalIPs, or a manually started port-forward bound to 0.0.0.0. Naming those limits prevented v0.1.0 from being presented as a complete Kubernetes network-security scanner.

## Reflecting on the Tools, Evidence, and Learning Journey

### Key tools and concepts

I used kind to create and rebuild the local Kubernetes cluster, Docker to run the nodes and Ollama image, and kubectl to inspect or change the Ollama Services. Go tests checked the policy logic, while go run . check evaluated the live cluster and returned PASS or OPEN with a matching process status.

Ollama and qwen3:1.7b handled only the ten-question model evaluation. Git and GitHub preserved the design-first history, contribution sign-off, issue, pull request, and release v0.1.0. The main concept was separation of duties: model output measured answer quality, while deterministic code and live Kubernetes evidence controlled the security verdict.

### Project duration and challenge

The work took about one hour by hand, matching the page estimate. That period included checking the toolchain, committing the design, creating the public repository, running the ten questions, implementing the Go policy, testing both exposure paths, restoring the safe state, rebuilding the lab, and reviewing the pull request.

The rebuild wait and pull-request verification were part of the same hour rather than uncounted overhead. However, I did not record a timestamp for every phase, so the hour remained an overall estimate. The most important procedural lesson was to capture phase start and finish times during the work instead of reconstructing the close-out duration afterward.

### Next learning goal

Seeing PASS change to OPEN when the NodePort Service appeared and return to PASS after deletion made the policy behavior easy to trust. I would keep the design-first commit order because it showed that the exposure rules existed before the implementation. That history is useful to maintainers reviewing why the check behaves as it does.

For the next iteration, I would record the clock at setup, design, model evaluation, implementation, negative testing, restoration, and release. I would also extend the reviewed policy to address the currently excluded exposure paths before adding code for them. That keeps new checks tied to written decisions rather than expanding coverage through undocumented assumptions.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/afc592f0-72b6-49dd-b73b-3064769828dc)*
