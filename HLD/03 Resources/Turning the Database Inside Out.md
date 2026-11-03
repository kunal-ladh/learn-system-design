---
title: Turning the Database Inside Out
type: resource
resource_type: talk
tags: [resource, talk]
url: https://www.confluent.io/blog/turning-the-database-inside-out-with-apache-samza/
author: "Martin Kleppmann (Strange Loop 2014; transcript on Confluent)"
year: "2015"
status: unread
priority: medium
related_concepts: ["[[Event Sourcing]]", "[[Change Data Capture]]", "[[Kafka]]"]
related_problems: []
summary: "The talk that reframes a database as a log plus derived views — replication logs, indexes, caches and materialised views all become stream processors over an immutable event log."
created: 2026-11-03
updated: 2026-11-03
---

# Turning the Database Inside Out

> [!abstract] Why this resource
> After this talk, [[Change Data Capture]], [[Event Sourcing]] and [[Search Indexing Pipelines]] stop being separate tricks and become one idea. It is the conceptual spine of the Messaging category.

**Link:** [https://www.confluent.io/blog/turning-the-database-inside-out-with-apache-samza/](https://www.confluent.io/blog/turning-the-database-inside-out-with-apache-samza/)  
**Type:** Conference talk · **Author:** Martin Kleppmann (Strange Loop 2014; transcript on Confluent) · **Year:** 2015

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free edited transcript on the Confluent blog; the video is linked from it. Link fetched and verified 2026-11-03.

## When to read it

The transcript (forty minutes) or the linked video after [[Kafka]] and [[Change Data Capture]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Stateless backends turn the database into a gigantic, global, shared, mutable state.
- Logical-log replication turns an update into an event — a fact that remains true even after later changes.
- A secondary index adds no new information; it is a derived structure that can be rebuilt from the base table.
- Materialised views are precomputed, so there is no such thing as a cache miss.
- Transactions over an asynchronous log are described as somewhat open research.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[CQRS]]
- [[Change Data Capture]]
- [[Data Lakes]]
- [[Data Warehouses]]
- [[Event Sourcing]]
- [[Kafka]]
- [[Search Indexing Pipelines]]

### Design problems

- *(none link here yet)*
