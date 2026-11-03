---
title: Jepsen Consistency Models
type: resource
resource_type: documentation
tags: [resource, documentation]
url: https://jepsen.io/consistency
author: "Kyle Kingsbury (Jepsen)"
year: ""
status: unread
priority: medium
related_concepts: ["[[Consistency Models]]", "[[Isolation Levels]]", "[[Eventual Consistency]]"]
related_problems: []
summary: "The reference map of consistency models — from read uncommitted to strict serializability — defined by the phenomena they forbid, with the dependency graph that relates them."
created: 2026-11-03
updated: 2026-11-03
---

# Jepsen Consistency Models

> [!abstract] Why this resource
> [[Consistency Models]] and [[Isolation Levels]] use the same words for subtly different things; this page is the authoritative place where both hierarchies meet. The diagram of models is worth redrawing from memory before an interview.

**Link:** [https://jepsen.io/consistency](https://jepsen.io/consistency)  
**Type:** Primary documentation · **Author:** Kyle Kingsbury (Jepsen)

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free reference page; the Models, Phenomena and Dependencies sub-pages hold the detail. Link fetched and verified 2026-11-03.

## When to read it

The index page plus the Models sub-page after [[Consistency Models]]; the Phenomena sub-page when [[Isolation Levels]] anomalies feel vague.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- A consistency model is a safety property: the set of histories a system is allowed to produce.
- Serializability means every legal history is equivalent to some total order of transactions.
- Models are often defined by proscribed phenomena, e.g. G1a (aborted read) is observing a write from a transaction that later aborted.
- Transactions form a dependency graph (write-read, write-write, read-write edges); forbidding particular cycles yields the different models.
- Three sub-sections: Models, Phenomena and Dependencies.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[BASE]]
- [[Cache Consistency]]
- [[Consistency Models]]
- [[Eventual Consistency]]
- [[Isolation Levels]]
- [[Quorums]]
- [[Read Replicas]]

### Design problems

- *(none link here yet)*
