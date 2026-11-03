---
title: Dynamo Paper
type: resource
resource_type: paper
tags: [resource, paper]
url: https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf
author: "Giuseppe DeCandia et al. (Amazon), SOSP 2007"
year: "2007"
status: unread
priority: medium
related_concepts: ["[[Consistent Hashing]]", "[[Quorums]]", "[[Gossip Protocols]]"]
related_problems: ["[[Design a Distributed Key-Value Store]]"]
summary: "The paper that put consistent hashing, sloppy quorums, hinted handoff, vector clocks, Merkle-tree anti-entropy and gossip membership into one 'always writeable' key-value store."
created: 2026-11-03
updated: 2026-11-03
---

# Dynamo Paper

> [!abstract] Why this resource
> Half of [[Design a Distributed Key-Value Store]] and the whole of [[Mini-KV Store]] are this paper. Its Table 1 is the best one-page summary of which technique solves which problem, and the 99.9th-percentile SLA framing is where tail-latency thinking in industry started.

**Link:** [https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf)  
**Type:** Paper · **Author:** Giuseppe DeCandia et al. (Amazon), SOSP 2007 · **Year:** 2007

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> PDF (SOSP 2007); about 16 pages. Link fetched and verified 2026-11-03.

## When to read it

Sections 4 (System Architecture) and 6 (Experiences) after the Consistent Hashing, Quorums and Eventual Consistency notes — about two hours. Keep Table 1 next to you while designing the KV store problem.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Table 1 maps problems to techniques: consistent hashing for incremental scalability, vector clocks reconciled during reads, sloppy quorum and hinted handoff, Merkle-tree anti-entropy, gossip-based membership and failure detection.
- Keys are hashed (MD5) to a 128-bit identifier that selects the responsible nodes; the whole interface is get() and put().
- Dynamo is a 'zero-hop DHT': every node holds enough routing state to reach the right node directly.
- SLAs are measured at the 99.9th percentile — the example is 300 ms for 99.9% of requests at 500 requests per second.
- The design is 'always writeable': conflicts are resolved on read, with last-write-wins or application merge (the shopping-cart merge) as options.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[BASE]]
- [[CAP Theorem]]
- [[Clocks and Time]]
- [[Consistency Models]]
- [[Consistent Hashing]]
- [[Coordination vs Availability]]
- [[Distributed Caching]]
- [[Eventual Consistency]]
- [[Failure Detection]]
- [[Functional vs Non-Functional Requirements]]
- [[Gossip Protocols]]
- [[Latency]]
- [[Network Partitions]]
- [[PACELC]]
- [[Quorums]]
- [[SQL vs NoSQL]]

### Design problems

- [[Design a Distributed Key-Value Store]]
