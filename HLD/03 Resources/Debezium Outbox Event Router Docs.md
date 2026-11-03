---
title: Debezium Outbox Event Router Docs
type: resource
resource_type: documentation
tags: [resource, documentation]
url: https://debezium.io/documentation/reference/stable/transformations/outbox-event-router.html
author: "Debezium community"
year: ""
status: unread
priority: medium
related_concepts: ["[[Outbox Pattern]]", "[[Change Data Capture]]"]
related_problems: []
summary: "Reference for the single message transformation that turns rows in an outbox table into routed, keyed Kafka messages — the production shape of the outbox pattern."
created: 2026-11-03
updated: 2026-11-03
---

# Debezium Outbox Event Router Docs

> [!abstract] Why this resource
> [[Outbox Pattern]] explains the idea; this page shows the table layout, keying and routing that make it work in practice, including why aggregateid becomes the Kafka key (ordering per aggregate). Read it before claiming 'use an outbox with CDC' in [[Design a Payment System]].

**Link:** [https://debezium.io/documentation/reference/stable/transformations/outbox-event-router.html](https://debezium.io/documentation/reference/stable/transformations/outbox-event-router.html)  
**Type:** Primary documentation · **Author:** Debezium community

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Official docs (stable release), free. Link fetched and verified 2026-11-03.

## When to read it

The example message, the basic outbox table and the routing sections, with [[Outbox Pattern]] and [[Change Data Capture]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Default outbox columns: id (uuid), aggregatetype, aggregateid, type (varchar) and payload (jsonb).
- aggregateid is emitted as the Kafka message key — the page says this is what keeps ordering correct within partitions.
- Default topic is outbox.event.${routedByValue}: an aggregatetype of 'orders' routes to outbox.event.orders.
- Apply the SMT only to outbox-table events (predicate or route.topic.regex) because heartbeat, tombstone and metadata messages have a different shape.
- Extra columns can be placed into the envelope, headers or partition; UPDATEs on the outbox trigger table.op.invalid.behavior and DELETEs are filtered out.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Change Data Capture]]
- [[Outbox Pattern]]
- [[Search Indexing Pipelines]]

### Design problems

- *(none link here yet)*
