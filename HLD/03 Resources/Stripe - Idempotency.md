---
title: Stripe - Idempotency
type: resource
resource_type: blog
tags: [resource, blog]
url: https://stripe.com/blog/idempotency
author: "Brandur Leach (Stripe)"
year: "2017"
status: unread
priority: high
related_concepts: ["[[Idempotency]]", "[[Designing APIs and Contracts]]", "[[Duplicate Message Processing]]"]
related_problems: ["[[Design a Notification System]]", "[[Design a Payment System]]", "[[Design a Ticket Booking System]]", "[[Design a Webhook Delivery System]]"]
summary: "How Stripe's Idempotency-Key header makes charge requests safe to retry across ambiguous network failures, with backoff and jitter on the client side."
created: 2026-11-03
updated: 2026-11-03
---

# Stripe - Idempotency

> [!abstract] Why this resource
> It is the shortest complete explanation of idempotency keys, from the company whose API everyone copies. It is required reading before [[Design a Payment System]] and the webhook and notification problems.

**Link:** [https://stripe.com/blog/idempotency](https://stripe.com/blog/idempotency)  
**Type:** Engineering blog post · **Author:** Brandur Leach (Stripe) · **Year:** 2017

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free blog post. Link fetched and verified 2026-11-03.

## When to read it

Whole post (ten minutes) in week 8 of [[Start Here]]; follow with [[Brandur - Implementing Idempotency Keys]] for the server side.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- The Idempotency-Key header on POST endpoints lets a client retry a charge with the same key and be charged once.
- On a response failure (the operation succeeded but the client never saw it) the server replies with the cached result.
- Clients back off exponentially (wait proportional to 2^n after n failures) so they do not hammer a struggling server.
- Random jitter spreads retries so many clients failing at once do not retry in lockstep.
- Under HTTP semantics PUT and DELETE are already idempotent; POST needs the key.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Designing APIs and Contracts]]
- [[Duplicate Message Processing]]
- [[Exactly-Once Semantics]]
- [[Idempotency]]
- [[Outbox Pattern]]
- [[REST]]

### Design problems

- [[Design a Notification System]]
- [[Design a Payment System]]
- [[Design a Ticket Booking System]]
- [[Design a Webhook Delivery System]]
