---
title: Confluent - Exactly-Once Semantics in Kafka
type: resource
resource_type: deep-dive
tags: [resource, deep-dive]
url: https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/
author: "Neha Narkhede (Confluent)"
year: "2017"
status: unread
priority: medium
related_concepts: ["[[Exactly-Once Semantics]]", "[[Kafka]]", "[[Idempotency]]"]
related_problems: ["[[Design a Message Queue]]"]
summary: "How Kafka 0.11 built exactly-once from idempotent producers (sequence numbers), multi-partition transactions and read_committed consumers — and where the guarantee stops."
created: 2026-11-03
updated: 2026-11-03
---

# Confluent - Exactly-Once Semantics in Kafka

> [!abstract] Why this resource
> 'Kafka does exactly-once' is a claim interviewers love to test. This post gives you the precise version: exactly-once inside Kafka Streams and transactional produce, not for arbitrary side effects — which is exactly what [[Exactly-Once Semantics]] says.

**Link:** [https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/](https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/)  
**Type:** Architecture deep dive · **Author:** Neha Narkhede (Confluent) · **Year:** 2017

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free blog post (2017, updated 2025). Link fetched and verified 2026-11-03.

## When to read it

The idempotent producer and transactions sections with [[Exactly-Once Semantics]] in week 8 of [[Start Here]]; skip the performance appendix on first pass.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Idempotent producer: enable.idempotence=true; each batch carries a sequence number the broker uses to deduplicate retries, persisted in the replicated log so a new leader can also detect duplicates.
- Transactions write atomically across partitions and can commit consumer offsets in the same transaction; consumers choose read_committed or read_uncommitted; transactional.id gives continuity across restarts.
- processing.guarantee=exactly_once in Kafka Streams covers processing and state written back to Kafka.
- The stated limit: exactly-once applies inside Kafka Streams; RPCs to remote stores or direct client reads and writes do not get the guarantee.
- Reported cost: about 3% versus an acks=all producer and about 20% versus the default acks=1 producer for 1 KB messages with 100 ms transactions.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Delivery Semantics]]
- [[Duplicate Message Processing]]
- [[Exactly-Once Semantics]]
- [[Idempotency]]
- [[Kafka]]

### Design problems

- [[Design a Message Queue]]
