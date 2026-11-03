---
title: BullMQ
type: resource
resource_type: open-source
tags: [resource, open-source]
url: https://github.com/taskforcesh/bullmq
author: "Taskforce.sh"
year: ""
status: unread
priority: medium
related_concepts: ["[[Message Queues]]", "[[Retry Strategies]]", "[[Dead-Letter Queues]]"]
related_problems: ["[[Design a Notification System]]"]
summary: "A Redis-backed job queue for Node.js (with other language ports): delayed and repeatable jobs, priorities, rate limiting, parent–child flows and retries with backoff."
created: 2026-11-03
updated: 2026-11-03
---

# BullMQ

> [!abstract] Why this resource
> It is the queue you would reach for in a TypeScript service, and reading how it models retries, delays and failed jobs on Redis is a compact lesson in [[Message Queues]] built on [[Redis]].

**Link:** [https://github.com/taskforcesh/bullmq](https://github.com/taskforcesh/bullmq)  
**Type:** Open-source implementation · **Author:** Taskforce.sh

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Public GitHub repository (MIT); docs at docs.bullmq.io. Link fetched and verified 2026-11-03.

## When to read it

README quick start and the feature table with [[Message Queues]]; use it for the delivery loop in [[Personal Notification Hub]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Queue and Worker API: queue.add(name, data) and a Worker that processes jobs by name.
- FlowProducer adds jobs with parent–child dependencies across queues.
- Open-source features include deduplication, priorities, delayed jobs, a rate limiter, sandboxed workers and repeatable jobs; grouping and batches are Pro-only.
- Ports for Python, Rust, Elixir, .NET and PHP; requires Redis 5+.
- MIT licence.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Dead-Letter Queues]]
- [[Message Queues]]
- [[Redis]]
- [[Retry Strategies]]

### Design problems

- [[Design a Notification System]]
