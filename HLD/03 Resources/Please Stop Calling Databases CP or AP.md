---
title: Please Stop Calling Databases CP or AP
type: resource
resource_type: blog
tags: [resource, blog]
url: https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html
author: "Martin Kleppmann"
year: "2015"
status: unread
priority: high
related_concepts: ["[[CAP Theorem]]", "[[Consistency Models]]", "[[Network Partitions]]"]
related_problems: []
summary: "Why CAP's narrow definitions (consistency = linearizability, availability = every non-failing node answers) make 'CP' and 'AP' labels misleading, with concrete counter-examples."
created: 2026-11-03
updated: 2026-11-03
---

# Please Stop Calling Databases CP or AP

> [!abstract] Why this resource
> Reciting CAP is the most common weak answer in design interviews; this post is how you give a strong one — name the actual guarantees. Read it the same day as [[CAP Theorem]].

**Link:** [https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html)  
**Type:** Engineering blog post · **Author:** Martin Kleppmann · **Year:** 2015

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free blog post (CC BY 3.0). Link fetched and verified 2026-11-03.

## When to read it

Whole post (fifteen minutes) in week 2 of [[Start Here]]; then write your own paragraph on why single-leader replication is not CAP-available.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- In CAP, consistency means linearizability and availability means every request to a non-failing node gets a non-error response.
- The CAP model is a single read-write register and considers only partitions as faults; it says nothing about latency or multi-object transactions.
- Single-leader replication is not CAP-available: a client partitioned from the leader cannot write, however it is marketed.
- ZooKeeper reads are not linearizable by default; a sync before the read makes them so.
- Dynamo-style stores are CAP-available only with R = W = 1; with quorum reads and writes, minority-side clients cannot reach a quorum during a partition.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[CAP Theorem]]
- [[Consistency Models]]
- [[Network Partitions]]

### Design problems

- *(none link here yet)*
