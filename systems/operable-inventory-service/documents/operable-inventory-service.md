<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Build an Operable Service

**Project Link:** [View Project](https://nextwork.ai/projects/d93f4fe8-53ec-41fa-8e8a-7799332c614e)

**Author:** Roy Piring Jr: Sr. Cloud Engineer | Architect  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/d93f4fe8-53ec-41fa-8e8a-7799332c614e_o875gpyt)

## Building an Operable Inventory Service

### Project overview

I built a FastAPI service that runs through Docker Compose, connects to PostgreSQL, emits structured logs, and exposes health checks that separate process health from dependency readiness.

The goal was to prove operability, not just functionality. A service that can start is not enough. It also needs clear configuration boundaries, honest health signals, and trace evidence that explains what happened during each request.

This mattered because production services fail in ways that code alone does not explain. The build showed how to make the service easier to operate, debug, and trust.

### Design before build

In this step, I defined the technical architecture before implementing the service. I drafted three MADR-compliant decision records and a C4 container diagram.

The decision records explained the technology choices and the trade-offs behind them. The C4 diagram showed how the FastAPI app, PostgreSQL database, Docker Compose environment, Jaeger, and OpenTelemetry pieces connected.

This mattered because operability starts with design clarity. The architecture needed a written rationale and a visual model before code and containers were wired together.

## Drafting and Correcting AI-Generated Design Records

### Reviewing the ADR draft

The ADR review corrected the claim about Factor IV and SQLite. SQLite through a configurable path can still be attached to a service, so the real issue was not that SQLite breaks Factor IV.

The better distinction was that SQLite does not fit the same multi-writer or network-service shape as PostgreSQL. That made the trade-off more precise.

I also added the short-lived lab trade-off for PostgreSQL-specific SQL and operational habits. The reversal trigger was changed to a measurable one: two failed lab or CI runs, or more than 3 minutes waiting on PostgreSQL.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/d93f4fe8-53ec-41fa-8e8a-7799332c614e_edatgy6n)

## Externalizing Config and Proving It Works

### Implementing Twelve-Factor config

In this step, I externalized the service configuration into environment variables. The goal was to keep credentials, connection strings, and runtime settings out of source code.

The service used DATABASE_URL from the environment so the backing database could change without editing main.py. That made the build environment-driven instead of code-bound.

I also added structured JSON logging so logs could be treated as an event stream. Together, the config and logging work aligned the service with Twelve-Factor principles before the changes were committed.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/d93f4fe8-53ec-41fa-8e8a-7799332c614e_kyg90sao)

### Proving environment-driven config

I changed the Compose DATABASE_URL host from postgres to does-not-exist without changing anything in main.py.

When the service restarted, it failed with failed to resolve host 'does-not-exist' and exited.

That proved the app used the environment value for the database connection. There was no hardcoded fallback hiding inside the code.

## Implementing Honest Health Signals

### Separating liveness from readiness

In this step, I added a liveness endpoint at /healthz/live. The endpoint let the service report that the process was alive separately from whether external dependencies were available.

This separation mattered because an orchestrator needs different answers for different decisions. A live process should not be killed just because PostgreSQL has a temporary issue.

The readiness check handled traffic eligibility. The liveness check handled process survival.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/d93f4fe8-53ec-41fa-8e8a-7799332c614e_zz42yy9r)

### The design decision behind two endpoints

Liveness asks, “is the process alive?” Readiness asks, “can it serve traffic right now?”

A single /health endpoint that checks the database can turn a PostgreSQL blip into a restart storm. The orchestrator may kill a healthy process because one dependency is temporarily unavailable.

Split probes prevent that failure mode. Readiness can return 503 and drain traffic while liveness stays 200, giving the app room to recover without being restarted.

## Wiring Distributed Tracing with OpenTelemetry

### Auto-instrumenting the service

In this step, I wired OpenTelemetry auto-instrumentation into the FastAPI service. The goal was to trace request lifecycles instead of relying only on logs.

Tracing showed how a request moved through the HTTP layer, handler, and database work. That made failures easier to inspect because each span showed part of the path.

This mattered because logs can show that something happened, but traces show where time and errors moved through the service.

### Full proof sequence results

The service stayed alive and responded quickly through /healthz/live, well under 200 ms, even when PostgreSQL was down.

Jaeger showed inventory-service traces with a parent-child span chain across HTTP, handler, and database activity. That proved requests were actually instrumented.

Readiness also stayed separate from liveness. When the database was down, readiness returned 503 while liveness stayed 200. When PostgreSQL returned, readiness healed without restarting the app.

## Scoring the AI and Writing the Teach-Back

### Auditing against the sealed key

In this step, I attempted to score the AI audit against the sealed factor key. The goal was to check whether the audit correctly identified Twelve-Factor compliance and gaps.

The scoring depended on the sealed ground truth file being present. Without that key, the audit could not be scored against a fixed answer set.

This was an important honesty point. The service could still be reviewed, but the AI scoring claim could not be completed without the missing sealed file.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/d93f4fe8-53ec-41fa-8e8a-7799332c614e_09ucbimr)

### AI accuracy score and missed findings

The AI accuracy score was not available because Confirmed, Contradicted, and Missed findings were never counted.

The sealed ground truth file .factor-key.sealed was missing, so the audit could not be scored.

No content-level miss against the key could be named. The recorded miss was that scoring against the key never happened.

## Closing the Repo and Proving from a Clean Clone

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/d93f4fe8-53ec-41fa-8e8a-7799332c614e_wnpyyby9)

### Clean-clone proof and feasibility record

The clean-clone proof took 1 minute and 9 seconds, from 21:08:17Z to 21:09:26Z.

docs/feasibility.md recorded the clone URL, the wall-clock time, and the three proof results: live 200, a Jaeger trace with 5 spans, and dead-database behavior with live 200 and ready 503.

The feasibility record also noted that Docker reused a local pip layer cache on this machine. That made the timing honest because the run was not a fully cold dependency build.

## Reflection and Key Takeaways

### Tools and concepts learned

The key tools I used included FastAPI for the service, Docker Compose for container orchestration, PostgreSQL for data storage, Jaeger for trace viewing, and OpenTelemetry for instrumentation.

The main concepts I learned included externalized configuration, separate liveness and readiness probes, structured JSON logging, distributed tracing, and clean-clone proof as an operability check.

I also learned the value of documentation such as AARs, honesty notes, and feasibility records. Those artifacts help stakeholders understand what was proven, what failed, and what limits still remain.

### Time and challenges

This build took me approximately 65 minutes. That time covered design records, the C4 diagram, environment-driven configuration, structured logging, health endpoints, tracing, AI audit scoring attempt, and clean-clone proof.

The hardest part was configuring the environment variables so the FastAPI service communicated correctly with the PostgreSQL container. The second hard part was making the health endpoints reflect database state correctly during the dead-database test.

I completed this build to learn the fundamentals of an operable service: externalized configuration, independent liveness and readiness signals, and end-to-end distributed tracing. Next, I want to apply these observability and reliability patterns at scale through horizontal autoscaling and advanced load testing in production environments.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/d93f4fe8-53ec-41fa-8e8a-7799332c614e)*
