---
title: Martin Fowler - Event Sourcing
type: resource
resource_type: blog
tags: [resource, blog]
url: https://martinfowler.com/eaaDev/EventSourcing.html
author: "Martin Fowler"
year: "2005"
status: unread
priority: low
related_concepts: ["[[Event Sourcing]]", "[[CQRS]]", "[[Audit Logs]]"]
related_problems: []
summary: "The original article: store every change as an event, rebuild state by replay, snapshot for speed, and gateway external systems so replay has no side effects."
created: 2026-11-03
updated: 2026-11-03
---

# Martin Fowler - Event Sourcing

> [!abstract] Why this resource
> It gives the mechanics that [[Event Sourcing]] summarises — replay, snapshots, temporal queries, gateways disabled during replay — and the accounting analogy that explains why ledgers are naturally event-sourced.

**Link:** [https://martinfowler.com/eaaDev/EventSourcing.html](https://martinfowler.com/eaaDev/EventSourcing.html)  
**Type:** Engineering blog post · **Author:** Martin Fowler · **Year:** 2005

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free article, explicitly marked as draft material by the author. Link fetched and verified 2026-11-03.

## When to read it

The first half (concepts, rebuild, snapshots, external systems) with [[Event Sourcing]]; skip the C# detail.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Every change to application state is stored as an event in sequence; both current state and the event log are persisted.
- Facilities built on the log: complete rebuild, temporal query (state at any time), event replay (reverse and re-apply a wrong event).
- Snapshots store state periodically so recovery replays only later events.
- External systems are wrapped in gateways that are disabled during replay; external query results can be logged so replay reproduces them.
- Suggested fits: audit logs and accounting systems, where entries link back to the events that created them.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Audit Logs]]
- [[CQRS]]
- [[Data Modeling]]
- [[Event Schemas]]
- [[Event Sourcing]]

### Design problems

- *(none link here yet)*
