<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Build a Workstation Readiness Doctor

**Project Link:** [View Project](https://nextwork.ai/projects/f7974606-9c5b-4f8a-9063-51e465937568)

**Author:** Roy Piring: Cloud Platform Engineer | Build Master  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/f7974606-9c5b-4f8a-9063-51e465937568_zpvtjsz1)

## Establishing a Portfolio-Ready Workstation Plan

### Tested platform and development workflow

I built and tested the workstation doctor on Windows using Cursor as the primary development environment. The implementation used Go to produce a compiled CLI that could inspect workstation capabilities without changing the installed tools. I treated Windows as the verified platform while keeping the check architecture suitable for later operating-system-specific implementations.

This work belonged in my portfolio because it demonstrated more than dependency installation. I defined readiness policy, encoded ten checks, enforced version requirements, added command timeouts, returned truthful process status, and documented the implementation through Git history and a merge-based pull request. The result showed how I approached developer enablement as an engineering control. I did not claim complete cross-platform support from one Windows run. That claim would require executing equivalent checks and failure cases on macOS and Linux.

## Preparing a Trusted Cross-Platform Toolchain

### Day Zero setup goals

I defined the acceptance criteria and technical decisions before implementing the workstation doctor. The contract stated which ten engineering capabilities needed checks, how versions would be evaluated, when commands should time out, and what exit status the process must return. This prevented the implementation from setting its own standard after the current workstation state was known.

I also documented the expected command behavior, output format, supported platform, failure conditions, and handover requirements. The design separated observation from mutation: the doctor could inspect installed software and configuration but could not install, upgrade, or repair anything automatically. This kept early runs reversible because a failed check changed no workstation state. Day Zero ended with a documented baseline that could guide implementation and later review. The criteria could still change, but any change would require an explicit decision rather than an unnoticed code edit.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/f7974606-9c5b-4f8a-9063-51e465937568_lfcs967x)

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/f7974606-9c5b-4f8a-9063-51e465937568_3mxip5pz)

### Installer provenance and safety checks

I downloaded installers only from the official vendor pages for Docker, Ollama, Rust, Git for Windows, Go, and Node.js. I saved remote scripts such as install.ps1 and rustup-init.sh locally so I could inspect them before execution. I did not pipe downloaded scripts directly into a shell because that would combine retrieval and execution without a review point.

I compared each installer or archive with the SHA-256 value published by its vendor in files such as checksums.txt, sha256sum.txt, rustup-init.exe.sha256, and SHASUMS256.txt. A matching digest was required before installation. WinGet also reported matching package hashes for Task and GitHub CLI. When the Go and Node installers under Program Files required unavailable administrator rights, I used the official ZIP archives only after their hashes passed. These controls proved file consistency with published digests, not that vendor software was free of defects.

## Designing the Doctor for Safe, Reversible Decisions

### Designing the first visible check

I designed the first check as a complete example of the doctor’s contract rather than a one-off command. It needed a capability name, expected policy, command invocation, parser, timeout behavior, visible result, and process-level outcome. This established the pattern that the remaining checks would follow and exposed architectural gaps before the system expanded.

The check reported PASS or FAIL with enough context to explain what was observed and what the policy required. It did not install missing software, rewrite PATH, or change configuration. That read-only behavior kept diagnosis separate from repair and allowed the doctor to run safely on an unfamiliar workstation. I also documented the decision in an Architecture Decision Record with its tradeoffs and reversal condition. The first visible result therefore tested both user-facing output and the internal contract that later checks needed to satisfy without duplicating control flow.

### Using reversal triggers to manage design risk

I added a reversal trigger to each important design decision so the implementation documented the condition that would make its current approach unsuitable. A trigger converted a permanent-looking choice into a conditional one. For example, a read-only diagnostic model could be reconsidered if users later required an approved repair mode with reliable rollback.

