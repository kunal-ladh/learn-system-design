---
title: Brandur - Implementing Idempotency Keys
type: resource
resource_type: blog
tags: [resource, blog]
url: https://brandur.org/idempotency-keys
author: "Brandur Leach"
year: "2017"
status: unread
priority: high
related_concepts: ["[[Idempotency]]", "[[Transactions]]", "[[Saga Pattern]]"]
related_problems: ["[[Design a Digital Wallet and Ledger]]", "[[Design a Payment System]]"]
summary: "A reference implementation of Stripe-style idempotency keys in Postgres: atomic phases, recovery points, a locked key row, and retries that resume where they stopped."
created: 2026-11-03
updated: 2026-11-03
---

# Brandur - Implementing Idempotency Keys

> [!abstract] Why this resource
> It is the server-side half of idempotency that interview answers usually skip: what the key row stores, how concurrent duplicates get a 409, how the server resumes after a crash halfway through a charge. [[Double-Entry Ledger Service]] and [[Webhook Relay]] implement exactly this.

**Link:** [https://brandur.org/idempotency-keys](https://brandur.org/idempotency-keys)  
**Type:** Engineering blog post · **Author:** Brandur Leach · **Year:** 2017

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free long-form article with a companion repo (rocket-rides-atomic). Link fetched and verified 2026-11-03.

## When to read it

The atomic-phases design and the 'Murphy in action' failure walkthrough (thirty minutes) with [[Idempotency]] in week 8 of [[Start Here]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- The server stores the request's state against the client-generated key; a retry with the same key resumes or returns the stored result.
- Work is split into atomic phases (ACID transactions committed between foreign state mutations) with recovery points such as started, ride_created, charge_created, finished.
- The idempotency_keys table has a UNIQUE (user_id, idempotency_key) index and a locked_at column; phases run at SERIALIZABLE and a concurrent duplicate gets 409 Conflict.
- The Stripe charge is made with its own derived idempotency key so a crashed server that retries does not double-charge.
- Keys need a reaper (about 24–72 hours), and none of this works on stores without transactions.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[ACID]]
- [[Duplicate Message Processing]]
- [[Idempotency]]
- [[Optimistic and Pessimistic Locking]]
- [[Saga Pattern]]
- [[Transactions]]

### Design problems

- [[Design a Digital Wallet and Ledger]]
- [[Design a Payment System]]
