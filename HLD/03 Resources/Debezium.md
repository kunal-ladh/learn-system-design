---
title: Debezium
type: resource
resource_type: open-source
tags: [resource, open-source]
url: https://debezium.io/
author: "Debezium project (Commonhaus Foundation)"
year: ""
status: unread
priority: medium
related_concepts: ["[[Change Data Capture]]", "[[Outbox Pattern]]", "[[Search Indexing Pipelines]]"]
related_problems: []
summary: "The open-source change-data-capture platform that streams committed inserts, updates and deletes from databases into Kafka or other sinks."
created: 2026-11-03
updated: 2026-11-03
---

# Debezium

> [!abstract] Why this resource
> It is the tool that makes [[Change Data Capture]] concrete: the change-event shape (before/after images, source LSN) and the fact that Debezium Server needs no Kafka are both useful to know in a design discussion.

**Link:** [https://debezium.io/](https://debezium.io/)  
**Type:** Open-source implementation · **Author:** Debezium project (Commonhaus Foundation)

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Project landing page; docs under /documentation/reference/stable/. Link fetched and verified 2026-11-03.

## When to read it

The landing page and the change-event example after [[Change Data Capture]]; the Outbox Event Router docs when you build an outbox relay.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Eighteen source connectors (PostgreSQL, MySQL, Oracle, SQL Server, MongoDB, …) and 26 sink connectors.
- Four ways to run it: Debezium Platform on Kubernetes, Kafka Connect, Debezium Server (standalone, no Kafka) and an operator.
- Debezium Server sinks include Kinesis, Pub/Sub, Pulsar, Redis, RabbitMQ and NATS.
- A change event carries op ('c', 'u', 'd'), before and after images and source metadata such as the PostgreSQL LSN.
- Apache-2.0; current stable 3.7 with 3.8 in development (adds an Elasticsearch sink).

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[CQRS]]
- [[Cache Invalidation]]
- [[Change Data Capture]]
- [[Data Warehouses]]
- [[Outbox Pattern]]
- [[Search Engines]]
- [[Search Indexing Pipelines]]

### Design problems

- *(none link here yet)*