Writing the trigger in advance preserved the reason behind the decision and reduced the chance that the code would continue unchanged only because its original context had been forgotten. It also gave future maintainers a shared test for change. When the condition occurred, they could revisit the decision using evidence rather than restarting the argument from memory. A reversal trigger did not automatically replace the design, and it did not predict every future requirement. It made known risk visible, defined when review was required, and kept architectural decisions from becoming silent habits.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/f7974606-9c5b-4f8a-9063-51e465937568_oi55kc6g)

## Scaling the Doctor into a Ten-Check Validation System

### Building a table-driven check loop

I represented the ten workstation checks as data and executed them through one shared loop. Each table entry contained the check name, command or inspection function, expected policy, and result handler. This allowed the doctor to apply the same timeout, formatting, evidence capture, and status aggregation rules without copying control logic for every dependency.

The table-driven structure separated what the doctor checked from how it executed checks. Adding a capability required a new entry and its focused parser rather than another full command pipeline. The loop collected every result so one early failure did not hide the state of later tools. After all entries ran, the process summarized the complete workstation condition and returned an aggregate status. This design reduced inconsistent behavior between checks and made the policy list easier to review. It did not remove tool-specific logic; version formats and configuration evidence still required dedicated parsing and tests.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/f7974606-9c5b-4f8a-9063-51e465937568_h96dj8rk)

### Why truthful process status matters

A printed FAIL represented a failed readiness check, so the process needed to communicate the same outcome to the shell. Exit code 0 conventionally indicates successful execution. If the doctor printed that Helm or Git line-ending policy had failed but still returned 0, a person would see a problem while automation would treat the workstation as ready.

That disagreement would make the CLI unreliable in Task, scripts, and CI. Downstream commands could continue after a missing dependency, unsupported version, or failed policy check. I therefore separated successful program execution from successful readiness evaluation: the doctor could complete all checks without crashing and still return a non-zero status because the environment failed its contract. This made the visible summary and machine-readable result consistent. The shell did not need to parse human text to understand whether the workstation passed, and the log retained the individual failures needed for repair.

## Hardening Automation with Timeouts and Reliable Failure Evidence

### Adding version gates, timeouts, and aggregate results

I added version gates so each relevant tool was compared with the expected major version rather than merely detected on PATH. This distinguished presence from policy compliance. A dependency could exist and run normally but still fail readiness because its installed major version differed from the version approved by the workstation contract.

Every external command received a five-second deadline. A blocked executable, stalled network call, or broken configuration could therefore fail with timeout evidence instead of hanging the entire doctor. I allowed all ten checks to finish and aggregated their results so the user received one complete diagnosis. The process returned 0 only when every required check passed and returned a non-zero status when any check failed or timed out. This aligned interactive output with automation behavior while keeping the doctor read-only. The implementation observed version drift and command failure without changing installed software.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/f7974606-9c5b-4f8a-9063-51e465937568_6oxx6rol)

### Making failures actionable for automation

I treated any failed required check as evidence that the workstation was not ready under the declared contract. The doctor still completed the remaining checks, printed each result, and produced an aggregate summary. It then returned a non-zero exit status so the calling shell, Task command, or CI job could stop or route the failure for repair.

This behavior made failures useful without giving the doctor authority to modify the workstation. A FAIL identified the capability, observed value, and expected policy, while the exit code communicated the overall decision to automation. Exit code 0 remained reserved for a complete pass. This prevented scripts from continuing as though Helm, Git line endings, or another required capability were healthy when the log showed otherwise. The doctor therefore served both audiences: people received readable evidence, and automation received a standard process signal that did not depend on scraping formatted output.

## Publishing a Complete Engineering Handover

### Turning local engineering work into public delivery evidence

I completed the README, design records, usage instructions, acceptance criteria, validation evidence, and release notes before publishing the repository to GitHub. The handover explained what the doctor checked, how to run it, how version gates behaved, what the exit statuses meant, and which platform had been tested. This allowed another engineer or hiring manager to inspect the implementation without relying on a live explanation.

I preserved the development sequence in Git so the repository showed the design baseline, table-driven expansion, timeout and status hardening, documentation, and final merge. The public evidence demonstrated how the CLI evolved in response to explicit requirements. I kept unsupported claims out of the handover: the tool was tested on Windows, while cross-platform architecture remained a design property awaiting execution on macOS and Linux. Publication turned the local build into a reviewable engineering artifact rather than proof that every workstation could pass.

