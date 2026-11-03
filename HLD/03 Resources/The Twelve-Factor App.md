---
title: The Twelve-Factor App
type: resource
resource_type: documentation
tags: [resource, documentation]
url: https://12factor.net/
author: "Adam Wiggins"
year: "2017"
status: unread
priority: low
related_concepts: ["[[Operational Simplicity]]", "[[Deployment and Rollback Strategies]]", "[[Secrets Management]]"]
related_problems: []
summary: "Twelve short rules for services that deploy cleanly — config in the environment, stateless processes, backing services as attached resources, logs as event streams."
created: 2026-11-03
updated: 2026-11-03
---

# The Twelve-Factor App

> [!abstract] Why this resource
> It is the baseline for 'operationally simple' in the Product Engineering notes and for everything you run on Kubernetes. Read once; most of it you already do, and the gaps are worth noticing.

**Link:** [https://12factor.net/](https://12factor.net/)  
**Type:** Primary documentation · **Author:** Adam Wiggins · **Year:** 2017

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free site; each factor has its own page. Link fetched and verified 2026-11-03.

## When to read it

The twelve factor pages (each is a few paragraphs) with [[Operational Simplicity]]; compare against one of your services at work.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- I Codebase: one codebase in revision control, many deploys. III Config: store config in the environment.
- IV Backing services: treat them as attached resources. VI Processes: execute the app as stateless processes.
- VIII Concurrency: scale out via the process model. IX Disposability: fast startup and graceful shutdown.
- X Dev/prod parity: keep environments as similar as possible. XI Logs: treat logs as event streams.
- XII Admin processes: run admin tasks as one-off processes; the background is experience with hundreds of apps on Heroku.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Build vs Buy Decisions]]
- [[Deployment and Rollback Strategies]]
- [[Encryption at Rest and in Transit]]
- [[Feature Flags]]
- [[Incremental Delivery]]
- [[Logging]]
- [[Operational Simplicity]]
- [[Secrets Management]]

### Design problems

- *(none link here yet)*
