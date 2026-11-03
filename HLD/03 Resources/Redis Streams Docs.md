---
title: Redis Streams Docs
type: resource
resource_type: documentation
tags: [resource, documentation]
url: https://redis.io/docs/latest/develop/data-types/streams/
author: "Redis documentation"
year: ""
status: unread
priority: medium
related_concepts: ["[[Redis]]", "[[Consumer Groups]]", "[[Message Queues]]"]
related_problems: ["[[Design a Chat System]]", "[[Design a Message Queue]]"]
summary: "Redis Streams as an append-only log with consumer groups: XADD, XREADGROUP, XACK, pending entries, XCLAIM/XAUTOCLAIM recovery and MAXLEN trimming."
created: 2026-11-03
updated: 2026-11-03
---

# Redis Streams Docs

> [!abstract] Why this resource
> Your team is moving towards Redis Streams, and it is the cheapest way to build a real queue with consumer groups on a laptop. The pending-entries list and XAUTOCLAIM are the concrete mechanics behind [[At-Least-Once Delivery]] and [[Dead-Letter Queues]].

**Link:** [https://redis.io/docs/latest/develop/data-types/streams/](https://redis.io/docs/latest/develop/data-types/streams/)  
**Type:** Primary documentation · **Author:** Redis documentation

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Official docs page with examples in many client libraries (noisy but free). Link fetched and verified 2026-11-03.

## When to read it

Sections on consumer groups and recovering from permanent failures, with [[Consumer Groups]] and [[Message Queues]]; then build the claim loop in [[Webhook Relay]] on it.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Entry IDs are time-based by default (e.g. 1692632086370-0); XADD with * generates them.
- XGROUP CREATE ... $ MKSTREAM creates a group (and the stream); consumers read new entries with XREADGROUP and the special > ID.
- XACK removes an entry from a consumer's pending list; XPENDING shows what is still unacknowledged per consumer.
- XCLAIM and XAUTOCLAIM reassign pending entries to another consumer — the documented failure-recovery pattern.
- XADD ... MAXLEN trims approximately by default; exact trimming needs the = modifier. Newer commands (XACKDEL, XDELEX, XNACK) appear in the 8.x command list.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[At-Least-Once Delivery]]
- [[At-Most-Once Delivery]]
- [[Client State and Reconnection]]
- [[Consumer Groups]]
- [[Dead-Letter Queues]]
- [[Delivery Semantics]]
- [[Message Queues]]
- [[Pub-Sub]]
- [[Real-Time Systems]]
- [[Redis]]

### Design problems

- [[Design a Chat System]]
- [[Design a Message Queue]]
