---
title: Spanner Paper
type: resource
resource_type: paper
tags: [resource, paper]
url: https://static.googleusercontent.com/media/research.google.com/en//archive/spanner-osdi2012.pdf
author: "James C. Corbett, Jeffrey Dean et al. (Google), OSDI 2012"
year: "2012"
status: unread
priority: low
related_concepts: ["[[Distributed Transactions]]", "[[Clocks and Time]]", "[[Consistency Models]]"]
related_problems: []
summary: "Google's globally distributed database: TrueTime exposes clock uncertainty as an interval, and 'commit wait' turns that into externally consistent transactions."
created: 2026-11-03
updated: 2026-11-03
---

# Spanner Paper

> [!abstract] Why this resource
> It is the strongest counter-example to 'you cannot have consistent cross-region transactions', and the clearest demonstration that [[Clocks and Time]] is a design input, not an implementation detail. Low priority for SDE-2 rounds — read it when the CAP/PACELC notes feel settled and you want to see what the C side costs.

**Link:** [https://static.googleusercontent.com/media/research.google.com/en//archive/spanner-osdi2012.pdf](https://static.googleusercontent.com/media/research.google.com/en//archive/spanner-osdi2012.pdf)  
**Type:** Paper · **Author:** James C. Corbett, Jeffrey Dean et al. (Google), OSDI 2012 · **Year:** 2012

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> PDF (OSDI 2012). Link fetched and verified 2026-11-03.

## When to read it

Sections 2–4 (implementation, TrueTime, concurrency control) after [[Clocks and Time]] and [[Distributed Transactions]]; skip the F1 case study unless you are curious.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- TrueTime returns a TTinterval; the paper says the uncertainty is generally under 10 ms, using GPS and atomic-clock references.
- Commit wait: the coordinator picks a commit timestamp no earlier than TT.now().latest and waits until TT.after(s) before the commit becomes visible, so the expected wait is at least twice the uncertainty.
- Read-write transactions use two-phase locking plus two-phase commit across Paxos groups; read-only transactions are lock-free snapshot reads at a system-chosen timestamp.
- Paxos leader leases default to 10 seconds, and every Paxos write is logged twice (tablet log and Paxos log).
- A directory — a set of contiguous keys sharing a prefix — is the unit of placement; a 50 MB directory can be moved in a few seconds.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[CAP Theorem]]
- [[Clocks and Time]]
- [[Consistency Models]]
- [[Coordination vs Availability]]
- [[Distributed Transactions]]
- [[PACELC]]
- [[Recovery Point Objective]]

### Design problems

- *(none link here yet)*
