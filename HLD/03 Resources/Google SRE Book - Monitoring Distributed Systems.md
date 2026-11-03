---
title: Google SRE Book - Monitoring Distributed Systems
type: resource
resource_type: deep-dive
tags: [resource, deep-dive]
url: https://sre.google/sre-book/monitoring-distributed-systems/
author: "Rob Ewaschuk (Google)"
year: "2017"
status: unread
priority: medium
related_concepts: ["[[Monitoring]]", "[[Metrics]]", "[[Alerting]]"]
related_problems: ["[[Design a Logging System]]", "[[Design a Metrics and Monitoring System]]"]
summary: "The four golden signals, symptoms versus causes, why averages hide tails, and the rules for what deserves a page — the chapter behind [[Monitoring]] and [[Alerting]]."
created: 2026-11-03
updated: 2026-11-03
---

# Google SRE Book - Monitoring Distributed Systems

> [!abstract] Why this resource
> When an interviewer asks 'what would you monitor?', latency-traffic-errors-saturation plus 'alert on symptoms, not causes' is the whole answer, and this chapter is where it comes from.

**Link:** [https://sre.google/sre-book/monitoring-distributed-systems/](https://sre.google/sre-book/monitoring-distributed-systems/)  
**Type:** Architecture deep dive · **Author:** Rob Ewaschuk (Google) · **Year:** 2017

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free chapter of the SRE book (CC BY-NC-ND). Link fetched and verified 2026-11-03.

## When to read it

The Four Golden Signals and Tying These Principles Together sections (twenty minutes) in week 11 of [[Start Here]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Four golden signals: latency, traffic, errors and saturation — if you can only measure four things, measure these.
- Alert on symptoms (HTTP 500s, slow responses); use causes (database refusing connections) for debugging, not paging.
- Mean latency hides tails: 100 ms average at 1,000 requests per second can still have 1% of requests taking 5 seconds; bucket latencies roughly by factors of 3.
- For a 99.9% service, probing more than once or twice a minute is probably unnecessary.
- Every page should be actionable and require intelligence; a page with a robotic response should not be a page.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Alerting]]
- [[Logging]]
- [[Metrics]]
- [[Monitoring]]
- [[Monitoring User-Facing Performance]]
- [[SLI SLO and SLA]]
- [[Time-Series Databases]]
- [[Tracing]]

### Design problems

- [[Design a Logging System]]
- [[Design a Metrics and Monitoring System]]
