---
title: Release It!
type: resource
resource_type: book
tags: [resource, book]
url: https://pragprog.com/titles/mnee2/release-it-second-edition/
author: "Michael Nygard (Pragmatic Bookshelf)"
year: "2018"
status: unread
priority: medium
related_concepts: ["[[Circuit Breakers]]", "[[Bulkheads]]", "[[Timeouts and Retries]]"]
related_problems: []
summary: "The book that named the stability patterns — timeouts, circuit breaker, bulkheads, fail fast, back pressure — and the antipatterns that make systems fall over in production."
created: 2026-11-03
updated: 2026-11-03
---

# Release It!

> [!abstract] Why this resource
> The stability patterns in the Architecture Patterns notes come from here, with the production stories that explain why each exists. It is the best single source for the 'what fails and what do you do about it' part of an interview answer.

**Link:** [https://pragprog.com/titles/mnee2/release-it-second-edition/](https://pragprog.com/titles/mnee2/release-it-second-edition/)  
**Type:** Book · **Author:** Michael Nygard (Pragmatic Bookshelf) · **Year:** 2018

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Publisher page for a paid book (376 pages, 2nd edition 2018); free excerpt PDFs for the Introduction and Stability Antipatterns are linked there. Link fetched and verified 2026-11-03.

## When to read it

Part I (Create Stability): the Stability Antipatterns chapter first, then Stability Patterns, after week 9 of [[Start Here]]. The free excerpt PDFs on the page cover the Introduction and Stability Antipatterns.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Stability patterns listed on the page: Timeouts, Circuit Breaker, Bulkheads, Steady State, Fail Fast, Let It Crash, Handshaking, Test Harnesses, Decoupling Middleware, Shed Load, Create Back Pressure, Governor.
- Stability antipatterns: Integration Points, Chain Reactions, Cascading Failures, Self-Denial Attacks, Scaling Effects, Unbalanced Capacities, Dogpile, Force Multiplier, Slow Responses, Unbounded Result Sets.
- Design for Production includes DNS, load balancing, demand control, network routing and service discovery, plus a control-plane chapter.
- The last part covers chaos engineering (Simian Army, disaster simulations) — the model for [[Chaos Lab]].
- Publisher's claim worth testing against your own experience: eighty percent of project life-cycle cost is in production.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Backpressure]]
- [[Bulkheads]]
- [[Circuit Breakers]]
- [[Connection Pooling]]
- [[DNS]]
- [[Dead-Letter Queues]]
- [[Disaster Recovery]]
- [[Fault Tolerance]]
- [[Graceful Degradation]]
- [[Health Checks]]
- [[Keep-Alive Connections]]
- [[Load Balancers]]
- [[Production Incident Analysis]]
- [[Retry Storms]]
- [[Reverse Proxies]]
- [[Synchronous vs Asynchronous Communication]]
- [[Timeouts and Retries]]
- [[WebSockets]]

### Design problems

- *(none link here yet)*
