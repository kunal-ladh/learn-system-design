---
title: Kafka Design Documentation
type: resource
resource_type: documentation
tags: [resource, documentation]
url: https://kafka.apache.org/44/design/design/
author: "Apache Kafka project"
year: ""
status: unread
priority: high
related_concepts: ["[[Kafka]]", "[[Topic Partitions]]", "[[Consumer Groups]]"]
related_problems: ["[[Design a Logging System]]", "[[Design a Message Queue]]", "[[Design a Real-Time Fraud Detection Pipeline]]"]
summary: "The 'Design' section of the Kafka docs — persistence on the filesystem, batching and zero-copy, the pull-based consumer, offsets, delivery semantics, ISR replication and log compaction, in the project's own words."
created: 2026-11-03
updated: 2026-11-03
---

# Kafka Design Documentation

> [!abstract] Why this resource
> Most Kafka explanations online are second-hand; this is the primary source behind [[Kafka]], [[Topic Partitions]], [[Consumer Groups]] and [[Exactly-Once Semantics]]. Interviewers who have run Kafka will ask about ISR and acks, and this is where those answers come from.

**Link:** [https://kafka.apache.org/44/design/design/](https://kafka.apache.org/44/design/design/)  
**Type:** Primary documentation · **Author:** Apache Kafka project

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Versioned docs page (Kafka 4.4). The unversioned kafka.apache.org/documentation/#design link is now a client-side redirect that automated tools cannot follow; the versioned URL serves the full text. Link fetched and verified 2026-11-03.

## When to read it

Motivation → Persistence → The Consumer → Message Delivery Semantics → Replication, in the week-7 slot of [[Start Here]]; Log Compaction when you reach [[Change Data Capture]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Random disk writes run at about 100 kB/s versus about 600 MB/s for linear writes on the cited setup — the reason Kafka appends sequentially to a log.
- All data goes immediately to a persistent log on the filesystem without necessarily flushing; the OS page cache replaces an in-process cache.
- sendfile (zero-copy) moves data from page cache straight to the network — but not when TLS is on, because encryption happens in user space.
- Producers group messages into compressed batches and the broker stores the batch compressed; GZIP, Snappy, LZ4 and ZStandard are supported.
- Pull model: a consumer that falls behind simply catches up when it can; its position in a partition is a single integer offset, and it can rewind.
- The table of contents also covers Message Delivery Semantics, Using Transactions, Replication (ISR, unclean leader election), Log Compaction and Quotas.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[At-Least-Once Delivery]]
- [[Backpressure]]
- [[Consumer Groups]]
- [[Data Lakes]]
- [[Event Schema Evolution]]
- [[Event Schemas]]
- [[Event-Driven Architecture]]
- [[Exactly-Once Semantics]]
- [[Kafka]]
- [[Message Queues]]
- [[Ordering Guarantees]]
- [[Pub-Sub]]
- [[Recovery Point Objective]]
- [[Replication]]
- [[Throughput]]
- [[Topic Partitions]]

### Design problems

- [[Design a Logging System]]
- [[Design a Message Queue]]
- [[Design a Real-Time Fraud Detection Pipeline]]
