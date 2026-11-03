---
title: Martin Fowler - Microservices
type: resource
resource_type: blog
tags: [resource, blog]
url: https://martinfowler.com/articles/microservices.html
author: "James Lewis and Martin Fowler"
year: "2014"
status: unread
priority: medium
related_concepts: ["[[Microservices]]", "[[Monoliths]]", "[[Data Ownership and Lifecycle]]"]
related_problems: []
summary: "The article that defined the style through nine characteristics — services around business capabilities, smart endpoints and dumb pipes, decentralised data, design for failure."
created: 2026-11-03
updated: 2026-11-03
---

# Martin Fowler - Microservices

> [!abstract] Why this resource
> It is the definition interviewers have in mind when they say 'microservices', and its own caution (monolith-first, unproven long-term) is useful ammunition when you argue for a modular monolith instead.

**Link:** [https://martinfowler.com/articles/microservices.html](https://martinfowler.com/articles/microservices.html)  
**Type:** Engineering blog post · **Author:** James Lewis and Martin Fowler · **Year:** 2014

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free article. Link fetched and verified 2026-11-03.

## When to read it

The characteristics section (twenty minutes) with [[Microservices]]; note the 'one synchronous call per user request' rule from The Guardian.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Definition: small services in their own processes, communicating through lightweight mechanisms, built around business capabilities, independently deployable.
- A component is a unit of software that is independently replaceable and upgradeable; services give explicit remote interfaces.
- Conway's law: organising around business capabilities gives cross-functional teams that own a product, not a project.
- Decentralised data: each service owns its database; transactionless coordination with eventual consistency and compensations.
- Nine characteristics, and the authors' own caution that long-term outcomes were unproven and that chained synchronous calls multiply downtime.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Data Ownership and Lifecycle]]
- [[Microservices]]
- [[Monoliths]]
- [[Synchronous vs Asynchronous Communication]]

### Design problems

- *(none link here yet)*
