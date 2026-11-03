---
title: Raft Paper
type: resource
resource_type: paper
tags: [resource, paper]
url: https://raft.github.io/raft.pdf
author: "Diego Ongaro and John Ousterhout (Stanford), extended version"
year: "2014"
status: unread
priority: medium
related_concepts: ["[[Consensus]]", "[[Leader Election]]", "[[Replication]]"]
related_problems: []
summary: "The understandable consensus algorithm: leader election with randomised timeouts, log replication, commitment by majority, and membership changes by joint consensus."
created: 2026-11-03
updated: 2026-11-03
---

# Raft Paper

> [!abstract] Why this resource
> [[Consensus]] and [[Leader Election]] are abstract until you read one algorithm end to end, and Raft was written to be read. It also explains why a five-node cluster survives two failures and why 'the leader has the most up-to-date log' is a safety property, not an accident.

**Link:** [https://raft.github.io/raft.pdf](https://raft.github.io/raft.pdf)  
**Type:** Paper · **Author:** Diego Ongaro and John Ousterhout (Stanford), extended version · **Year:** 2014

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> PDF (extended version, 2014); 18 pages. Link fetched and verified 2026-11-03.

## When to read it

Sections 5 (The Raft consensus algorithm) and 6 (Cluster membership changes) after the Consensus note; skip the user study. Pair with [[etcd Raft Library]] to see the same rules in code.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Leader election uses randomised timeouts (the example range is 150–300 ms) so split votes are rare and resolve quickly.
- An entry is committed once the leader that created it has replicated it on a majority; committing it also commits every preceding entry.
- The election restriction: a candidate with a less up-to-date log is denied votes, so a new leader always holds every committed entry.
- Membership changes use joint consensus, where majorities of both the old and new configuration must agree during the transition.
- Figure 3 lists the five safety properties: Election Safety, Leader Append-Only, Log Matching, Leader Completeness, State Machine Safety; a five-server cluster tolerates any two failures.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Consensus]]
- [[Leader Election]]
- [[Quorums]]
- [[Replication]]
- [[Split-Brain]]

### Design problems

- *(none link here yet)*
