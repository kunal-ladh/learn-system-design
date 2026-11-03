---
title: PACELC Paper
type: resource
resource_type: paper
tags: [resource, paper]
url: https://www.cs.umd.edu/~abadi/papers/abadi-pacelc.pdf
author: "Daniel J. Abadi, IEEE Computer"
year: "2012"
status: unread
priority: medium
related_concepts: ["[[PACELC]]", "[[CAP Theorem]]", "[[Coordination vs Availability]]"]
related_problems: []
summary: "The six-page article that extends CAP: during a partition choose availability or consistency; otherwise choose latency or consistency — and classifies real systems on both axes."
created: 2026-11-03
updated: 2026-11-03
---

# PACELC Paper

> [!abstract] Why this resource
> CAP alone does not explain why Dynamo-style systems give up consistency when the network is healthy. This paper does, in one acronym, and its classification table is the fastest way to answer 'where does Cassandra / MongoDB / Spanner sit?' in an interview.

**Link:** [https://www.cs.umd.edu/~abadi/papers/abadi-pacelc.pdf](https://www.cs.umd.edu/~abadi/papers/abadi-pacelc.pdf)  
**Type:** Paper · **Author:** Daniel J. Abadi, IEEE Computer · **Year:** 2012

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> PDF, six pages (IEEE Computer, Feb 2012). Link fetched and verified 2026-11-03.

## When to read it

The whole thing — six pages — right after [[CAP Theorem]] and before [[PACELC]]. Reproduce the classification table from memory afterwards.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- CAP only constrains a system while a partition exists; it says nothing about normal operation, so it cannot explain reduced consistency without partitions.
- The consistency/latency trade-off exists even without partitions because availability needs replication, and replication forces a choice among synchronous, asynchronous and quorum update paths.
- PACELC reads: if there is a Partition, trade Availability vs. Consistency; Else trade Latency vs. Consistency.
- Classification in the paper: Dynamo, Cassandra and Riak default to PA/EL; VoltDB/H-Store, Megastore, BigTable and HBase are PC/EC; PNUTS is PC/EL; MongoDB is PA/EC.
- Quorum reads with R + W > N preserve consistency in the usual sense but still fall short of full Gilbert–Lynch consistency; a cited Cassandra experiment found quorum reads about four times slower than weak reads.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[CAP Theorem]]
- [[Coordination vs Availability]]
- [[PACELC]]

### Design problems

- *(none link here yet)*
