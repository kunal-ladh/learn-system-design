---
title: The Tail at Scale
type: resource
resource_type: paper
tags: [resource, paper]
url: https://research.google/pubs/the-tail-at-scale/
author: "Jeffrey Dean and Luiz André Barroso, Communications of the ACM"
year: "2013"
status: unread
priority: medium
related_concepts: ["[[Latency]]", "[[Fan-Out]]", "[[Timeouts and Retries]]"]
related_problems: ["[[Design a Social Feed]]", "[[Design a Video Streaming Service]]"]
summary: "Why p99 latency dominates at scale when a request fans out to many servers, and the tail-tolerant techniques — hedged requests, tied requests, micro-partitions — that keep it bounded."
created: 2026-11-03
updated: 2026-11-03
---

# The Tail at Scale

> [!abstract] Why this resource
> Every fan-out design in [[Design a Social Feed]] or [[Design Search Autocomplete]] runs into the arithmetic in this paper: with 100 backends, a 1-in-100 slow server is hit by nearly every request. It is the justification for hedging, timeouts and the latency percentiles the dashboard tracks.

**Link:** [https://research.google/pubs/the-tail-at-scale/](https://research.google/pubs/the-tail-at-scale/)  
**Type:** Paper · **Author:** Jeffrey Dean and Luiz André Barroso, Communications of the ACM · **Year:** 2013

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Google Research landing page with abstract; the full text is the CACM link on the page (open access). Landing page fetched and verified 2026-11-03.

## When to read it

The CACM full text linked from the page (about 8 pages) after [[Latency]] and [[Fan-Out]]. Note the techniques table and reproduce the 'probability that at least one of N is slow' calculation yourself.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- The abstract's framing: temporary high-latency episodes that do not matter in moderate-size systems come to dominate at large scale.
- Systems that respond within 100 ms feel fluid to users; Google search updates results within a few tens of milliseconds.
- Tail-tolerant techniques can reuse resources already deployed for fault tolerance, so their overhead is low.
- The claim that these techniques let utilisation be driven higher without lengthening the tail, instead of over-provisioning.
- In the full text: hedged requests, tied requests, micro-partitions, selective replication and latency-induced probation.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Distributed Tracing]]
- [[Fan-Out]]
- [[Functional vs Non-Functional Requirements]]
- [[Latency]]
- [[Load Balancers]]
- [[Monitoring User-Facing Performance]]
- [[Quorums]]
- [[Retry Storms]]
- [[Scalability]]
- [[Search Engines]]
- [[Sharding]]
- [[Synchronous vs Asynchronous Communication]]
- [[Timeouts and Retries]]
- [[Tracing]]

### Design problems

- [[Design a Social Feed]]
- [[Design a Video Streaming Service]]
