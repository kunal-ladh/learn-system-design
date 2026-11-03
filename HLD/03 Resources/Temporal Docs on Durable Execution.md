---
title: Temporal Docs on Durable Execution
type: resource
resource_type: documentation
tags: [resource, documentation]
url: https://docs.temporal.io/evaluate/understanding-temporal
author: "Temporal Technologies"
year: ""
status: unread
priority: low
related_concepts: ["[[Saga Pattern]]", "[[Event-Driven Architecture]]", "[[Retry Strategies]]"]
related_problems: ["[[Design a Distributed Job Scheduler]]", "[[Design a Workflow Orchestrator]]"]
summary: "Temporal's conceptual model — deterministic workflows, activities for side effects, workers polling task queues, and replay from event history — as a worked example of durable execution."
created: 2026-11-03
updated: 2026-11-03
---

# Temporal Docs on Durable Execution

> [!abstract] Why this resource
> [[Design a Workflow Orchestrator]] and [[Design a Distributed Job Scheduler]] are easier once you have seen one real orchestrator's mental model. The determinism rule and replay-based recovery explain why sagas and retries can be written as plain code.

**Link:** [https://docs.temporal.io/evaluate/understanding-temporal](https://docs.temporal.io/evaluate/understanding-temporal)  
**Type:** Primary documentation · **Author:** Temporal Technologies

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Official docs page, free. Link fetched and verified 2026-11-03.

## When to read it

The Workflows, Workers (task queues) and Event History sections after [[Saga Pattern]]; about thirty minutes.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- A workflow must be deterministic because Temporal reruns it to rebuild state; anything that touches the outside world belongs in an activity.
- Workers poll named task queues — nothing is pushed to a worker, so a saturated worker's tasks simply wait.
- Workflow code records commands; the worker sends them when the workflow task completes and the history service appends the resulting events.
- Recovery is a new worker fetching the event history and replaying the workflow code to rebuild local state.
- The service is several components behind one gRPC endpoint (frontend, history, matching, worker) over Cassandra, MySQL, PostgreSQL or SQLite; worker versioning pins executions to the code they started on.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Event-Driven Architecture]]
- [[Idempotency]]
- [[Retry Strategies]]
- [[Saga Pattern]]

### Design problems

- [[Design a Distributed Job Scheduler]]
- [[Design a Workflow Orchestrator]]
