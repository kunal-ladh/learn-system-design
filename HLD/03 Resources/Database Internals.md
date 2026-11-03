---
title: Database Internals
type: resource
resource_type: book
tags: [resource, book]
url: https://www.databass.dev/
author: "Alex Petrov (O'Reilly)"
year: "2019"
status: unread
priority: medium
related_concepts: ["[[Relational Database Internals]]", "[[Indexing]]", "[[Replication]]"]
related_problems: []
summary: "A deep dive into storage engines (B-trees, LSM-trees, WAL, page cache) and the distributed-systems half (failure detection, leader election, gossip, consensus)."
created: 2026-11-03
updated: 2026-11-03
---

# Database Internals

> [!abstract] Why this resource
> When [[Relational Database Internals]] or [[Indexing]] feels hand-wavy, this is where the mechanics are. Part II also doubles as a second explanation of the Distributed Systems notes, written for engineers rather than researchers.

**Link:** [https://www.databass.dev/](https://www.databass.dev/)  
**Type:** Book · **Author:** Alex Petrov (O'Reilly) · **Year:** 2019

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Author's landing page for a paid book (also on O'Reilly Learning); errata at databass.dev/errata. Link fetched and verified 2026-11-03.

## When to read it

Part I (Storage Engines) chapters on B-trees and LSM-trees in week 4 of [[Start Here]]; Part II (Distributed Systems) as a second pass after week 10.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- The Storage Engines part covers B-tree variants, the write-ahead log, the page cache and log-structured storage (LSM-trees, Bitcask, WiscKey).
- The Distributed Systems part covers failure detection, leader election, gossip dissemination, consistency models and consensus.
- FLP impossibility and the Two Generals problem are covered explicitly.
- The author states the book does not dissect any one database, and that two-phase commit, gossip and leader election are not NoSQL-specific.
- References 100+ papers — it is the bridge from this vault to primary sources.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[ACID]]
- [[Backups and Restoration]]
- [[Blob Storage]]
- [[Block Storage]]
- [[Connection Pooling]]
- [[Database Scaling]]
- [[File Systems]]
- [[Indexing]]
- [[Inverted Indexes]]
- [[Partitioning]]
- [[Query Execution and Optimization]]
- [[Read Replicas]]
- [[Relational Database Internals]]
- [[Replication]]
- [[SQL vs NoSQL]]
- [[Sharding]]
- [[Write-Back Caching]]

### Design problems

- *(none link here yet)*
