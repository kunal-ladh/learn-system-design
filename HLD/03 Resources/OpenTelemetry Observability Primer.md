---
title: OpenTelemetry Observability Primer
type: resource
resource_type: documentation
tags: [resource, documentation]
url: https://opentelemetry.io/docs/concepts/observability-primer/
author: "OpenTelemetry project"
year: ""
status: unread
priority: medium
related_concepts: ["[[Distributed Tracing]]", "[[Tracing]]", "[[Logging]]"]
related_problems: ["[[Design a Logging System]]", "[[Design a Metrics and Monitoring System]]"]
summary: "The vendor-neutral definitions of observability, reliability, SLIs/SLOs, logs, spans and distributed traces, and how the three signals relate."
created: 2026-11-03
updated: 2026-11-03
---

# OpenTelemetry Observability Primer

> [!abstract] Why this resource
> It gives one consistent vocabulary for [[Logging]], [[Metrics]] and [[Distributed Tracing]], which the interview question 'how would you debug a slow request across five services?' needs. It is short and free of vendor detail.

**Link:** [https://opentelemetry.io/docs/concepts/observability-primer/](https://opentelemetry.io/docs/concepts/observability-primer/)  
**Type:** Primary documentation · **Author:** OpenTelemetry project

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Official docs page, free. Link fetched and verified 2026-11-03.

## When to read it

Whole page (fifteen minutes) in week 11 of [[Start Here]]; then instrument one project with the OTel Java agent and look at a real trace.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Observability lets you ask questions about a system without knowing its internals — the 'unknown unknowns' framing.
- Reliability is about doing what users expect; a system can be up 100% of the time and still unreliable.
- An SLI measures behaviour from the user's perspective; an SLO ties SLIs to business value.
- A log on its own lacks context; it becomes useful when attached to a span or correlated with a trace and span ID.
- A span is one unit of work with a name, timing, structured events and attributes such as http.request.method.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Audit Logs]]
- [[Distributed Tracing]]
- [[Logging]]
- [[Metrics]]
- [[Monitoring User-Facing Performance]]
- [[Tracing]]

### Design problems

- [[Design a Logging System]]
- [[Design a Metrics and Monitoring System]]
