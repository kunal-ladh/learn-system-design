---
title: Netflix - Fault Tolerance in a High Volume Distributed System
type: resource
resource_type: case-study
tags: [resource, case-study]
url: https://netflixtechblog.com/fault-tolerance-in-a-high-volume-distributed-system-91ab4faae74a
author: "Ben Christensen (Netflix)"
year: "2012"
status: unread
priority: medium
related_concepts: ["[[Circuit Breakers]]", "[[Bulkheads]]", "[[Graceful Degradation]]"]
related_problems: []
summary: "The post that introduced Netflix's approach to isolating dependency failures — timeouts, bounded thread pools per dependency, circuit breakers and fallbacks — which became Hystrix."
created: 2026-11-03
updated: 2026-11-03
---

# Netflix - Fault Tolerance in a High Volume Distributed System

> [!abstract] Why this resource
> It is the origin story of the pattern set behind [[Circuit Breakers]] and [[Bulkheads]], told with the arithmetic that makes it urgent (dozens of dependencies at 99.99% each still fail you hours a month). Pair it with the Hystrix wiki for the mechanics.

**Link:** [https://netflixtechblog.com/fault-tolerance-in-a-high-volume-distributed-system-91ab4faae74a](https://netflixtechblog.com/fault-tolerance-in-a-high-volume-distributed-system-91ab4faae74a)  
**Type:** Engineering case study · **Author:** Ben Christensen (Netflix) · **Year:** 2012

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Netflix Tech Blog on Medium. Medium refuses automated fetches, so the URL was confirmed through the search index on 2026-11-03 (title and slug match); open it in a browser. The Hystrix wiki is the fetchable companion.

## When to read it

Whole post (fifteen minutes) in week 9 of [[Start Here]]; then the 'How it Works' page of the Hystrix wiki (github.com/Netflix/Hystrix/wiki/How-it-Works) for the diagrams.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- The dependency arithmetic: many dependencies each at 'four nines' still compound into meaningful downtime for the whole API.
- Network timeouts and bounded per-dependency thread pools (or semaphores) so one slow dependency cannot exhaust the container.
- Circuit breakers that open on error rate and probe periodically, with fallbacks that degrade rather than fail.
- Real-time dashboards of per-dependency health as an operational necessity, not an afterthought.
- The same ideas later shipped as Hystrix (now in maintenance) and live on in Resilience4j — see [[Resilience4j]].

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Availability]]
- [[Bulkheads]]
- [[Circuit Breakers]]
- [[Fault Tolerance]]
- [[Graceful Degradation]]
- [[Reliability]]
- [[Testing Strategies]]

### Design problems

- *(none link here yet)*
