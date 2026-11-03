---
title: Latency Numbers Every Programmer Should Know
type: resource
resource_type: documentation
tags: [resource, documentation]
url: https://gist.github.com/jboner/2841832
author: "Community gist (credits Jeff Dean and Peter Norvig)"
year: ""
status: unread
priority: high
related_concepts: ["[[Latency]]", "[[Back-of-the-Envelope Estimation]]"]
related_problems: ["[[Design Search Autocomplete]]"]
summary: "The one-page table of reference latencies — L1 cache to a cross-ocean round trip — that every capacity estimate leans on."
created: 2026-11-03
updated: 2026-11-03
---

# Latency Numbers Every Programmer Should Know

> [!abstract] Why this resource
> Capacity estimation in interviews is only convincing when the orders of magnitude are right: memory in nanoseconds, SSD in microseconds, disk and datacenter round trips in hundreds of microseconds to milliseconds, cross-region in tens to hundreds of milliseconds. Memorise this table; the [[Estimation Cheat Sheet]] builds on it.

**Link:** [https://gist.github.com/jboner/2841832](https://gist.github.com/jboner/2841832)  
**Type:** Primary documentation · **Author:** Community gist (credits Jeff Dean and Peter Norvig)

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Public GitHub gist with a long comment thread. Link fetched and verified 2026-11-03.

## When to read it

Memorise the ten rows that matter (see the cheat sheet) in week 1 of [[Start Here]]; re-derive them from the hardware before an interview. Note that the base figures are ~2012 and the comment thread disputes a few.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- L1 cache reference 0.5 ns; main memory reference 100 ns; mutex lock/unlock 25 ns.
- Read 1 MB sequentially from memory 250 µs; from SSD about 1 ms; from disk about 20 ms.
- Round trip within the same datacenter 500 µs; disk seek 10 ms.
- Packet California → Netherlands → California about 150 ms.
- Later revisions add LLM rows (time to first token about 1 s); the table header says the numbers are ~2012 except where noted, and comments flag that they ignore queuing.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Back-of-the-Envelope Estimation]]
- [[Block Storage]]
- [[CDNs]]
- [[Capacity Planning]]
- [[Latency]]
- [[Relational Database Internals]]
- [[TCP and UDP]]
- [[TLS]]

### Design problems

- [[Design Search Autocomplete]]
