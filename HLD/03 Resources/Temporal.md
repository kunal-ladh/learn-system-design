---
title: Temporal
type: resource
resource_type: open-source
tags: [resource, open-source]
url: https://github.com/temporalio/temporal
author: "Temporal Technologies"
year: ""
status: unread
priority: low
related_concepts: ["[[Saga Pattern]]", "[[Retry Strategies]]", "[[Idempotency]]"]
related_problems: ["[[Design a Distributed Job Scheduler]]"]
summary: "The durable-execution server (a fork of Uber's Cadence) that runs workflows and retries failed activities — the production shape of [[Design a Workflow Orchestrator]]."
created: 2026-11-03
updated: 2026-11-03
---

# Temporal

> [!abstract] Why this resource
> Running it locally for an afternoon shows what 'durable workflow' means in practice: retries, timers and history survive process restarts. Low priority for interviews; useful before building [[Durable Job Scheduler]].

**Link:** [https://github.com/temporalio/temporal](https://github.com/temporalio/temporal)  
**Type:** Open-source implementation · **Author:** Temporal Technologies

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Public GitHub repository (MIT). Link fetched and verified 2026-11-03.

## When to read it

README quick start (temporal server start-dev) plus docs/architecture after [[Temporal Docs on Durable Execution]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- The server executes workflows and automatically handles intermittent failures and retries.
- A fork of Uber's Cadence by Cadence's creators.
- Local start: brew install temporal, temporal server start-dev, Web UI on localhost:8233.
- The repository is only the server; workflows, activities and workers are written with language SDKs (Go, Java, …).
- MIT licence; the architecture README under docs/ describes the service components.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Distributed Transactions]]
- [[Idempotency]]
- [[Retry Strategies]]
- [[Saga Pattern]]

### Design problems

- [[Design a Distributed Job Scheduler]]
