---
title: microservices.io - Transactional Outbox
type: resource
resource_type: documentation
tags: [resource, documentation]
url: https://microservices.io/patterns/data/transactional-outbox.html
author: "Chris Richardson (microservices.io)"
year: ""
status: unread
priority: medium
related_concepts: ["[[Outbox Pattern]]", "[[Change Data Capture]]", "[[At-Least-Once Delivery]]"]
related_problems: ["[[Design a Payment System]]"]
summary: "The pattern page for atomically updating a database and publishing a message by writing the message to an outbox table in the same transaction and relaying it afterwards."
created: 2026-11-03
updated: 2026-11-03
---

# microservices.io - Transactional Outbox

> [!abstract] Why this resource
> It is the one-page justification for the outbox table in [[Webhook Relay]] and the ledger projects, and it states the at-least-once consequence plainly: the relay can publish twice, so consumers must be idempotent.

**Link:** [https://microservices.io/patterns/data/transactional-outbox.html](https://microservices.io/patterns/data/transactional-outbox.html)  
**Type:** Primary documentation · **Author:** Chris Richardson (microservices.io)

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free pattern page. Link fetched and verified 2026-11-03.

## When to read it

Solution, Result context and Issues sections with [[Outbox Pattern]]; five minutes. Then read [[Debezium Outbox Event Router Docs]] for the production shape.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Store the message in the database as part of the transaction that updates the business entities; a separate relay sends it to the broker.
- Participants: sender, database, message outbox (a table, or a property on each record in NoSQL) and message relay.
- Messages are sent if and only if the transaction commits, in the order the application wrote them, without 2PC.
- The relay may publish a message more than once (crash after publish, before marking it sent), so consumers must deduplicate.
- Two relay implementations: transaction-log tailing and polling publisher; event sourcing is named as an alternative.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[At-Least-Once Delivery]]
- [[Change Data Capture]]
- [[Outbox Pattern]]

### Design problems

- [[Design a Payment System]]
