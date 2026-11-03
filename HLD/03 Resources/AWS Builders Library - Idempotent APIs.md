---
title: AWS Builders Library - Idempotent APIs
type: resource
resource_type: deep-dive
tags: [resource, deep-dive]
url: https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/
author: "Malcolm Featonby (AWS)"
year: ""
status: unread
priority: medium
related_concepts: ["[[Idempotency]]", "[[Timeouts and Retries]]", "[[Designing APIs and Contracts]]"]
related_problems: ["[[Design a Payment System]]", "[[Design an Inventory Management System]]"]
summary: "How AWS makes mutating APIs safe to retry with caller-supplied client tokens, why hashing parameters is the wrong duplicate test, and what a retried request should return."
created: 2026-11-03
updated: 2026-11-03
---

# AWS Builders Library - Idempotent APIs

> [!abstract] Why this resource
> Together with [[Stripe - Idempotency]] it is the complete answer to 'how do you make a payment API safe to retry?'. The detail that a retry gets an equivalent response rather than ResourceAlreadyExists is what separates a thoughtful answer from a textbook one.

**Link:** [https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/)  
**Type:** Architecture deep dive · **Author:** Malcolm Featonby (AWS)

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free article on aws.amazon.com (served directly). Link fetched and verified 2026-11-03.

## When to read it

Whole article (twenty minutes) with [[Idempotency]] in week 8 of [[Start Here]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Preferred design: a unique caller-provided request identifier in the API contract (ClientToken in EC2); the same identifier from the same caller is a duplicate.
- Hashing request parameters is rejected as a duplicate test, because two identical RunInstances calls may genuinely mean two instances.
- The token is recorded and the mutation performed as one ACID operation, so the token is never stored without the resource or vice versa.
- A retried request gets a semantically equivalent response instead of ResourceAlreadyExists, because the error would change the caller's control flow.
- The token is kept for the resource's lifetime plus a window; reusing a token with different parameters returns a validation error.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[At-Least-Once Delivery]]
- [[Delivery Semantics]]
- [[Designing APIs and Contracts]]
- [[Duplicate Message Processing]]
- [[Idempotency]]
- [[Input Validation]]
- [[REST]]
- [[Timeouts and Retries]]

### Design problems

- [[Design a Payment System]]
- [[Design an Inventory Management System]]
