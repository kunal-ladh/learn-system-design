---
title: Discord - How Discord Stores Trillions of Messages
type: resource
resource_type: case-study
tags: [resource, case-study]
url: https://discord.com/blog/how-discord-stores-trillions-of-messages
author: "Bo Ingram (Discord)"
year: "2023"
status: unread
priority: high
related_concepts: ["[[Hot Keys]]", "[[Data Modeling]]", "[[Sharding]]"]
related_problems: ["[[Design a Chat System]]", "[[Design a Distributed ID Generator]]", "[[Design a Social Feed]]"]
summary: "Discord's move from 177 Cassandra nodes to 72 ScyllaDB nodes with a Rust data-service layer that coalesces hot-partition reads — a complete, numbers-included chat storage case study."
created: 2026-11-03
updated: 2026-11-03
---

# Discord - How Discord Stores Trillions of Messages

> [!abstract] Why this resource
> It is the best real-world companion to [[Design a Chat System]]: the partition key (channel plus time bucket), what hot partitions do to tail latency, request coalescing, consistent-hash routing to make coalescing work, and how a live migration was validated. Interviewers who know it will recognise the vocabulary.

**Link:** [https://discord.com/blog/how-discord-stores-trillions-of-messages](https://discord.com/blog/how-discord-stores-trillions-of-messages)  
**Type:** Engineering case study · **Author:** Bo Ingram (Discord) · **Year:** 2023

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free engineering blog post. Link fetched and verified 2026-11-03.

## When to read it

Whole post (twenty-five minutes) after [[Hot Keys]] and before attempting the chat problem.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- 177 Cassandra nodes became 72 ScyllaDB nodes with per-node disk rising from about 4 TB to 9 TB.
- Historical message fetch p99 fell from 40–125 ms to about 15 ms; insert p99 from 5–70 ms to a steady 5 ms.
- Rust data services sit between the API monolith and the database and coalesce concurrent requests for the same row into one query.
- Consistent-hash routing by channel ID sends all requests for a channel to the same data-service instance so coalescing is effective.
- The migrator checkpointed token ranges in SQLite and wrote up to 3.2 million messages per second; reads were dual-issued to both stores for validation before cut-over in May 2022.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Data Modeling]]
- [[Horizontal vs Vertical Scaling]]
- [[Hot Keys]]
- [[Partitioning]]
- [[SQL vs NoSQL]]
- [[Scalability]]
- [[Sharding]]
- [[Topic Partitions]]

### Design problems

- [[Design a Chat System]]
- [[Design a Distributed ID Generator]]
- [[Design a Social Feed]]
