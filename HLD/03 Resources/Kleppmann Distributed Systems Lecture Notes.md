---
title: Kleppmann Distributed Systems Lecture Notes
type: resource
resource_type: course
tags: [resource, course]
url: https://www.cl.cam.ac.uk/teaching/2122/ConcDisSys/dist-sys-notes.pdf
author: "Martin Kleppmann (University of Cambridge)"
year: "2021"
status: unread
priority: medium
related_concepts: ["[[Clocks and Time]]", "[[Consensus]]", "[[Consistency Models]]"]
related_problems: ["[[Design a Collaborative Document Editor]]"]
summary: "Eight lectures' worth of notes — models, two generals, Byzantine generals, clocks, broadcast, replication, consensus, case studies — short, rigorous and free, with videos."
created: 2026-11-03
updated: 2026-11-03
---

# Kleppmann Distributed Systems Lecture Notes

> [!abstract] Why this resource
> They are the best free alternative to DDIA chapters 8–9: the same author, tighter, with exercises. The clocks and broadcast lectures explain what [[Clocks and Time]] and [[Ordering Guarantees]] only summarise.

**Link:** [https://www.cl.cam.ac.uk/teaching/2122/ConcDisSys/dist-sys-notes.pdf](https://www.cl.cam.ac.uk/teaching/2122/ConcDisSys/dist-sys-notes.pdf)  
**Type:** Course / lecture series · **Author:** Martin Kleppmann (University of Cambridge) · **Year:** 2021

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free PDF (about 100 pages). Link fetched and verified 2026-11-03.

## When to read it

Lectures 3 (time), 5 (replication/quorums) and 6 (consensus) with the matching notes after week 10 of [[Start Here]]; the YouTube playlist linked from the course page pairs with the PDF.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Two generals: certainty cannot be reached with any finite number of messages.
- Byzantine generals need 3f + 1 nodes to tolerate f malicious ones.
- Latency scale: same datacenter about 1 ms, between continents about 100 ms, hard drives in a van about a day.
- RPC: what if the service crashes during the call — retrying may charge a card twice.
- Contents cover Raft, two-phase commit, linearizability, eventual consistency, quorums and a Spanner case study; licensed CC BY-SA.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Clocks and Time]]
- [[Consensus]]
- [[Consistency Models]]
- [[Distributed Locks]]
- [[Ordering Guarantees]]
- [[Quorums]]

### Design problems

- [[Design a Collaborative Document Editor]]
