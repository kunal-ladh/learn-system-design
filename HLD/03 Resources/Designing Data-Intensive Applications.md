---
title: Designing Data-Intensive Applications
type: resource
resource_type: book
tags: [resource, book]
url: https://dataintensive.net/
author: "Martin Kleppmann"
year: "2017"
status: unread
priority: high
related_concepts: ["[[Replication]]", "[[Sharding]]", "[[Consistency Models]]"]
related_problems: ["[[Design File Storage]]", "[[Design a Collaborative Document Editor]]", "[[Design a Distributed Key-Value Store]]", "[[Design a Message Queue]]"]
summary: "The one book behind most of this curriculum: how storage, replication, partitioning, transactions and stream processing actually work, and what each design gives up."
created: 2026-11-03
updated: 2026-11-03
---

# Designing Data-Intensive Applications

> [!abstract] Why this resource
> Almost every database, messaging and distributed-systems note in this vault cites a DDIA chapter, because it explains the *why* behind replication lag, isolation anomalies, partition rebalancing and log-based messaging better than any blog post. For SDE-2 rounds it is the source of the vocabulary interviewers use when they probe a design.

**Link:** [https://dataintensive.net/](https://dataintensive.net/)  
**Type:** Book · **Author:** Martin Kleppmann · **Year:** 2017

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Author's landing page for a paid O'Reilly book (print and ebook); the page itself holds no chapter text. Link fetched and verified 2026-11-03.

## When to read it

Chapters 5–9 (replication, partitioning, transactions, the trouble with distributed systems, consistency and consensus) after weeks 4–8 of [[Start Here]]; chapter 11 (stream processing) with the messaging notes. Read a chapter *after* writing your own explanation of the matching concept note, not before.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- The book compares the ideas behind many systems rather than the configuration of one tool, and ties research ideas back to practice.
- Its stated goal is to explain why systems were designed the way they were, not only how they work — the same habit the trade-off sections here ask for.
- Chapter 5 is where leader-based replication, replication lag and read-your-writes come from; chapter 6 covers partitioning by key range vs. hash and rebalancing.
- Chapter 7 defines the isolation anomalies (dirty, non-repeatable, phantom, write skew) that [[Isolation Levels]] uses.
- Chapters 8–9 are the source for unreliable clocks, fencing tokens, linearizability and consensus used across the Distributed Systems notes.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[ACID]]
- [[At-Most-Once Delivery]]
- [[Availability]]
- [[BASE]]
- [[Backups and Restoration]]
- [[Backward Compatibility]]
- [[Blob Storage]]
- [[Block Storage]]
- [[CAP Theorem]]
- [[CQRS]]
- [[Cache Invalidation]]
- [[Change Data Capture]]
- [[Client State and Reconnection]]
- [[Connection Pooling]]
- [[Consensus]]
- [[Consistency Models]]
- [[Consistent Hashing]]
- [[Consumer Groups]]
- [[Coordination vs Availability]]
- [[Data Lakes]]
- [[Data Modeling]]
- [[Data Ownership and Lifecycle]]
- [[Data Retention and Archival]]
- [[Data Warehouses]]
- [[Database Scaling]]
- [[Dead-Letter Queues]]
- [[Delivery Semantics]]
- [[Disaster Recovery]]
- [[Distributed ID Generation]]
- [[Distributed Locks]]
- [[Distributed Transactions]]
- [[Domain-Driven Design]]
- [[Event Schema Evolution]]
- [[Event Schemas]]
- [[Event Sourcing]]
- [[Event-Driven Architecture]]
- [[Eventual Consistency]]
- [[Exactly-Once Semantics]]
- [[Failure Detection]]
- [[Fan-Out]]
- [[File Systems]]
- [[Full-Text Search]]
- [[Gossip Protocols]]
- [[HTTP and HTTPS]]
- [[Horizontal vs Vertical Scaling]]
- [[Indexing]]
- [[Inverted Indexes]]
- [[Isolation Levels]]
- [[Kafka]]
- [[Latency]]
- [[Leader Election]]
- [[Message Queues]]
- [[Network Partitions]]
- [[Object Storage]]
- [[Optimistic and Pessimistic Locking]]
- [[Ordering Guarantees]]
- [[Outbox Pattern]]
- [[PACELC]]
- [[Partitioning]]
- [[Pub-Sub]]
- [[Query Execution and Optimization]]
- [[Quorums]]
- [[Read Replicas]]
- [[Real-Time Systems]]
- [[Recovery Point Objective]]
- [[Recovery Time Objective]]
- [[Relational Database Internals]]
- [[Reliability]]
- [[Replication]]
- [[SQL vs NoSQL]]
- [[Scalability]]
- [[Schema Evolution]]
- [[Search Engines]]
- [[Search Indexing Pipelines]]
- [[Sharding]]
- [[Split-Brain]]
- [[Synchronous vs Asynchronous Communication]]
- [[Throughput]]
- [[Time-Series Databases]]
- [[Topic Partitions]]
- [[Transactions]]
- [[Versioning and Migrations]]
- [[WebSockets]]
- [[Write-Back Caching]]

### Design problems

- [[Design File Storage]]
- [[Design a Collaborative Document Editor]]
- [[Design a Distributed Key-Value Store]]
- [[Design a Message Queue]]