### Closing the issue through a merge-based pull request

I ended the pull-request body with Closes #1, connecting the implementation directly to its tracked GitHub issue. GitHub processed that keyword when the pull request merged into main and closed the issue as part of the repository workflow. This tied completion to merged code rather than a manual status change disconnected from delivery.

I used a merge commit through --merge instead of squashing the branch. The resulting history retained the individual growth commits for the design baseline, table-driven loop, timeout and exit-status controls, handover documentation, and final integration. That sequence gave reviewers more context about how the implementation developed and which commit introduced each control. A merge-based history was useful for this evidence package, though it was not inherently better for every repository. The choice reflected the goal of preserving reviewable development stages rather than minimizing the number of commits on main.

## Proving Version Drift Is Caught Safely

### Testing pin drift without changing installed software

I tested version-policy drift by changing only the Node.js expectedMajor value inside the doctor. Node.js 24.21.0 remained installed, stayed on PATH, and continued functioning normally. The next doctor run failed because the observed major version no longer matched the modified policy value, proving the gate compared the environment against an expectation rather than echoing whatever version it found.

This test was safer than replacing the runtime. Installing another Node.js version would have required a download, a PATH change, and a rollback plan if other tools stopped working. Editing one local expectation created a controlled, reversible mismatch. I restored the expected major to 24 after capturing the failure evidence, returning the policy to its original state without altering the workstation. The experiment proved that the doctor could detect pin drift; it did not prove that every possible version string or package manager layout was parsed correctly.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/f7974606-9c5b-4f8a-9063-51e465937568_fblltx6s)

## Reflecting on Workstation Engineering Skills

### Tools and concepts developed

I used Go to build the compiled CLI, Task to provide repeatable developer commands, Git to preserve implementation history, and GitHub to manage the repository, issue, pull request, and release. The doctor combined command execution, version parsing, five-second deadlines, table-driven checks, aggregate reporting, and standard exit statuses into one read-only diagnostic workflow.

The central lesson was that workstation readiness required policy, not only tool detection. A dependency on PATH could still be unsupported, stalled, or incorrectly configured. Version gates made expectations explicit, while non-zero exits made failures visible to automation. The pin-drift experiment proved that the doctor enforced policy without replacing installed software. I also practiced documenting design decisions with reversal triggers and packaging the implementation as an engineering handover whose claims could be checked from the repository.

### Completion time and challenges

I completed the build in approximately 65 minutes. The hardest part was understanding how to test the Node.js version gate without disrupting the working environment. Changing the installed runtime would have introduced download, installation, PATH, and rollback risks unrelated to the policy behavior I wanted to measure.

I instead changed the expected major version in the doctor, ran the check, and observed the intended failure while Node.js 24.21.0 remained untouched. Restoring the expected value to 24 reversed the experiment. This clarified the distinction between environment drift and policy drift: the same workstation could pass or fail depending on the declared requirement, even though its installed tools had not changed. The exercise also reinforced why failure evidence needed both the observed version and expected version. A bare FAIL would identify a problem but would not explain the mismatch or guide correction.

### Next learning goal

I completed this build to learn how to create a cross-platform workstation-readiness CLI in Go with explicit version policy, command deadlines, aggregate results, and truthful exit statuses. The implementation gave me practical experience separating diagnosis from repair, scaling checks through a data table, and preserving design and delivery evidence through Git and GitHub.

My next goal is to add automated unit and integration testing. Unit tests should cover version parsing, result aggregation, timeout classification, and platform-specific policy decisions without executing real installers. Integration tests should run the compiled doctor against controlled fixtures or containers that contain known passing, missing, outdated, and stalled commands. I also want CI jobs for Windows, macOS, and Linux so the cross-platform design can be validated through actual execution rather than inferred from the Go code alone.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/f7974606-9c5b-4f8a-9063-51e465937568)*
