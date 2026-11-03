---
title: microservices.io - Saga Pattern
type: resource
resource_type: documentation
tags: [resource, documentation]
url: https://microservices.io/patterns/data/saga.html
author: "Chris Richardson (microservices.io)"
year: ""
status: unread
priority: medium
related_concepts: ["[[Saga Pattern]]", "[[Distributed Transactions]]", "[[Event-Driven Architecture]]"]
related_problems: ["[[Design a Workflow Orchestrator]]"]
summary: "The canonical pattern page: a saga as a sequence of local transactions with compensations, in choreography and orchestration variants, with the drawbacks spelled out."
created: 2026-11-03
updated: 2026-11-03
---

# microservices.io - Saga Pattern

> [!abstract] Why this resource
> Interviewers ask 'how do you keep orders and payments consistent without 2PC?' and this page is the standard answer plus its costs — no automatic rollback and no isolation. Read it before [[Design a Payment System]] or [[Design a Workflow Orchestrator]].

**Link:** [https://microservices.io/patterns/data/saga.html](https://microservices.io/patterns/data/saga.html)  
**Type:** Primary documentation · **Author:** Chris Richardson (microservices.io)

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free pattern page (carries a sidebar promotion for a paid bootcamp). Link fetched and verified 2026-11-03.

## When to read it

Solution, Resulting context and Related patterns sections with [[Saga Pattern]]; ten minutes.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- The motivating problem: a credit check spans the Orders and Customers databases, so one local ACID transaction cannot work and 2PC is ruled out.
- Each business transaction that spans services becomes a saga; a failed step triggers compensating transactions that undo earlier steps.
- Choreography: each local transaction publishes events that trigger the next (Order PENDING → Order Created → credit reserved). Orchestration: an orchestrator issues commands.
- Drawbacks: no automatic rollback (you write compensations) and no isolation, so countermeasures are needed.
- Client options for an asynchronous saga: respond after completion, return an ID to poll, or push a notification.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Distributed Transactions]]
- [[Event-Driven Architecture]]
- [[Saga Pattern]]

### Design problems

- [[Design a Workflow Orchestrator]]
